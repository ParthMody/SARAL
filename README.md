# SARAL

**A reproducible experimental platform for studying how administrative reviewers integrate machine recommendations with field context.**

SARAL presents welfare-eligibility cases to a reviewer alongside a rule-based recommendation and contextual field information, then records whether the reviewer approves, rejects, or escalates the case, and why.

The platform was built to study when and how human reviewers depart from algorithmic recommendations when they hold information the algorithm cannot observe.

This repository preserves the **v2 experimental instrument** used for the pre-registered study. The experimental design and the v1 → v2 history are documented in [`DESIGN.md`](./DESIGN.md).

---

## Research materials

Paper, pre-registration and amendment, consent materials, governance protocol, de-identified data, and analysis code:

**[parthmody.me/saral-materials](https://www.parthmody.me/saral-materials)**

This repository reproduces the **experimental instrument**. The research archive reproduces the **study data and analysis**. The two layers are kept separate so that the participant-facing environment is not conflated with downstream statistical analysis.

The original Railway production deployment used during data collection has been retired. The instrument remains reproducible from this repository.

---

## Quickstart

```bash
git clone https://github.com/ParthMody/SARAL.git
cd SARAL
git checkout <<< FILL: study tag, e.g. v2-study-2026 >>>

python -m venv .venv
source .venv/bin/activate          # Windows: .\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r requirements.txt

docker run -d --name saral-db -e POSTGRES_PASSWORD=saral -p 5432:5432 postgres:<<< FILL: version >>>
export SARAL_ENV=development
export DATABASE_URL=postgresql://postgres:saral@localhost:5432/postgres

alembic upgrade head
python scripts/seed_cases.py
python scripts/verify_stimuli.py   # confirms the loaded stimulus set matches the study
uvicorn app.main:app --reload
```

The application is then available at `http://127.0.0.1:8000`.

Check out the **study tag**, not the `v2` branch. The branch continues to evolve; the tag defines the instrument used in the reported study.

---

## Frozen study version

The reported experiment corresponds to an immutable Git reference:

```text
Git branch: v2
Git tag:    <<< FILL: study tag >>>
Git commit: <<< FILL: full 40-character SHA >>>
DOI:        <<< FILL: Zenodo DOI for the frozen release >>>
```

Do not reproduce the reported study from an arbitrary later commit on `v2`.

---

## Experimental design

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

The machine recommendation derives from the structured record alone. The reviewer additionally sees contextual evidence unavailable to the machine.

### Factors

**Algorithm recommendation** — `APPROVE` / `REJECT`

**Direction of contextual evidence** — `WITH` the recommendation / `AGAINST` the recommendation

**<<< FILL: third factor >>>** — <<< FILL: levels >>>

> **This section needs your correction before publication.** The previous draft described the study as 2 × 2 × 2, then listed the third factor as CONTROL (no direction of contextual evidence) versus TREATMENT (direction present). If direction is undefined under control, direction is nested within arm rather than crossed with it, which is not a 2 × 2 × 2. Either name the actual third crossed factor here, or describe the structure accurately (e.g. "2 × 2 factorial plus a no-context control"). The cell table below and the twelve-cases-per-participant figure must both follow from whatever you state.

### Cells

| Recommendation | Field signal | Interpretation |
|---|---|---|
| APPROVE | WITH | Context supports approval |
| APPROVE | AGAINST | Context introduces evidence against approval |
| REJECT | WITH | Context supports rejection |
| REJECT | AGAINST | Context introduces evidence against rejection |
| <<< FILL: control cells >>> | | |

Each participant reviews twelve cases in randomised order. <<< FILL: one sentence stating how twelve cases map onto the cells — e.g. how many replicates per cell, and whether control cases are interleaved. >>>

All experimental applicant profiles are composite, inspired by the field deployment (Phase 1). No profile corresponds to a real welfare applicant.

---

## Outcome definition

The primary behavioural outcome is **override**: the final reviewer decision differs from the binary machine recommendation.

| Recommendation | Reviewer decision | Override |
|---|---|---:|
| APPROVE | APPROVE | 0 |
| APPROVE | REJECT | 1 |
| APPROVE | ESCALATE | 1 |
| REJECT | REJECT | 0 |
| REJECT | APPROVE | 1 |
| REJECT | ESCALATE | 1 |

Override therefore contains two substantively distinct actions:

- **Reversal** — the reviewer issues the opposite substantive determination (`APPROVE → REJECT`, `REJECT → APPROVE`).
- **Escalation** — the reviewer selects `ESCALATE`, declining to issue a determination.

Escalation is coded as override rather than compliance. The instrument records the two separately so that the composition of override can be examined; the reversal/escalation decomposition is reported in the paper as a secondary, exploratory analysis.

---

## Session flow

1. **Consent** — participant information sheet and informed consent.
2. **Briefing** — the welfare scheme, eligibility rule, machine recommendation, field information, and the three available decisions.
3. **Practice case** — not recorded; familiarises the participant with the interface.
4. **Comprehension check** — gated item confirming the participant can interpret the case record and field information. Up to two attempts; a pass is required to continue.
5. **Twelve experimental cases** — randomised order. Decisions are final on submission. A reference panel with the relevant rules and definitions is available during each case.
6. **Post-task survey** — field-note salience, standout observations, decision confidence.

For every experimental case, SARAL records the participant/session identifier, vignette identifier, experimental condition, machine recommendation, field-context presentation, reviewer decision, written rationale, and response timing.

Randomisation, assignment, and outcome recording are handled server-side. Participants never see treatment labels or assignment metadata.

---

## Randomisation and assignment

Assignment is handled by the application, not manually. The application controls vignette selection, recommendation/context pairing, condition assignment, case order, and session-level recording.

In the reported study:

- **Unit of randomisation:** <<< FILL: session or case >>>
- **Case order:** <<< FILL: how order is randomised >>>
- **Vignette selection:** <<< FILL: with or without replacement >>>
- **Cell representation within a session:** <<< FILL: how the cells appear across the twelve cases >>>
- **Balancing constraints:** <<< FILL: any composition constraints, or "none" >>>

Randomisation is non-deterministic. Exact reproduction means reproducing the **assignment mechanism**, not the specific random sequence any original participant saw.

The implementation is in the v2 application code; the procedure is documented in [`DESIGN.md`](./DESIGN.md).

---

## Repository structure

```text
SARAL/
├── app/                 # Experimental application
├── docs/                # Supporting technical/research documentation
├── migrations/          # Database migrations
├── scripts/             # Database and experiment utilities
├── saralv1/             # Preserved earlier implementation — not used for v2 reproduction
├── DESIGN.md            # Experimental design and v1 → v2 history
├── README.md
├── alembic.ini
└── requirements.txt
```

`saralv1/` is preserved for design history only. Do not use it when reproducing the v2 experiment.

---

## Environment

The study ran on:

- **Python** <<< FILL: exact version, e.g. 3.11.9 >>>
- **PostgreSQL** <<< FILL: version >>>
- Dependency versions pinned in `requirements.txt`

`requirements.txt` pins exact versions. Do not relax the pins when reproducing the reported study.

### Database

Data collection used PostgreSQL. The Docker command in the Quickstart reproduces that environment and is the canonical path.

SQLite is supported as a convenience for interface inspection:

```bash
export SARAL_ENV=development
export DATABASE_URL=sqlite:///./saral.db
```

SQLite differs from PostgreSQL in type affinity and constraint enforcement. Use it to walk the participant-facing flow, not to reproduce the study.

### Configuration

`DATABASE_URL` and `SARAL_ENV` are the only variables required for local reproduction. Copy `.env.example` to `.env` if you prefer a file-based configuration. No production secret is needed to reproduce the experimental behaviour locally.

---

## Verifying a reproduction

After seeding, `scripts/verify_stimuli.py` confirms that the loaded stimulus set matches the study. It checks:

- vignette count: <<< FILL: N >>>
- cases per condition: <<< FILL: counts >>>
- SHA-256 of the canonicalised stimulus set: <<< FILL: hash >>>

A correct reproduction lets a researcher complete the same participant-facing sequence used in the study:

```text
Consent → Briefing → Practice case → Comprehension check → 12 cases → Post-task survey
```

---

## Reproducibility requirements

A valid reproduction of the v2 experiment holds the following fixed.

**Software** — study Git tag/commit; Python version; dependency versions; database schema; application logic.

**Experimental materials** — vignette set; structured applicant records; recommendation assignments; field-note variants; condition definitions; eligibility rules; participant instructions; practice case; comprehension-check content; post-task survey.

**Procedure** — twelve cases per participant; randomisation and assignment logic; presentation order; response options; final-on-submission behaviour; timing capture; written-rationale collection.

**Analysis** (research archive) — inclusion and exclusion rules; timing thresholds; outcome coding; repeated-measures structure; estimator specification; robustness checks; figure and table generation.

---

## Stimulus set

The vignette pool is a fixed research artifact. Each vignette combines a structured applicant profile, a machine recommendation, a field observation, and a defined relationship between field signal and recommendation.

`scripts/seed_cases.py` loads the versioned v2 stimuli; it does not generate cases probabilistically. Each vignette preserves its study-defined identifier, structured record, recommendation, field-note content, condition, signal direction, arm, and assignment metadata.

The stimulus set used in the reported study is immutable. Any substantive revision creates a new version rather than replacing the original.

---

## Data and statistical reproducibility

De-identified data and analysis code are in the research archive at [parthmody.me/saral-materials](https://www.parthmody.me/saral-materials), which contains the pre-registration and amendment, participant information and consent materials, governance protocol, experimental materials, de-identified raw exports, session-timing records, exclusion criteria, and analysis scripts.

The analysis pipeline begins from the de-identified raw exports, not from a manually edited analytic dataset. Every transformation between raw export and analytic dataset is performed programmatically.

A full reproduction regenerates participant inclusion and exclusion counts, analytic sample construction, override coding, descriptive statistics, primary treatment estimates, recommendation × signal-direction estimates, escalation/reversal decompositions, robustness specifications, and all reported figures and tables.

---

## Domain adaptation

The welfare implementation is one instantiation of a more general decision architecture:

```text
Structured information available to machine
                +
Machine recommendation
                +
Context available only to reviewer
                ↓
Human decision
```

The architecture adapts to another domain by replacing the stimulus set while preserving recommendation presentation, contextual-evidence presentation, randomisation, decision recording, escalation, timing capture, and rationale collection.

A modified stimulus set constitutes a new experimental version and does not overwrite the materials used in the reported welfare experiment.

Any reuse in a new empirical setting should distinguish clearly between (1) the original v2 instrument, (2) modifications to the software, (3) changes to the stimulus set, (4) changes to procedure, and (5) changes to the analysis plan.

---

## Ethics and research governance

The study was conducted independently under a documented self-governance protocol covering informed consent, voluntariness, withdrawal, data handling, and minimisation of personally identifying information. Institutional ethics review was sought and was not available to the investigator in this capacity.

All applicant profiles in the controlled experiment are composite, inspired by the field deployment. No experimental vignette corresponds to a real welfare applicant, and the instrument does not adjudicate real welfare claims.

The governance protocol, participant information sheet, and consent materials are preserved in the research archive.

This repository is released as a research artifact. It is not a validated system for operational welfare adjudication, automated eligibility determination, or deployment in other consequential decision-making settings.

Researchers reproducing or adapting the paradigm are responsible for obtaining any approvals required in their own jurisdiction or institution.

---

## Research question

SARAL is not designed to measure whether people follow an algorithm. The question is:

> **Under what informational conditions do human reviewers comply with, reverse, or escalate an algorithmic recommendation?**

Departure from an algorithmic recommendation is not automatically algorithm aversion. Where the reviewer holds relevant evidence the machine cannot observe, disagreement may instead constitute evidence integration. The design is built to distinguish these.

---

## Citation

> Mody, P. (2026). *SARAL: Field-Generated Context and Algorithmic Override in Welfare Decision-Making* (working paper). [](https://static1.squarespace.com/static/68d08a08b06391749b62502d/t/6a951205dfa7a85a69d4450c/1788154373626/When-Context-Contradicts-Algorithm_Draft1.pdf)

---

## License

No open-source licence has currently been assigned to this repository.

Unless otherwise stated, the source code and materials in this repository remain
copyrighted by the author. No permission is granted by default to reproduce,
modify, redistribute, sublicense, or use the software for operational deployment.

The repository is made publicly available for research transparency,
reproducibility, academic review, and inspection.

Researchers wishing to reuse, adapt, or redistribute SARAL or substantial parts
of the implementation should contact the author for permission.

Research materials distributed separately through the associated archive —
including the paper, pre-registration, experimental stimuli, datasets, consent
materials, and analysis outputs — may be subject to separate reuse terms stated
with those materials.

---

## Contact

**Parth Mody**  
Email: [modyparth7@gmail.com](mailto:modyparth7@gmail.com)

For questions concerning reproduction, research use, the experimental design,
or access to associated materials, contact the author at the address above.
