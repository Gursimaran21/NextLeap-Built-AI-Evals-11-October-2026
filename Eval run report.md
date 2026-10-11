# Eval Run Report — Meeting Summarizer

**Purpose.** A completed-run record for the **NextLeap PMF submission**: paste the numbers from an n8n Evaluation into this template, add the failure analysis, and you have a defensible "here is how we know this summarizer is (or isn't) good enough" artifact.

**Companion files.** Methodology & thresholds: [PRD § 11](./PRD.md#11-metrics--thresholds--the-release-gate) · How to run in n8n: [n8n-eval-workflow.md](./n8n-eval-workflow.md) · Go/no-go gate: [Release gate checklist.md](./Release%20gate%20checklist.md) · Data: `Test cases - Golden set.csv` (200, split 40/120/40).

---

## 1. Run metadata (fill in)

| Field | Value |
|-------|-------|
| Run ID | ____________________ |
| Date & time | ____________________ |
| Run by | ____________________ |
| Purpose of this run | ⬜ Baseline ⬜ Prompt change ⬜ Model change ⬜ Rubric change ⬜ Release gate ⬜ Other: ______ |
| Candidate version (prompt/model/commit) | ____________________ |
| Judge model & version | ____________________ |
| Rubric version | ____________________ |
| Golden set version | ____________________ |
| Split used & rows run | ⬜ Validation (120) ⬜ Test/Blind (40) ⬜ Both ⬜ Other: ______ (rows: ____) |
| n8n Evaluation run link / export | ____________________ |

---

## 2. Executive summary

**Headline verdict:** ⬜ Looks good to ship ⬜ Not yet — fix and re-run ⬜ Baseline — numbers to beat

In one paragraph, state what was evaluated, how it performed overall, and the biggest risk found:

> ____________________________________________________________________
> ____________________________________________________________________
> ____________________________________________________________________

**Key numbers (fill in):**

| Metric | Value |
|--------|-------|
| Faithful pass rate | ____ / ____ (____%) |
| Complete pass rate | ____ / ____ (____%) |
| Fair pass rate | ____ / ____ (____%) |
| Weighted failure points (High×3 + Med×2 + Low×1) | ________ |
| High-severity failures | ____ |
| Judge false-pass rate (vs human sample) | ____% |

---

## 3. Methodology (short form)

- **What was graded:** summaries produced by the Meeting Summarizer agent over the golden set transcripts.
- **Dataset:** 200 rows across 25 evals (E01–E25) × 8 case types, split Train 40 / Validation 120 / Test 40 (stratified per eval). This run covers the split noted in § 1.
- **Judges used (tick all that applied):** ⬜ Code checks (parse, ≤80 words, banned phrases, grounded numbers/names, owner-in-attendees) ⬜ LLM judge `{faithful, complete, fair}` ⬜ Agent judge (search/investigate) ⬜ Human review sample (____ rows)
- **Thresholds:** PRD § 11.2 — Faithful ≥ 95% · Complete ≥ 90% · Fair ≥ 95% · 100% parse · weighted points never regress.
- System/production monitors (E26–E32) are not part of the golden-set run; they are tracked in live traffic.

---

## 4. Results — aggregate by dimension

| Dimension | Rows passed | Rows run | Rate | Bar | Status |
|-----------|------------:|---------:|-----:|-----|:------:|
| Faithful | ____ | ____ | ____% | ≥ 95% | ⬜ ✅ ⬜ ❌ |
| Complete | ____ | ____ | ____% | ≥ 90% | ⬜ ✅ ⬜ ❌ |
| Fair | ____ | ____ | ____% | ≥ 95% | ⬜ ✅ ⬜ ❌ |
| Format & limits | ____ | ____ | ____% | 100% parse · ≥99% ≤80 wds | ⬜ ✅ ⬜ ❌ |
| Robustness (E23–E25, Δ vs clean) | Long ____ · Messy ____ · Non-English ____ | ____ | ____% | within 5 pts | ⬜ ✅ ⬜ ❌ |
| **Weighted failure points** | — | — | — | ≤ previous (________) | ⬜ ✅ ⬜ ❌ |

---

## 5. Results — per eval (E01–E25)

Fill from the n8n per-eval breakdown. 💥 = blocked-release evals (13: E01, E03, E04, E05, E06, E07, E10, E11, E12, E19, E21, E22, E26 per PRD Appendix A).

| ID | Eval | Pass | Fail | Rate | Bar | Status | Notes |
|----|------|-----:|-----:|-----:|-----|:------:|-------|
| E01 | Numbers are grounded 💥 | ____ | ____ | ____% | 100% | | |
| E02 | Names are grounded | ____ | ____ | ____% | 100% | | |
| E03 | No unauthorised offers/promises 💥 | ____ | ____ | ____% | Zero | | |
| E04 | Claims are supported 💥 | ____ | ____ | ____% | ≥97% | | |
| E05 | Commitments were really made 💥 | ____ | ____ | ____% | 100% | | |
| E06 | Decision stated with figures 💥 | ____ | ____ | ____% | ≥95% | | |
| E07 | Action-item match to golden answer 💥 | ____ | ____ | ____% | ≥95% rec | | |
| E08 | Missed commitments in live traffic | ____ | ____ | ____% | <3% miss | | |
| E09 | "Nothing decided" handled | ____ | ____ | ____% | 100% | | |
| E10 | Action items are complete 💥 | ____ | ____ | ____% | 100% | | |
| E11 | Owner is an attendee 💥 | ____ | ____ | ____% | 100% | | |
| E12 | Owner is the real committer 💥 | ____ | ____ | ____% | ≥98% | | |
| E13 | Due date resolved correctly | ____ | ____ | ____% | ≥98% | | |
| E14 | Conditions are preserved | ____ | ____ | ____% | ≥95% | | |
| E15 | No invented owner for vague asks | ____ | ____ | ____% | 100% | | |
| E16 | Length limit (≤80 words) | ____ | ____ | ____% | ≥99% | | |
| E17 | No chatbot filler | ____ | ____ | ____% | 100% | | |
| E18 | Skimmable in 30 seconds | ____ | ____ | ____% | ≥90% | | |
| E19 | Fair attribution 💥 | ____ | ____ | ____% | F-p <5% | | |
| E20 | Neutral, professional tone | ____ | ____ | ____% | ≥97% | | |
| E21 | Sensitive information stays out 💥 | ____ | ____ | ____% | Zero | | |
| E22 | Prompt-injection resistance 💥 | ____ | ____ | ____% | 100% | | |
| E23 | Long meetings (>1h) | ____ | ____ | ____% | ±5 pts | | |
| E24 | Messy transcripts | ____ | ____ | ____% | ±5 pts | | |
| E25 | Mixed / non-English meetings | ____ | ____ | ____% | ±5 pts | | |

---

## 6. Results — by case type

Shows whether the hard cases are where the failures are (paste from n8n).

| Case type | Pass | Fail | Rate | Observations |
|-----------|-----:|-----:|-----:|--------------|
| happy_path | ____ | ____ | ____% | |
| negative | ____ | ____ | ____% | |
| edge_boundary | ____ | ____ | ____% | |
| ambiguous_or_incomplete | ____ | ____ | ____% | |
| adversarial | ____ | ____ | ____% | |
| noisy_input | ____ | ____ | ____% | |
| multi_turn_or_context | ____ | ____ | ____% | |
| tool_or_agent_failure | ____ | ____ | ____% | |

---

## 7. Failure analysis (every fail → a fix and a test)

| Test id | Eval | What went wrong | Severity | Expected (gold) vs actual | Root cause | Fix + new golden case? |
|---------|------|-----------------|:--------:|---------------------------|------------|------------------------|
| ________ | ________ | ________________ | High/Med/Low | ____________________ | ________ | ✅ 🔁 ⬜ |
| ________ | ________ | ________________ | High/Med/Low | ____________________ | ________ | ✅ 🔁 ⬜ |

Patterns observed across failures (e.g., "all injection fails are paraphrase variations", "dates break only for non-English meetings"):

> ____________________________________________________________________

---

## 8. Judge calibration (do not skip the first few runs)

| Measure | Value | Target |
|---------|------:|:------:|
| Human-labelled rows in sample | ____ | ≥ 20 |
| Judge–human agreement | ____% | tracked |
| **False-pass rate** (judge ✅, human ❌) | ____% | < 5% |
| False-fail rate (judge ❌, human ✅) | ____% | tracked low |

---

## 9. Conclusions & recommendations

1. Did any dimension miss its bar, and why (list the top causes from § 7)?
2. What is the highest-severity remaining risk, and who owns it?
3. Recommended next step: ⬜ Ship (gate green) ⬜ Fix top failures → re-run ⬜ Grow the golden set (add new cases) ⬜ Recalibrate judge (false-pass ≥ 5%) ⬜ Other: ______
4. What would make the next run more conclusive (more evals, more edge cases, bigger blind set)?

---

## 10. Sign-off

| Role | Name | Verdict | Date |
|------|------|---------|------|
| Run owner | ________ | ⬜ Ship ⬜ Fix ⬜ Re-run | ________ |
| Release owner | ________ | ⬜ Ship ⬜ Fix ⬜ Re-run | ________ |
| Human reviewer (Fair sample) | ________ | ⬜ Agree ⬜ Disagree | ________ |

*Report generated from `Test cases - Golden set.csv` (v____) · n8n run ______ · Checklist verdict: ⬜ GREEN ⬜ RED ⬜ INCONCLUSIVE.*