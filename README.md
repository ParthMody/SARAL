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
git checkout v2-study-2026

python -m venv .venv
source .venv/bin/activate          # Windows: .\.venv\Scripts\Activate.ps1
pip install --upgrade pip
pip install -r requirements.txt

docker run -d --name saral-db -e POSTGRES_PASSWORD=saral -p 5432:5432 postgres:16
export SARAL_ENV=development
export DATABASE_URL=postgresql://postgres:saral@localhost:5432/postgres

python -m scripts.seed_vignettes
uvicorn app.main:app --reload
```

The application is then available at `http://127.0.0.1:8000`.

Check out the **study tag**, not the `v2` branch. The branch continues to evolve; the tag defines the instrument used in the reported study.

---

## Frozen study version

The reported experiment corresponds to an immutable Git reference:

```text
Git branch: v2
Git tag:    v2-study-2026
Git commit: (to be recorded after data collection concludes)
DOI:        (to be assigned via Zenodo after study completion)
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

**Algorithm recommendation** — `APPROVE` / `REJECT` (fixed per vignette profile)

**Arm** — `CONTROL` (neutral procedural field note) / `TREATMENT` (contextual field signal present)

**Signal direction** (nested within treatment) — `WITH` the recommendation / `AGAINST` the recommendation

The design is a **2 (recommendation: approve/reject) × 2 (arm: control/treatment) within-subject factorial**, with signal direction nested within the treatment arm. Signal direction is undefined for control cases because the control field note carries no directional content. The primary treatment contrast is presence versus absence of a contextual signal. The secondary contrast — signal direction — is the moderator that identifies whether reinforcing and contradicting signals produce symmetric or asymmetric override behaviour.

This is not a fully crossed 2 × 2 × 2. Direction is a property of the treatment signal, not an independently manipulated factor. Control cases contribute to the main-effect estimate; direction contrasts are estimated within the treatment arm only.

### Cells

| Arm | Recommendation | Signal direction | Profiles | Interpretation |
|---|---|---|---|---|
| Treatment | APPROVE | WITH | 3, 4, 10, 15 | Context supports approval |
| Treatment | APPROVE | AGAINST | 7, 8, 13, 14 | Context introduces doubt about approval |
| Treatment | REJECT | WITH | 1, 2, 5, 11 | Context supports rejection |
| Treatment | REJECT | AGAINST | 6, 9, 12, 16 | Context introduces doubt about rejection |
| Control | APPROVE | — | 3, 4, 7, 8, 10, 13, 14, 15 | Neutral note, no directional signal |
| Control | REJECT | — | 1, 2, 5, 6, 9, 11, 12, 16 | Neutral note, no directional signal |

Each participant reviews twelve cases in randomised order. Twelve cases are drawn from sixteen profiles: six assigned to control and six to treatment. Within each arm, three cases carry an APPROVE recommendation and three carry REJECT, giving a balanced 3/3/3/3 split across the four recommendation × arm cells per participant. Treatment cases span the four direction cells; the exact coverage depends on the random draw.

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

1. **Consent** — participant information sheet (linked) and digital informed consent with server-side timestamp.
2. **Demographics** — education level, occupation category, years of relevant experience, country of residence.
3. **Briefing** — the welfare scheme, eligibility rule (pre-2011 cutoff), accepted proof types, common disqualifications, machine recommendation, field information, and the three available decisions.
4. **Practice case** — not recorded; familiarises the participant with the interface.
5. **Comprehension check** — case-based item presenting an unambiguous rejection case (post-cutoff arrival, no pre-cutoff proof, newly constructed structure, REJECT recommendation). The participant must select the correct decision. Up to two attempts; participants who fail both proceed but are flagged in the audit log.
6. **Twelve experimental cases** — randomised order. Decisions are final on submission. A reference panel with eligibility rules and definitions is available via a persistent help button during each case. A one-time tooltip after Case 1 notifies the participant of the help panel.
7. **Post-task survey** — field-note salience (1–5), standout observations (1–5), decision confidence (1–5), open-ended feedback.
8. **Completion** — unique reference code displayed; automatic redirect to Prolific after five-second countdown.

For every experimental case, SARAL records the participant/session identifier, Prolific ID, vignette identifier, experimental condition, machine recommendation, field-context presentation, reviewer decision, written rationale (optional), and response timing (total response time, time to first action, time after decision selection).

Randomisation, assignment, and outcome recording are handled server-side. Participants never see treatment labels or assignment metadata.

---

## Randomisation and assignment

Assignment is handled by the application, not manually. The application controls vignette selection, recommendation/context pairing, condition assignment, case order, and session-level recording.

In the reported study:

- **Unit of randomisation:** Case-level within participant. Each participant is both control and treatment (within-subject design).
- **Case order:** Randomly shuffled per session using a logged integer seed.
- **Vignette selection:** Without replacement. Twelve profiles drawn from sixteen; each profile appears at most once per session.
- **Cell representation within a session:** Exactly six control cases and six treatment cases. Within each arm, three APPROVE-recommended and three REJECT-recommended cases.
- **Balancing constraints:** 3 control-approve + 3 control-reject + 3 treatment-approve + 3 treatment-reject per session. No participant sees both the control and treatment version of the same profile.

Randomisation is non-deterministic. The random seed is logged per session for auditability. Exact reproduction means reproducing the **assignment mechanism**, not the specific random sequence any original participant saw.

The implementation is in the v2 application code; the procedure is documented in [`DESIGN.md`](./DESIGN.md).

---

## Repository structure

```text
SARAL/
├── app/                 # Experimental application
│   ├── main.py          # FastAPI app entry point
│   ├── models.py        # SQLAlchemy data model
│   ├── db.py            # Database configuration
│   ├── settings.py      # Application settings
│   ├── audit.py         # Audit logging
│   ├── routes/          # Route handlers
│   │   ├── dashboard.py # Landing, consent, demographics
│   │   ├── session.py   # Briefing, practice, cases, survey, completion
│   │   └── admin.py     # Admin panel, CSV exports
│   └── templates/       # Jinja2 HTML templates
├── docs/                # Consent materials, data security protocol, codebook
├── scripts/
│   └── seed_vignettes.py# Loads the versioned stimulus set
├── Procfile             # Railway deployment start command
├── requirements.txt     # Pinned dependencies
├── runtime.txt          # Python version pin
└── README.md
```

---

## Environment

The study ran on:

- **Python** 3.13.13
- **PostgreSQL** 16 (Railway-managed instance)
- **Hosting** Railway Hobby plan, US West (California), `saral-production.up.railway.app`
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

| Variable | Purpose | Required |
|---|---|---|
| `DATABASE_URL` | PostgreSQL or SQLite connection string | Yes |
| `SARAL_ENV` | `development` or `production` | Yes |
| `SARAL_ADMIN_SECRET` | Admin panel passcode | Production only |
| `PROLIFIC_COMPLETION_URL` | Prolific redirect URL with study code | Production only |
| `PIS_URL` | Participant Information Sheet URL | Production only |

Copy `.env.example` to `.env` if you prefer a file-based configuration. No production secret is needed to reproduce the experimental behaviour locally.

---

## Verifying a reproduction

After seeding, confirm the loaded stimulus set matches the study:

```bash
python -c "
from app.db import SessionLocal
from app.models import Vignette, ArmEnum, AlgoRecommendationEnum
db = SessionLocal()
vigs = db.query(Vignette).all()
assert len(vigs) == 32, f'Expected 32, got {len(vigs)}'
assert len(set(v.pair_id for v in vigs)) == 16
assert sum(1 for v in vigs if v.arm == ArmEnum.CONTROL) == 16
assert sum(1 for v in vigs if v.arm == ArmEnum.TREATMENT) == 16
assert sum(1 for v in vigs if v.algo_recommendation == AlgoRecommendationEnum.APPROVE) == 16
assert sum(1 for v in vigs if v.algo_recommendation == AlgoRecommendationEnum.REJECT) == 16
print('Stimulus set verified: 32 vignettes, 16 profiles, balanced 16/16 by arm and recommendation')
db.close()
"
```

Expected:
- Vignettes: 32
- Profiles: 16 (pair_id 1–16)
- Control: 16, Treatment: 16
- Approve: 16, Reject: 16
- Pool version: `v2.4-mumbai-final`

A correct reproduction lets a researcher complete the same participant-facing sequence used in the study:

```text
Consent → Demographics → Briefing → Practice → Comprehension check → 12 cases → Survey → Completion
```

---

## Reproducibility requirements

A valid reproduction of the v2 experiment holds the following fixed.

**Software** — study Git tag/commit; Python version; dependency versions; database schema; application logic.

**Experimental materials** — vignette set (16 profiles × 2 arms = 32 vignette objects); structured applicant records; recommendation assignments; field-note variants (control and treatment); condition definitions; eligibility rules grounded in GR ZoPuDho-0810/Pr.Kr.96/2018/ZoPaSu-1; participant instructions; practice case; comprehension-check content; post-task survey items.

**Procedure** — twelve cases per participant; within-subject 6 control + 6 treatment assignment; randomisation and assignment logic; presentation order; response options (approve/reject/escalate); final-on-submission behaviour; timing capture (response time, time to first action, time after decision); optional written-rationale collection.

**Analysis** (research archive) — inclusion and exclusion rules; timing thresholds (fast response < 8 seconds, minimum session duration 5 minutes); outcome coding (override = decision ≠ recommendation); repeated-measures structure; estimator specification; robustness checks; figure and table generation.

---

## Stimulus set

The vignette pool is a fixed research artifact. Each vignette combines a structured applicant profile, a machine recommendation, a field observation, and a defined relationship between field signal and recommendation.

`scripts/seed_vignettes.py` loads the versioned v2 stimuli; it does not generate cases probabilistically. Each vignette preserves its study-defined profile identifier (pair_id), structured record, recommendation, field-note content (English), condition, signal direction, arm, and pool version.

All sixteen profiles are grounded in GR ZoPuDho-0810/Pr.Kr.96/2018/ZoPaSu-1 (Maharashtra SRA eligibility framework). Treatment signals are paraphrased from the Phase 1 verification corpus (260 PMAY field observations, Maharashtra, 2026) and GR-documented disqualification clauses (D1–D4, VP1–VP6, Track A/B). Control notes are neutral procedural verification statements matched in length and format.

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

The architecture adapts to another domain by replacing the stimulus set (vignette pool in `scripts/seed_vignettes.py`, briefing content, help panel definitions, comprehension check case, profile field labels in `routes/session.py`) while preserving recommendation presentation, contextual-evidence presentation, randomisation, decision recording, escalation, timing capture, and rationale collection.

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

> Mody, P. (2026). *When Context Contradicts the Algorithm: Conditional Reliance and the Anatomy of Override in Welfare Decisions* (working paper). [parthmody.me/saral-materials](https://www.parthmody.me/saral-materials)

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
