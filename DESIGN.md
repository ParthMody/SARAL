# SARAL Experimental Design

This document describes the experimental architecture implemented in SARAL and
the design transition from the original field-deployment system (v1) to the
controlled experimental instrument (v2).

SARAL is designed to study a specific decision problem:

> How does a human reviewer respond when a machine recommendation is accompanied
> by contextual evidence that the machine has not observed?

The welfare-eligibility setting is the first implementation of this paradigm.
The underlying architecture is intended to generalise to other settings involving
a machine recommendation, private contextual evidence, and a human reviewer with
authority to comply, reverse, or defer.

---

## 1. Core decision structure

Each case contains three informational components:

1. **Structured applicant record**

   * demographic and household information,
   * documentation,
   * eligibility-relevant attributes.

2. **Algorithmic recommendation**

   * binary recommendation: `APPROVE` or `REJECT`,
   * generated from the structured record,
   * does not observe the contextual field note.

3. **Field context**

   * a short observation attributed to prior field verification,
   * may support, contradict, or be largely irrelevant to the recommendation.

The reviewer then chooses one of three actions:

* `APPROVE`
* `REJECT`
* `ESCALATE`

The decision is final once submitted.

A short written rationale is recorded alongside the decision.

---

## 2. Why the algorithm does not see the field note

The central informational asymmetry is deliberate.

The recommendation is produced from the structured record alone. The reviewer sees
both that recommendation and contextual information unavailable to the machine.

This creates a setting in which disagreement with the recommendation need not
represent distrust of automation. It may instead reflect the incorporation of
additional evidence.

The design therefore distinguishes between:

* **algorithm compliance**,
* **evidence-responsive reversal**, and
* **escalation under unresolved conflict**.

The experiment is intended to study how these responses vary with the informational
relationship between the recommendation and the contextual evidence.

---

## 3. Experimental factors

The v2 experiment uses a 2 × 2 within-subject design.

### Factor 1: Algorithm recommendation

The system recommends either:

* `APPROVE`, or
* `REJECT`.

### Factor 2: Direction of contextual evidence

The field note is constructed so that its eligibility implications either run:

* **WITH** the algorithmic recommendation, or
* **AGAINST** the algorithmic recommendation.

This produces four conceptual conditions:

| Algorithm recommendation | Field signal | Interpretation                                |
| ------------------------ | ------------ | --------------------------------------------- |
| APPROVE                  | WITH         | Context supports approval                     |
| APPROVE                  | AGAINST      | Context introduces evidence against approval  |
| REJECT                   | WITH         | Context supports rejection                    |
| REJECT                   | AGAINST      | Context introduces evidence against rejection |

Participants encounter cases drawn across these conditions within the same session.

Case order is randomised.

---

## 4. Context manipulation

The experimental contrast is implemented through the field-note component.

In the context-rich condition, the reviewer receives a substantive observation
from prior field verification. These observations may concern matters such as:

* visible household assets,
* dwelling characteristics,
* documentation discrepancies,
* family property,
* apparent economic circumstances,
* other facts potentially relevant to eligibility.

The field note is intentionally outside the information set used to generate the
algorithmic recommendation.

The corresponding control presentation contains only minimal procedural context
rather than substantive field evidence.

The manipulation therefore changes the amount and decision relevance of contextual
information available to the reviewer without changing the recommendation itself.

---

## 5. Primary outcome: override

The primary behavioural outcome is **override**.

For a binary recommendation \(R\) and reviewer decision \(D\):

$$
Override =
\begin{cases}
0, & \text{if the reviewer complies with the recommendation} \\
1, & \text{if the reviewer reverses or escalates}
\end{cases}
$$

Operationally:

| Recommendation | Reviewer decision | Override |
| -------------- | ----------------- | -------- |
| APPROVE        | APPROVE           | 0        |
| APPROVE        | REJECT            | 1        |
| APPROVE        | ESCALATE          | 1        |
| REJECT         | REJECT            | 0        |
| REJECT         | APPROVE           | 1        |
| REJECT         | ESCALATE          | 1        |

Escalation is therefore **not** coded as compliance.

This matters because escalation represents a distinct institutional response:
the reviewer declines to accept the recommendation but also declines to issue the
opposite determination.

---

## 6. Decomposing override

Override contains two substantively different behaviours.

### Reversal

The reviewer issues the opposite substantive determination:

