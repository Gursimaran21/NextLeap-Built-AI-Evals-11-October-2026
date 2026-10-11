<!--
⚠️ DEMO / ILLUSTRATIVE ONLY — every entry below is fabricated for demonstration
(consistent with "Eval run report - Example filled.md": VAL-003 validation run → fixes
→ blind re-run VAL-004 GREEN). Replace all values with real results before submitting.
Delete this banner too.
-->

# Release Gate Checklist

**Purpose.** Turn the release-gate thresholds from [PRD § 11 — Metrics & thresholds](./PRD.md#11-metrics--thresholds--the-release-gate) into a scored, fill-in-the-blanks gate that a release owner can complete in one sitting.

**When to use.** Before **every release** and after **every prompt/model/rubric change**. The suite runs over the **blind (Test) split** — 40 rows — via the n8n workflow in [n8n-eval-workflow.md](./n8n-eval-workflow.md).

**How the numbers are produced.** Run the n8n **Evaluation**, then read off per-row verdicts (`faithful` / `complete` / `fair` from the LLM judge; parse/format/limits from code checks). A row **passes a dimension** when the judge and any applicable code check agree. Pass rate = `rows passing / rows in blind set`. Weighted failure points = `High×3 + Medium×2 + Low×1`.

**Scoring.** Mark every applicable gate item ✅ PASS or ❌ FAIL. The release is **GREEN only if every applicable item is PASS**. Any ❌ ⇒ **RED — do not ship.** If a gate item lands **within 1 failed row of its bar**, it is **inconclusive** — re-run once (no cherry-picking) or add blind rows before shipping.

> Once the gate has a verdict, copy the numbers and failure analysis into the [**Eval run report**](./Eval%20run%20report.md) — the completed-run record for the PMF submission.

---

## Run template (fill in the blanks)

| Field | Value |
|-------|-------|
| Candidate (prompt/model version or commit) | **prompt v3.1 (fixes from VAL-003) · model v0.3 · commit `def4567`** |
| Judge model & version | **LLM judge `gpt-4o-mini` · rubric v2** |
| Rubric version | **v2 (Faithful / Complete / Fair, JSON verdict)** |
| Golden set version & blind split size | **v1.1 · 40 rows (Test split)** |
| Previous release weighted failure points | **6** |
| Run date & time | **2026-10-12 · 09:45** |
| Run by | **Gursimaran (PM) + ML engineer** |
| n8n Evaluation run ID / link | **VAL-004 · [n8n.cloud/workflows/…/evaluations/VAL-004](https://example.invalid/val-004)** |

---

## Scored gate

| # | Gate item (dimension) | Bar (v1, blind set of *40*) | Evidence to attach | Result |
|---|-----------------------|-----------------------------|--------------------|--------|
| 1 | **Faithful** | ≥ 95% pass (≤ 2 rows fail) **and** zero High-severity failures (invented number/offer, unsupported claim, wrong commitment — E01/E03/E04/E05) | Judge export + failing cases | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 2 | **Complete** | ≥ 90% pass (≤ 4 rows fail) **and** no missing High decision/action; action-item recall ≥ 95% (E06–E10) | Judge export + recall report | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 3 | **Fair** | ≥ 95% pass **and** judge false-pass vs human labels < 5% (E19) | Judge export + human sample labels | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 4 | **Format & limits** | 100% parse; ≥ 99% of summaries ≤ 80 words; zero banned/filler phrases (E16/E17/E26) | Code-check report | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 5 | **Robustness** | Long / messy / non-English (E23–E25) within 5 points of the clean baseline | Per-slice pass rates | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 6 | **Weighted failure points** | ≤ previous release — never regress | Total vs. previous (field above) | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 7 | **Judge calibration** | False-pass < 5% on the fresh human sample (E31) | Calibration report | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 8 | **Blind set integrity** | Re-run on this **exact** candidate; no rows added/removed; no tuning on blind rows since last run | Commit diff of blind CSV, run metadata | **☑ PASS** ⬜ FAIL ⬜ N/A |
| 9 | **Gate hygiene** | Drift monitors (E08, E29–E32) reviewed; any alerts triaged | Monitor screenshots / notes | ⬜ PASS ⬜ FAIL **☑ N/A (monitors go live at v2)** |

### Observed numbers (fill in)

| Metric | Observed | Bar | Status |
|--------|----------|-----|--------|
| Faithful | **38 / 40 (95.0%)** | ≥ 95% | **☑ PASS** ⬜ FAIL |
| Complete | **36 / 40 (90.0%)** | ≥ 90% | **☑ PASS** ⬜ FAIL |
| Fair | **39 / 40 (97.5%)** | ≥ 95% | **☑ PASS** ⬜ FAIL |
| Format & limits | **40 / 40 parse (100%) · 40 / 40 ≤ 80 words (100%)** | 100% parse · ≥ 99% ≤ 80 words | **☑ PASS** ⬜ FAIL |
| Robustness vs clean | Long **1/1** · Messy **1/1** · Non-English **1/1** (Δ **0** pts) | within 5 pts | **☑ PASS** ⬜ FAIL |
| Weighted failure points | **4** | ≤ previous (**6**) | **☑ PASS** ⬜ FAIL |
| Judge false-pass rate | **3.0%** (1 / 33 fresh human rows) | < 5% | **☑ PASS** ⬜ FAIL |

### Failures found (feed error analysis)

| Test id | Eval | Failure | Severity | Root cause (guessed) | New golden case? |
|---------|------|---------|----------|----------------------|------------------|
| E13-02 | Due date resolved correctly | "due Friday" left as meeting date where Friday is a weekend later in the month | Med | date-resolver weekend edge | Yes |
| E18-03 | Skimmable in 30 seconds | one list-heavy summary scored low on skim | Low | dense bullet formatting | Yes |
| E20-01 | Neutral, professional tone | one idiom slightly informal ("killed it") | Low | tone calibration | Yes |

**3 failing rows · weighted 4 (1 Med × 2 + 2 Low × 1) · 0 High.** All three became new golden cases (FR-2.7); nothing the gold set missed.

---

## Verdict

- Gate items failed (❌): **0** of **8** applicable (1 N/A)
- Weighted failure points this run: **4** (previous: **6** → improved)
- **Overall: ☑ GREEN — ship** ⬜ RED — do not ship ⬜ INCONCLUSIVE (within margin — re-run)

Sign-off (release owner): **ML lead**  Date: **2026-10-12**

---

## Before you ship (final checks)

- [x] Blind (Test) set rows were **not** used to tune the prompt, rubric, or thresholds (PRD § 11.5).
- [x] Every confirmed failure from this run was turned into a golden case and (where possible) a regression eval (FR-2.7).
- [x] Gate run is logged with prompt/rubric/golden/judge versions (NFR-6).
- [x] Human sample reviewed the **Fair** dimension — automated green alone never clears fairness (FR-6.5).

---

*Example (illustrative only — delete before use): Faithful 38/40 = 95% ✅ · Complete 36/40 = 90% ✅ · Fair 39/40 = 97.5% ✅ · Weighted points 4 ≤ previous 6 ✅ · False-pass 3% ✅ ⇒ GREEN.*