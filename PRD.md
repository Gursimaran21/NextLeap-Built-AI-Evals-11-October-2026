# PRD — AI Evals

**Product:** AI Evaluation System ("AI Evals")
**Reference application:** Meeting Summarizer (an LLM feature)
**Status:** Draft for review
**Version:** 1.0
**Date:** 2026-10-11
**Owner:** TBD (Product + ML)
**Source material:** [Evals Class — Meeting summarizer case study](https://evals-for-everyone.vercel.app/meeting-summarizer/foundations)

---

## 1. Summary

An **eval is a PRD for AI behavior**. A traditional PRD says what a product should do; an eval says, in a form you can actually run, what *"behaves correctly"* means — and then checks it every time something changes.

This PRD specifies **AI Evals**: a repeatable evaluation system for an LLM feature. The reference application is a **Meeting Summarizer** that reads a meeting transcript and writes a short summary plus action items, each with an owner and a due date. People who missed the meeting will act on that output, so the cost of being wrong is real.

The system must take a team from **pilot** (no users, no traces, only a hand-built golden set) to **production** (traces flowing, new failure patterns appearing) without ever letting a wrong-but-green summary ship.

---

## 2. Background & problem

Teams routinely ship LLM features on intuition. Three failure patterns motivate this product:

1. **Benchmarks are not product quality.** A generic model score says nothing about whether *this* feature does the right thing for *this* user.
2. **System metrics crowd out outcome metrics.** Latency and cost are easy to log, so teams report them and skip the questions that matter. A fast, cheap summary that is wrong has a great system score and a terrible outcome score.
3. **Failures that read beautifully still ship.** An invented discount, a wrong owner, or a missed decision can pass every automated check if nobody has defined what "good" means in a runnable way.

The core insight: quality must be **defined from user outcomes and business risk**, named as **failure modes**, weighted by **severity**, and mapped to the **cheapest grader that can actually detect each one**.

---

## 3. Goals

| # | Goal |
|---|------|
| G1 | Define "good" from user outcomes and business risk, not from a model benchmark. |
| G2 | Turn raw observations of real outputs into a **named, severity-ranked list of failure modes** (the error-analysis step). |
| G3 | Build a **golden dataset** of hard inputs with known-good answers, versioned and held-out. |
| G4 | Provide a **ladder of graders** — code checks, LLM judge, agent judge, human review — each added because it sees what cheaper rungs cannot. |
| G5 | Turn failures into **automated checks** and use the cheapest grader that can detect each failure. |
| G6 | Gate releases on a **short, trusted set of blocking evals** and monitor the rest. |
| G7 | **Calibrate** machine judges against human labels, tracking false passes and false fails separately. |
| G8 | Make the suite **grow from real use**: every production miss becomes a new golden case and a new eval. |

### Non-goals

- Training or fine-tuning the underlying model.
- Generic public leaderboards or model comparisons.
- The end-user UI of the Meeting Summarizer itself.
- Agent-eval topics that do not apply to a read/write summarizer (autonomy tiers, escalation to a human, refund-style policy caps). These are noted for any future version that *takes real actions*.

---

## 4. Users & stakeholders

| Persona | What they need |
|---------|----------------|
| **Product manager** | A defensible answer to "is it good enough to ship?" and a severity-weighted quality trend. |
| **ML engineer** | A runnable suite that fires on every prompt/model change with clear pass/fail and failure attribution. |
| **Domain reviewer / expert** | A sampled review queue with the same rubric the machines use, ideally drawn from real users. |
| **On-call / release owner** | A short, trusted release gate; alerts that feed back into error analysis. |
| **Leadership** | Outcome evidence (right person, right task, right date) rather than raw latency/cost. |

---

## 5. Success metrics

| Metric | Target |
|--------|--------|
| Severity-weighted failure points (High=3, Medium=2, Low=1) | Monotonically decreasing release over release |
| Coverage of named failure modes by at least one grader | 100% |
| False-pass rate of LLM/agent judges vs. human labels | < 5% |
| Release-blocking evals in the gate | Small, trusted, stable (a curated subset) |
| Production failures converted to golden cases | 100% of confirmed misses |
| Judge-human agreement on held-out data | Tracked and not regressing |

> A prompt change that lowers raw pass counts while **raising** severity-weighted failure points is a regression, even if the summary "looks" fine.

---

## 6. Core concepts

- **Eval** — a repeatable test that tells you whether the feature behaves the way you'd stake your business on.
- **Failure mode** — a named, user-visible way the output can be wrong (e.g., "invented number or offer", "right task, wrong person").
- **Severity** — how much a failure hurts; used to weight the pass counts (High 3 / Medium 2 / Low 1).
- **Golden dataset** — curated inputs each paired with a human-decided correct answer, re-run on every change.
- **Reference-free grader** — a check that needs only the transcript and the summary, so it also runs on live traffic. Anything requiring a golden answer can only run on golden cases.
- **Metric types:**
  - **Outcome** — did the reader end up with the right result?
  - **Process** — did the pipeline do the right steps?
  - **System** — how fast and how cheap was it?

---

## 7. Functional requirements

### 7.1 Define good and rank the risks

- **FR-1.1** The system MUST capture success criteria in user terms. For the summarizer, "good" means:
  - **Faithful** — everything it says was said; no invented numbers, commitments, or names.
  - **Complete** — the decisions and every real action item are present, so nobody must open the transcript.
  - **Usable** — short enough to skim; each action has one real owner and a real date; fair to people named.
- **FR-1.2** The system MUST support **error analysis** on real outputs before any test is written:
  - Open coding: raw, uncategorized notes on everything that bothered a reader.
  - Axial coding: group notes into named failure modes.
- **FR-1.3** Each failure mode MUST record: name, **severity**, who is hurt and how, and where it was seen.
- **FR-1.4** Each failure mode MUST be convertible into a **testable sentence** that two people would agree is pass or fail. If two reviewers can't agree, the sentence is too vague and MUST be rewritten before any grader is built for it.
- **FR-1.5** The system MUST support **severity weighting** so a prompt change that fixes many Low failures but adds one High failure is reported as a regression.

*Reference failure modes (summarizer):* invented number/offer (High), misses the decision (High), right task/wrong person (High), action item with no real owner (Medium), wrong due date (Medium), unfair attribution of blame (Medium), too long to skim (Low).

### 7.2 Golden dataset

- **FR-2.1** The system MUST maintain a curated golden set: inputs paired with an answer a person decided is right, re-run on every change.
- **FR-2.2** Cases MUST be **written before seeing the model output**, so reviewers aren't anchored.
- **FR-2.3** Cases MUST be **stratified** across meeting type and risk, deliberately over-covering rare high-stakes categories rather than sampling at random.
- **FR-2.4** The set MUST seed **edge and adversarial** cases: prompt injection, messy audio, "no decisions at all", ambiguous commitments.
- **FR-2.5** The golden set MUST be **versioned**; when a golden answer changes, the reason is recorded.
- **FR-2.6** The set MUST include a **held-out slice** used only for checking, never for tuning prompts or rubrics.
- **FR-2.7** The set MUST grow: every confirmed production failure becomes a case and (usually) a regression test.
- **FR-2.8** The set MUST be large enough that one extra failure does not change the conclusion (the reference pilot uses 6 meetings × 2 prompt versions = 12 summaries purely so a reader can inspect every transcript; production sets are larger).

### 7.3 Graders

The system MUST provide four grader types. Each is added because it sees something the cheaper ones cannot.

#### 7.3.1 Code checks (rules that never get tired)
- **FR-3.1.1** Small functions returning pass/fail; deterministic, near-zero cost, run on **every change** and **all live traffic**.
- **FR-3.1.2** MUST support: format/schema validation, lookups (owner in attendee list), grounding by string match (each number/name appears in source), hard limits/guardrails (length, banned phrases, leaked data), and reference match against the golden answer.
- **FR-3.1.3** The system MUST be explicit that **a green code check is not a good summary** — passing rules is the entry ticket, not the verdict.

#### 7.3.2 LLM judge (meaning that rules can't see)
- **FR-3.2.1** A second model reads the transcript + summary and answers a **written rubric** (e.g., Faithful / Complete / Fair), each as a **yes/no question** with a clear definition, reasoning before the verdict.
- **FR-3.2.2** The rubric MUST be precise; a **binary (yes/no)** default is expected, with anchored scales (1–5 with a written meaning per point) as a legitimate alternative.
- **FR-3.2.3** One criterion at a time; broad "quality" verdicts are not allowed as the sole measure.
- **FR-3.2.4** Output MUST be structured (e.g., JSON) with reasoning, and MUST be **calibrated against human labels before it is trusted**.

#### 7.3.3 Agent judge (checks that need investigation)
- **FR-3.3.1** A tool-using model MAY search the transcript, read exact lines, and resolve relative dates, iterating until it can give a verdict.
- **FR-3.3.2** The agent MUST work from the transcript (so it runs on **live meetings with no golden answer**) and is expected to be used as the expensive specialist (e.g., action items only).
- **FR-3.3.3** The system MUST grade the agent's **trajectory**, not only its final verdict: tool-call correctness, ordering, and **cited evidence** for every verdict.
- **FR-3.3.4** Where a future version causes state changes (e.g., creating tracker tasks), the system MUST verify the **state change itself**, not the agent's claim.

#### 7.3.4 Human judge (what "good" really means)
- **FR-3.4.1** Humans MUST do three jobs no automated grader can do for itself:
  1. **Calibrate** the LLM/agent judges by labelling a sample.
  2. **Spot** problems nobody has written a rule for yet.
  3. **Decide** where the quality bar sits.
- **FR-3.4.2** Reviewers MUST work from the **same rubric** as the machines; if two reviewers disagree, the criterion is too vague for any grader.
- **FR-3.4.3** Review MUST be **sampled** (e.g., a few dozen per release, including borderline scores and grader disagreements), never everything.
- **FR-3.4.4** The system MUST measure **false-pass and false-fail rates separately** and tolerate far fewer false passes than false fails.

### 7.4 Match each failure to the cheapest grader

- **FR-4.1** Every named failure mode MUST be mapped to the **cheapest grader that can detect it on live traffic** (no reference answer available), per the ladder:
  1. **Code check** — facts, format, lookups, counts, string matches.
  2. **LLM judge** — judgement about meaning.
  3. **Agent judge** — lookups and multi-step investigation.
  4. **Human judge** — the unknown and the calibration of all rungs below.
- **FR-4.2** A **golden-answer reference check** is a bonus where an answer exists; it MUST NOT be the only protection for a failure mode that also appears on live traffic.
- **FR-4.3** The system MUST NOT default to the smartest grader for everything: it is slower, costlier, and can be wrong without telling you. "A code check that fails is a fact; an LLM score is an opinion."

*Reference mapping (summarizer):*

| Failure | Severity | Lead grader |
|---------|----------|-------------|
| Invented number or offer | High | Code check (+ LLM judge) |
| Misses the decision | High | LLM judge |
| Right task, wrong person | High | Agent judge (+ golden code check) |
| Action item with no real owner | Medium | Code check |
| Wrong due date | Medium | Agent judge |
| Unfair attribution of blame | Medium | LLM judge (+ Human) |
| Too long to skim | Low | Code check |

### 7.5 Eval catalogue

- **FR-5.1** The system MUST maintain a working catalogue of evals, each tied to a failure mode and carrying: **grader, pass bar, owner, and a defined action on failure** ("an eval without an action is a number nobody uses").
- **FR-5.2** The catalogue MUST group evals into tiers: **Faithfulness, Completeness, Action items, Format & usability, Safety/tone/fairness, Robustness, System, Outcome-in-production**.
- **FR-5.3** Only a **small subset** MUST be release-blocking; the rest are monitors that raise alerts and feed error analysis.
- **FR-5.4** Pass bars MUST be starting points, set from measured human agreement and baseline, then tightened as the product matures.
- **FR-5.5** The reference catalogue (32 evals) is reproduced in [Appendix A](#appendix-a--reference-eval-catalogue).

### 7.6 Run, read, fix, repeat

- **FR-6.1** The full suite MUST run over the whole golden set on **every prompt/model change**, producing pass counts per grader and a **severity-weighted failure-point total**.
- **FR-6.2** The primary output MUST be the **list of failures, which grader found each one, and how much each matters** — not just pass counts.
- **FR-6.3** The workflow MUST be: **Run → Read → Fix → Add** (turn each new failure into a permanent golden case).
- **FR-6.4** Agreement numbers from a rubric tuned on the same cases it is measured on MUST be flagged as optimistic; confirmation requires a **held-out / blind** set.
- **FR-6.5** The system MUST surface the dangerous case where **every automated grader is green and a human still rejects the output** (e.g., unfair blame passing all machines).

### 7.7 Monitor and recalibrate in production

- **FR-7.1** Once real usage exists, MUST log traces and run **error analysis on real traces** to find new failure modes.
- **FR-7.2** Reference-free graders MUST run on live traffic (all traffic for hard limits; a defined sample for the rest).
- **FR-7.3** The system MUST alert on outcome drift, e.g.:
  - user edit rate on action items rising above baseline,
  - weekly human audit "ready to send" rate falling,
  - thumbs-down / complaint rate rising.
- **FR-7.4** LLM and agent judges MUST be **recalibrated on a schedule** against fresh human labels, since they drift as traffic changes.
- **FR-7.5** Release gating MUST be repeated on a regular cadence **and before every release**.

---

## 8. Non-functional requirements

| # | Requirement |
|---|-------------|
| NFR-1 | **Cheapest-grader-first.** Prefer reference-free graders wherever a failure can be covered without an answer key. |
| NFR-2 | **Determinism for facts.** Code checks return the same answer every run. |
| NFR-3 | **Evidence.** Agent (and ideally judge) verdicts cite the transcript lines they relied on. |
| NFR-4 | **Separation of error direction.** False passes and false fails are always reported separately. |
| NFR-5 | **Immutability of held-out data.** Tuning must never touch the held-out slice. |
| NFR-6 | **Versioning.** Prompts, rubrics, golden answers, and eval definitions are all versioned with recorded reasons. |
| NFR-7 | **Auditability.** A reviewer can check any machine verdict in seconds from its cited evidence. |
| NFR-8 | **Actionability.** Every eval has a named owner and a defined action when it fires. |

---

## 9. Rollout plan (pilot → production)

| Stage | State | What happens |
|-------|-------|--------------|
| **1. Define good, rank risks** | No users, no traces | Write success criteria and a severity-ranked failure list. |
| **2. Build the golden set** | Sample transcripts only | Hand-build hard, stratified, adversarial cases with known-good answers. |
| **3. Add code checks** | Golden set exists | Rules + the golden-answer check. |
| **4. Add an LLM judge** | Golden set exists | Rubric-based meaning checks, calibrated against human labels. |
| **5. Add an agent judge** | Golden set exists | Multi-step verification of action items. |
| **6. Pilot gate** | Golden set is all you have | Go / no-go decision to let real users in. |
| **7. Launch, log traces** | Real users arrive | Start capturing usage. |
| **8. Error analysis on traces** | Traces flowing | Find failure modes nobody predicted. |
| **9. New patterns → new evals** | Traces flowing | Grow the suite from real use. |
| **10. Monitor live traffic** | Production | Reference-free graders on live samples. |
| **11. Recalibrate judges** | Production | Fresh human labels on a schedule. |
| **12. Release gate, repeat** | Production | Gate every release, then go round again. |

---

## 10. Risks & mitigations

| Risk | Mitigation |
|------|------------|
| Passing code checks taken as proof of quality | Explicitly treat green rules as an entry ticket; add meaning graders. |
| Over-trusting an LLM judge | Calibrate against human labels; track false-pass rate; add a human-in-the-loop for the unknown. |
| Tuning on the test set ("studying with the exam paper") | Keep a held-out slice; run a blind check. |
| Severity-blind pass counting | Weight failures by severity; watch the weighted total, not raw pass rate. |
| Too many release gates, so red gets ignored | Keep the blocking set short and trusted; everything else is a monitor. |
| Live traffic has no answer key | Cover every important failure with at least one reference-free grader. |
| Small golden sets give noisy scores | Grow the set so one extra failure can't swing the conclusion. |
| Unfair/harmful framing invisible to machines | Human review is mandatory for tone/fairness and calibrates the LLM judge. |

---

## 11. Open questions

1. What is the target size and composition of the production golden set, and who owns curation?
2. Which pass bars are release-blocking vs. advisory in v1? (Reference: 13 of 32 block release.)
3. Which LLM/agent judge model(s) are approved, and what is the cost/latency budget per meeting?
4. What is the human-review cadence and headcount (reference: 30 summaries/week)?
5. How is the "ready to send" human bar defined and audited across reviewers?
6. What trace/logging backend is used, and what is the retention/PII policy for sensitive meetings?
7. When (if ever) does the summarizer take real actions, triggering state-change verification and policy caps?

---

## Appendix A — Reference eval catalogue

32 evals for the Meeting Summarizer, each tied to a failure mode, grader, and pass bar. **13 block release.** Bars are practitioner starting points, not figures from any real product; adjust to your own data.

### Faithfulness — nothing in the output that wasn't said

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E01 | Numbers are grounded | High · blocks | Code check | 100% — any failure blocks |
| E02 | Names are grounded | Medium | Code check | 100% |
| E03 | No unauthorised offers/promises | High · blocks | Code check (+ LLM) | Zero tolerance |
| E04 | Claims are supported | High · blocks | LLM judge | ≥97% yes, false-pass <5% |
| E05 | Commitments were really made | High · blocks | Agent judge | 100% of sampled items |

### Completeness — nothing important left out

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E06 | Decision stated with figures | High · blocks | LLM judge | ≥95% yes |
| E07 | Action-item match to golden answer | High · blocks | Code check | No missing High items; recall ≥95% |
| E08 | Missed commitments in live traffic | High | Agent (+ Human) | Miss rate <3%, not rising |
| E09 | "Nothing decided" handled | Medium | LLM judge (+ Code) | 100% on golden cases |

### Action items — right person, right task, right date

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E10 | Action items are complete | Medium · blocks | Code check | 100% |
| E11 | Owner is an attendee | Medium · blocks | Code check | 100% |
| E12 | Owner is the real committer | High · blocks | Agent (+ Code) | ≥98% of sampled items |
| E13 | Due date resolved correctly | Medium | Agent (+ Code) | ≥98% |
| E14 | Conditions are preserved | Medium | LLM judge | ≥95% on conditional cases |
| E15 | No invented owner for vague asks | Medium | Agent (+ Code) | 100% on golden cases |

### Format & usability

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E16 | Length limit (≤80 words) | Low | Code check | ≥99% within limit |
| E17 | No chatbot filler | Low | Code check | 100% |
| E18 | Skimmable in 30 seconds | Low | Human judge | ≥90% rated skimmable |

### Safety, tone & fairness

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E19 | Fair attribution | Medium · blocks | LLM judge (+ Human) | False-pass <5% |
| E20 | Neutral, professional tone | Low | LLM judge | ≥97% yes |
| E21 | Sensitive information stays out | High · blocks | Code check (+ LLM) | Zero tolerance |
| E22 | Prompt-injection resistance | High · blocks | Code check (+ LLM) | 100% on adversarial cases |

### Robustness — awkward real-world inputs

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E23 | Long meetings (>1h) | Medium | Agent (+ LLM) | Recall within 5 pts of short |
| E24 | Messy transcripts | Medium | LLM judge (+ Agent) | Within 5 pts of clean |
| E25 | Mixed / non-English meetings | Medium | Human judge (+ LLM) | Within 5 pts of English |

### System — fast, cheap, reliable

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E26 | Output parses | Medium · blocks | Code check | ≥99.9% parse |
| E27 | Latency (p95) | Low | Code check | User-research target + alert |
| E28 | Cost per meeting | Low | Code check | Within budget + alert on jump |

### Outcome in production — what real use tells you

| ID | Eval | Severity | Lead grader | Pass bar |
|----|------|----------|-------------|----------|
| E29 | User edit rate on action items | High | Code check (+ Human) | Alert above baseline |
| E30 | Weekly human audit | High | Human judge | ≥95% ready to send |
| E31 | Judge calibration drift | High | Human (+ LLM/Agent) | False-pass <5%; recalibrate |
| E32 | Thumbs-down and complaints | Medium | Code check (+ Human) | Alert on rise; tags feed error analysis |

### How to run the catalogue

- **Five things every eval needs:** the failure it guards against, the grader, a pass bar, an owner, and a defined action on failure.
- **Few evals block a release** (13 here). Keep the gate short and trusted; gate on everything and people start ignoring red.
- **Prefer reference-free graders.** Anything needing a golden answer can only run on golden cases; live traffic has no answer key.
- **Set bars from data**, then tighten as the product matures.
- **Every production miss becomes a case** and usually a new eval — decide which grader *should* have caught it.
- **Re-calibrate judges on a schedule** (E31) with fresh human labels.

---

## Appendix B — Glossary

| Term | Meaning |
|------|---------|
| Eval | Repeatable test of whether a feature behaves correctly. |
| Failure mode | Named, user-visible way output can be wrong. |
| Severity weighting | High=3, Medium=2, Low=1, applied to failures. |
| Golden dataset | Curated hard inputs with human-decided correct answers. |
| Reference-free | Grader needing only transcript + summary; runs on live traffic. |
| Outcome / process / system metric | Right result / right steps / speed & cost. |
| False pass | Judge approves what a human rejects (the dangerous one). |
| False fail | Judge rejects something fine (wastes time). |
| Held-out slice | Cases kept out of tuning to honestly check results. |
| Trajectory | The full path an agent judge took, not just its verdict. |

---

## Appendix C — Sources

- Evals Class, *Meeting summarizer* case study — <https://evals-for-everyone.vercel.app/meeting-summarizer/foundations>
- Companion pages: Error analysis, Golden dataset, Code checks, LLM judge, Agent judge, Human judge, Pick graders, Run & iterate, Summary, Evals list, How it works, Quiz.

> All data on the source site is synthetic and generated for teaching. In the meeting summarizer case study, code checks run live while LLM, agent, and human verdicts are scripted.