* `APPROVE → REJECT`, or
* `REJECT → APPROVE`.

This is a direct counter-determination.

### Escalation

The reviewer selects `ESCALATE`.

This represents refusal to resolve the case at the current level of review.

The primary binary override measure combines both behaviours, while secondary
analyses distinguish them directly.

This decomposition allows the study to separate:

* disagreement strong enough to produce reversal,
* uncertainty or conflict producing escalation,
* straightforward compliance.

---

## 7. Experimental session

Each experimental session follows the same structure.

### 7.1 Consent

Participants first receive the participant information and consent materials.

Consent is required before proceeding.

### 7.2 Briefing

Participants receive:

* the welfare scheme description,
* the relevant eligibility rule,
* definitions of `APPROVE`, `REJECT`, and `ESCALATE`,
* instructions describing their role as reviewer.

### 7.3 Practice case

A practice case familiarises participants with:

* the applicant record,
* the recommendation display,
* the field-note interface,
* the three decision options,
* the rationale field.

The practice response is not part of the analytic dataset.

### 7.4 Comprehension check

Participants complete a gated comprehension item before entering the experiment.

The item verifies that they can correctly interpret the structured record and field
information.

Participants receive up to two attempts.

Failure to pass prevents progression to the experimental cases.

### 7.5 Experimental cases

Each participant reviews twelve cases.

For every case the system records:

* participant/session identifier,
* vignette identifier,
* condition assignment,
* recommendation,
* field-note presentation,
* final decision,
* written rationale,
* timing information.

Cases are shown in randomised order.

Once submitted, a decision cannot be changed.

### 7.6 Post-task survey

After the twelve decisions, participants complete a short post-task survey covering:

* perceived importance of field information,
* observations that stood out,
* confidence in their decisions.

---

## 8. Vignette construction

The experimental instrument uses synthetic applicant profiles.

No vignette corresponds to a real applicant.

The vignette pool is constructed so that cases vary in substantive content while
preserving the underlying experimental structure.

Each vignette combines:

* a structured profile,
* an algorithm recommendation,
* a field observation,
* an intended directional relationship between the recommendation and field signal.

The experimental pool contains multiple vignette families so that treatment effects
are not identified from a single wording or factual pattern.

The field-note manipulation includes several substantive signal families, including
observations concerning:

* apparent affluence,
* documentation,
* housing conditions,
* ownership or family property,
* other eligibility-relevant contextual evidence.

The stimulus set should be treated as a fixed research artifact for replication.

---

## 9. Assignment and randomisation

Randomisation is handled by the application rather than by the participant-facing
interface.

The participant sees only the case currently under review.

The system controls:

* vignette selection,
* condition assignment,
* presentation order,
* recommendation/context pairing,
* session-level recording.

Randomisation and treatment information are stored server-side.

The interface does not label conditions as treatment or control.

---

## 10. Measurement

The application records behavioural and process outcomes.

### Primary

* override indicator.

### Decision decomposition

* compliance,
* reversal,
* escalation.

### Process measures

* case-level response time,
* written rationale,
* rationale length or density,
* session timing.

### Post-task measures

* perceived field-note salience,
* confidence,
* recalled or standout observations.

These measures allow analysis of both the final determination and the process by
which reviewers respond to conflicting information.

---

## 11. Analytical structure

The main estimand concerns whether contextual evidence changes the probability of
overriding the algorithmic recommendation.

Because each participant makes repeated decisions, observations are clustered
within participant.

The primary analysis uses a repeated-measures specification appropriate to the
binary override outcome.

The principal comparison examines override rates across contextual conditions.

A second central analysis examines whether the effect depends on the direction of
the field evidence relative to the recommendation.

Conceptually:

$$
Override_{ij}
=
f(
Context_{ij},
Recommendation_{ij},
SignalDirection_{ij},
Context \times Direction,
\ldots
)
$$

where \(i\) indexes participants and \(j\) indexes cases.

The research archive contains the exact pre-registered specification, estimator,
exclusion rules, robustness checks, and analysis code.

---

## 12. Interpretation

A central design objective is to avoid treating every departure from an algorithm
as evidence of "algorithm aversion."

Suppose the machine recommends approval but the reviewer observes new evidence that
strongly suggests ineligibility.

A rejection in that setting could reflect:

* distrust of the algorithm,
* correct incorporation of information unavailable to the algorithm,
* a combination of both.

