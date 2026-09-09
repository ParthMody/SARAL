# SARAL

**A reproducible experimental platform for studying how administrative reviewers integrate machine recommendations with field context.**

SARAL presents welfare-eligibility cases to a reviewer alongside a rule-based recommendation and contextual field information, then records whether the reviewer approves, rejects, or escalates the case and why.

The platform was developed to study when and how human reviewers depart from algorithmic recommendations when they possess information unavailable to the algorithm.

This repository preserves the **v2 experimental instrument** used for the pre-registered study.

For the full experimental design and the transition from the original field deployment to the controlled experiment, see [`DESIGN.md`](./DESIGN.md).

## Research materials

Paper, pre-registration, amendment, consent materials, governance protocol, de-identified data, and analysis code:

[parthmody.me/saral-materials](https://www.parthmody.me/saral-materials)

The original Railway production deployment used during data collection has been retired. The instrument remains reproducible from this repository.

---

# Experimental design

The core decision structure is:

```text
Structured applicant record
        +
Machine recommendation
        +
Reviewer-only field context
        ↓
Human decision
```

The machine recommendation is derived from the structured record alone.

The reviewer additionally sees contextual evidence that is unavailable to the machine.

The v2 experiment uses a **2 × 2 within-subject design**.

## Experimental factors

### Algorithm recommendation

- `APPROVE`
- `REJECT`

### Direction of contextual evidence

- `WITH` the recommendation
- `AGAINST` the recommendation

This produces four conceptual combinations:

| Recommendation | Field signal | Interpretation |
|---|---|---|
| APPROVE | WITH | Context supports approval |
| APPROVE | AGAINST | Context introduces evidence against approval |
| REJECT | WITH | Context supports rejection |
| REJECT | AGAINST | Context introduces evidence against rejection |

Each participant reviews twelve cases in randomised order.

All experimental applicant profiles are synthetic.

---

# Outcome definition

The primary behavioural outcome is **override**.

An override occurs whenever the final reviewer decision differs from the binary machine recommendation.

| Recommendation | Reviewer decision | Override |
|---|---|---:|
| APPROVE | APPROVE | 0 |
| APPROVE | REJECT | 1 |
| APPROVE | ESCALATE | 1 |
| REJECT | REJECT | 0 |
| REJECT | APPROVE | 1 |
| REJECT | ESCALATE | 1 |

Override therefore contains two substantively distinct actions.

## Reversal

The reviewer issues the opposite substantive determination:

```text
APPROVE → REJECT
REJECT  → APPROVE
```

## Escalation

The reviewer selects:

```text
ESCALATE
```

Escalation is coded as override rather than compliance.

This distinction is central to the design because disagreement with a machine can take the form of either counter-determination or unresolved conflict.

---

# Session flow

Each experimental session follows the same sequence.

1. **Consent**  
   Participant information sheet and informed consent.

2. **Briefing**  
   Explanation of the welfare scheme, eligibility rule, machine recommendation, field information, and the three available reviewer decisions.

3. **Practice case**  
   A non-recorded case used to familiarise the participant with the interface.

4. **Comprehension check**  
   A gated item confirming that the participant can correctly interpret the case record and field information. Participants receive up to two attempts and must pass to continue.

5. **Experimental cases**  
   Twelve vignettes presented in randomised order. Decisions are final once submitted. A reference panel containing the relevant rules and definitions is available during each case.

6. **Post-task survey**  
   Measures including field-note salience, standout observations, and decision confidence.

For every experimental case, SARAL records:

- participant/session identifier;
- vignette identifier;
- experimental condition;
- machine recommendation;
- field-context presentation;
- reviewer decision;
- written rationale;
- response timing.

Randomisation, assignment, and outcome recording are handled server-side.

---

# Repository structure

The `v2` branch contains the controlled experimental instrument.

```text
SARAL/
├── app/                 # Experimental application
├── docs/                # Supporting technical/research documentation
├── migrations/          # Database migrations
├── scripts/             # Database and experiment utilities
├── saralv1/             # Preserved earlier implementation
├── DESIGN.md            # Experimental design and v1 → v2 history
├── README.md
├── alembic.ini
└── requirements.txt
```

The `saralv1/` directory preserves the earlier implementation for design-history purposes and should not be used when reproducing the v2 experiment.

The application repository and the research archive serve different purposes:

- this repository reproduces the **experimental instrument**;
- the research archive reproduces the **study data and analysis**.

---

# Reproducing the experiment

## 1. Clone the repository

```bash
git clone https://github.com/ParthMody/SARAL.git
cd SARAL
git checkout v2
```

For exact reproduction of the reported study, use the frozen study tag or commit recorded in the research archive rather than an arbitrary later version of the branch.

For example:

```bash
git checkout <study-tag-or-commit>
```

Do not reproduce the reported study from an arbitrary later commit.

---

## 2. Create a Python environment

SARAL requires Python 3.10 or later.

```bash
python -m venv .venv
```

Activate the environment.

### macOS / Linux

```bash
source .venv/bin/activate
```

### Windows PowerShell

```powershell
.\.venv\Scripts\Activate.ps1
```

---

## 3. Install dependencies

```bash
pip install --upgrade pip
pip install -r requirements.txt
```

Application dependencies are defined in [`requirements.txt`](./requirements.txt).

For exact reproduction, use the dependency versions recorded by the frozen study release.

---

## 4. Configure the environment

SARAL supports database configuration through `DATABASE_URL`.

For local reproduction with SQLite:

### macOS / Linux

```bash
export SARAL_ENV=development
export DATABASE_URL=sqlite:///./saral.db
```

### Windows PowerShell

```powershell
$env:SARAL_ENV="development"
$env:DATABASE_URL="sqlite:///./saral.db"
```

If the repository contains an `.env.example`, create a local environment file with:

```bash
cp .env.example .env
```

Only non-public deployment credentials should differ from the study configuration.

No production secret should be required to reproduce the experimental behaviour locally.

---

## 5. Initialise the database

Apply the database migrations:

```bash
alembic upgrade head
```

This creates the database schema expected by the v2 application.

---

## 6. Load the experimental stimuli

Load the fixed vignette set used by the experiment:

```bash
python scripts/seed_cases.py
```

The seeding process should load the versioned v2 experimental stimuli rather than generate new cases probabilistically.

Each vignette should preserve the study-defined:

- vignette identifier;
- structured applicant record;
- algorithm recommendation;
- field-note content;
- experimental condition;
- signal direction;
- treatment/control status;
- any metadata required by the assignment logic.

The vignette set used during the reported study should be treated as immutable.

Any substantive revision to a vignette should receive a new version rather than replacing the original stimulus.

---

## 7. Run the instrument

Start the local application:

```bash
uvicorn app.main:app --reload
```

The application should then be available at:

```text
http://127.0.0.1:8000
```

A successful reproduction should allow a researcher to complete the same participant-facing sequence used in the study:

```text
Consent
→ Briefing
→ Practice case
→ Comprehension check
→ 12 experimental cases
→ Post-task survey
```

The former Railway-hosted production instance is no longer maintained.

---

# Reproducibility requirements

A valid reproduction of the v2 experiment requires the following components to be held fixed.

## Software

- study Git commit or tag;
- Python version;
- dependency versions;
- database schema;
- application logic.

## Experimental materials

- vignette set;
- structured applicant records;
- recommendation assignments;
- field-note variants;
- condition definitions;
- eligibility rules;
- participant instructions;
- practice case;
- comprehension-check content;
- post-task survey.

## Experimental procedure

- twelve cases per participant;
- randomisation procedure;
- assignment logic;
- presentation order;
- response options;
- final-on-submission decision behaviour;
- timing capture;
- written-rationale collection.

## Analysis

- sample inclusion and exclusion rules;
- timing thresholds;
- outcome coding;
- repeated-measures structure;
- estimator specification;
- robustness checks;
- figure and table generation.

The repository reproduces the **instrument**.

The associated research archive reproduces the **analysis**.

---

# Randomisation and assignment

Experimental assignment is handled by the application rather than manually.

The participant sees only the currently assigned case and does not see treatment labels or internal assignment metadata.

The application controls:

- vignette selection;
- recommendation/context pairing;
- condition assignment;
- case order;
- session-level recording.

The exact randomisation procedure used in the reported study is documented in [`DESIGN.md`](./DESIGN.md) and implemented in the v2 application code.

A reproducer should be able to determine:

- the unit of randomisation;
- whether assignment occurs at the session or case level;
- how case order is randomised;
- how the four experimental combinations are represented within a session;
- whether vignette selection occurs with or without replacement;
- any balancing or composition constraints.

If the production experiment used non-deterministic randomisation, exact reproduction means reproducing the **assignment mechanism**, not reconstructing the exact random sequence observed by each original participant.

---

# Frozen study version

The reported experiment should correspond to an immutable Git reference.

The research archive should record:

```text
Git branch: v2
Git tag: <study-tag>
Git commit: <full-commit-SHA>
```

The branch itself may continue to evolve. The frozen tag or commit defines the exact experimental instrument associated with the reported study.

A suitable naming convention is:

```text
v2-study-2026
```

or:

```text
study-2026-06
```

---

# Vignettes and domain adaptation

The experimental vignette pool is a fixed research artifact.

Each vignette combines:

- a structured applicant profile;
- a machine recommendation;
- a field observation;
- a defined relationship between the field signal and recommendation.

The welfare implementation is one instantiation of the broader SARAL decision architecture.

Conceptually:

```text
Structured information available to machine
                +
Machine recommendation
                +
Context available only to reviewer
                ↓
Human decision
```

The same architecture can be adapted to another domain by replacing the domain-specific stimulus set while preserving:

- recommendation presentation;
- contextual-evidence presentation;
- randomisation;
- decision recording;
- escalation;
- timing capture;
- written rationale collection.

A modified stimulus set constitutes a new experimental version and should not overwrite the materials used in the reported welfare experiment.

---

# Data and statistical reproducibility

De-identified study data and analysis code are available at:

[parthmody.me/saral-materials](https://www.parthmody.me/saral-materials)

The archive includes:

- pre-registration;
- pre-registration amendment;
- participant information and consent materials;
- governance protocol;
- experimental materials;
- de-identified raw exports;
- session-timing records;
- exclusion criteria;
- analysis scripts.

The analysis pipeline should begin from de-identified raw exports rather than a manually edited analytic dataset.

A full reproduction should regenerate:

- participant inclusion and exclusion counts;
- analytic sample construction;
- override coding;
- descriptive statistics;
- primary treatment estimates;
- recommendation × signal-direction estimates;
- escalation/reversal decompositions;
- robustness specifications;
- reported figures;
- reported tables.

Any transformation between the raw export and final analytic dataset should be performed programmatically.

---

# Instrument reproducibility vs statistical reproducibility

SARAL has two distinct reproducibility layers.

## Instrument reproducibility

This repository reconstructs the environment in which participants made their decisions.

It preserves:

- the participant-facing interface;
- experiment flow;
- experimental stimuli;
- assignment logic;
- recommendation presentation;
- contextual-information presentation;
- decision recording;
- response timing;
- rationale collection.

## Statistical reproducibility

The research archive reconstructs the empirical results reported in the paper.

It preserves:

- de-identified observations;
- data-cleaning logic;
- exclusion rules;
- outcome construction;
- statistical models;
- robustness checks;
- figure generation;
- table generation.

Keeping these layers separate prevents the participant-facing experimental instrument from being conflated with downstream statistical analysis.

---

# Ethics and research governance

The study was conducted independently under a documented self-governance protocol covering:

- informed consent;
- voluntariness;
- withdrawal;
- data handling;
- minimisation of personally identifying information.

Institutional ethics review was not available to the investigator in this capacity.

All applicant profiles used in the controlled experiment are synthetic.

No experimental vignette corresponds to a real welfare applicant.

The governance protocol, participant information sheet, and consent materials are preserved in the associated research archive.

This repository is released as a research artifact. It should not be interpreted as a validated system for operational welfare adjudication, automated eligibility determination, or deployment in other consequential decision-making environments.

---

# v1 → v2 design history

SARAL originated as a live field-deployment system.

The v1 implementation combined structured welfare records, machine-generated outputs, field observations, operator review, and written reasoning.

The field deployment showed that reviewer decisions were often influenced by information that was not represented in the structured inputs available to the decision system.

This created an identification problem.

When a reviewer departed from the machine recommendation, the disagreement could reflect:

- contextual evidence unavailable to the machine;
- reviewer experience;
- distrust of the system;
- case difficulty;
- institutional practice;
- or several of these mechanisms simultaneously.

v2 converted that observational problem into a controlled experiment.

The redesign preserved the core decision architecture:

```text
Machine recommendation
        +
Reviewer-only information
        +
Human discretion
```

while experimentally controlling the relationship between the machine recommendation and contextual evidence.

This makes it possible to study whether departures from an algorithmic recommendation are systematically responsive to information that the algorithm itself could not observe.

The full conceptual and experimental history is documented in [`DESIGN.md`](./DESIGN.md).

---

# Research question

SARAL is not designed merely to measure whether people follow an algorithm.

The underlying question is:

> **Under what informational conditions do human reviewers comply with, reverse, or escalate an algorithmic recommendation?**

A departure from an algorithm should not automatically be interpreted as algorithm aversion.

If the reviewer possesses relevant evidence unavailable to the machine, disagreement may instead constitute evidence integration.

The experimental design is intended to distinguish these mechanisms.

---

# Ethics note on the field and experimental phases

SARAL has both a field-development history and a controlled experimental implementation.

The v2 experimental applicant profiles are synthetic, and the experimental instrument does not adjudicate real welfare claims.

Materials describing the independent research-governance process, participant consent, data handling, and study procedures are maintained in the associated research archive.

Researchers reproducing or adapting the paradigm are responsible for obtaining any ethics or institutional approvals required in their own jurisdiction or institution.

---

# Citation

If you use SARAL, its experimental design, or associated research materials, please cite:

> Mody, P. (2026). *SARAL: Field-Generated Context and Algorithmic Override in Welfare Decision-Making* (working paper).  
> https://static1.squarespace.com/static/68d08a08b06391749b62502d/t/6a951205dfa7a85a69d4450c/1788154373626/When-Context-Contradicts-Algorithm_Draft1.pdf

---

# Reuse

The software in this repository is provided as a research artifact.

Research materials distributed through the associated archive, including the paper, pre-registration, stimuli, datasets, consent materials, and analysis outputs, may carry separate reuse terms.

Any reuse of SARAL in a new empirical setting should clearly distinguish:

1. the original v2 experimental instrument;
2. modifications to the software;
3. changes to the vignette set;
4. changes to the experimental procedure;
5. changes to the analysis plan.