The experimental design therefore manipulates the informational relationship
between the recommendation and the field evidence.

The relevant question is not simply:

> Do reviewers follow the algorithm?

It is:

> Under what informational conditions do reviewers comply, reverse, or escalate?

This distinction is the conceptual core of SARAL.

---

# v1 → v2 design history

## 13. v1: field deployment

SARAL originated as a live administrative-review prototype.

The initial system combined:

* structured welfare records,
* rule-based eligibility outputs,
* machine-generated auxiliary signals,
* operator review,
* audit flags,
* free-text observations.

The platform was deployed in a rural Maharashtra setting to examine how operators
handled algorithmically assisted welfare cases in practice.

The field deployment showed that decisions were frequently shaped by information
outside the formal structured record.

Examples included informal observations concerning:

* housing quality,
* assets,
* household circumstances,
* documentation,
* local knowledge.

Operators sometimes accepted the system recommendation, sometimes rejected it, and
sometimes deferred or escalated the case.

The deployment therefore exposed a recurring empirical problem:

> the algorithm and reviewer did not possess the same information.

This made simple measures of "agreement with the algorithm" difficult to interpret.

---

## 14. Limitation of the v1 observational design

The field deployment provided behavioural realism but limited causal identification.

Several factors varied simultaneously:

* applicant characteristics,
* algorithm recommendations,
* operator knowledge,
* field observations,
* perceived uncertainty,
* scheme-specific rules,
* reviewer discretion.

When an operator departed from a recommendation, it was therefore difficult to
identify whether the cause was:

* contextual evidence,
* prior beliefs,
* distrust of the system,
* experience,
* case difficulty,
* institutional norms,
* some combination of these factors.

The field study motivated the experimental redesign.

---

## 15. v2: controlled experimental instrument

v2 isolates the informational mechanism observed in the field.

The redesign retains the essential decision structure:

$$
\text{Structured case}
+
\text{Machine recommendation}
+
\text{Reviewer context}
\rightarrow
\text{Human decision}
$$

but replaces naturally occurring informational variation with controlled
experimental manipulation.

The principal changes were:

| v1                                        | v2                                                  |
| ----------------------------------------- | --------------------------------------------------- |
| Live administrative deployment            | Controlled experimental instrument                  |
| Naturally occurring cases                 | Synthetic fixed vignettes                           |
| Multiple uncontrolled information sources | Explicit field-note manipulation                    |
| Operational reviewers                     | Experimental participants with relevant backgrounds |
| Observational disagreement                | Randomised recommendation/context relationships     |
| Multiple algorithmic components           | Clearly identified rule-based recommendation        |
| Behaviour observed in practice            | Causal mechanism tested experimentally              |

The redesign allows the contextual evidence mechanism to be identified directly
while preserving the institutional structure that motivated the study.

---

## 16. What was retained from v1

v2 is not a separate conceptual system.

Several features of the original field deployment are deliberately preserved:

* reviewers evaluate individual welfare cases;
* the machine recommendation is visible;
* the reviewer retains final decision authority;
* contextual information may be unavailable to the machine;
* reviewers can comply or issue a counter-determination;
* escalation remains a distinct institutional action;
* written reasoning is recorded.

These features preserve the decision architecture observed in the field while
making the source of informational conflict experimentally tractable.

---

## 17. Generalisation beyond welfare

The SARAL paradigm can be represented abstractly as:

$$
(X, A(X), C) \rightarrow H
$$

where:

* \(X\) = structured information available to the machine,
* \(A(X)\) = machine recommendation,
* \(C\) = contextual evidence available only to the reviewer,
* \(H\) = human decision.

The same architecture can arise in:

* public administration,
* medical decision support,
* credit review,
* insurance,
* fraud detection,
* content moderation,
* compliance,
* hiring,
* auditing,
* other human-in-the-loop decision systems.

The welfare implementation should therefore be understood as one domain-specific
instantiation of a broader experimental paradigm.

---

## 18. Reproducibility

The following should be versioned together for any frozen experimental release:

* vignette definitions,
* recommendation assignments,
* field-note variants,
* randomisation logic,
* interface version,
* exclusion rules,
* pre-registration,
* amendments,
* analysis code.

Changing substantive vignette content after data collection creates a new stimulus
version and should not overwrite the version used in the reported study.

The repository documents the instrument implementation. Frozen research materials
and the analysis archive are available separately at:

https://www.parthmody.me/saral-materials
