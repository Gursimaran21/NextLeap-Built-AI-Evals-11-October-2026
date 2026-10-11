# Release Gate Checklist

**Purpose.** Turn the release-gate thresholds from [PRD § 11 — Metrics & thresholds](./PRD.md#11-metrics--thresholds--the-release-gate) into a scored, fill-in-the-blanks gate that a release owner can complete in one sitting.

**When to use.** Before **every release** and after **every prompt/model/rubric change**. The suite runs over the **blind (Test) split** — 40 rows — via the n8n workflow in [n8n-eval-workflow.md](./n8n-eval-workflow.md).

**How the numbers are produced.** Run the n8n **Evaluation**, then read off per-row verdicts (`faithful` / `complete` / `fair` from the LLM judge; parse/format/limits from code checks). A row **passes a dimension** when the judge and any applicable code check agree. Pass rate = `rows passing / rows in blind set`. Weighted failure points = `High×3 + Medium×2 + Low×1`.

**Scoring.** Mark every applicable gate item ✅ PASS or ❌ FAIL. The release is **GREEN only if every applicable item is PASS**. Any ❌ ⇒ **RED — do not ship.** If a gate item lands **within 1 failed row of its bar**, it is **inconclusive** — re-run once (no cherry-picking) or add blind rows before shipping.

---

## Run template (fill in the blanks)

| Field | Value |
|-------|-------|
| Candidate (prompt/model version or commit) | ____________________ |
| Judge model & version | ____________________ |
| Rubric version | ____________________ |
| Golden set version & blind split size | ____________________ (default: 40) |
| Previous release weighted failure points | ____________________ |
| Run date & time | ____________________ |
| Run by | ____________________ |
| n8n Evaluation run ID / link | ____________________ |

---

## Scored gate

| # | Gate item (dimension) | Bar (v1, blind set of *40*) | Evidence to attach | Result |
|---|-----------------------|-----------------------------|--------------------|--------|
| 1 | **Faithful** | ≥ 95% pass (≤ 2 rows fail) **and** zero High-severity failures (invented number/offer, unsupported claim, wrong commitment — E01/E03/E04/E05) | Judge export + failing cases | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 2 | **Complete** | ≥ 90% pass (≤ 4 rows fail) **and** no missing High decision/action; action-item recall ≥ 95% (E06–E10) | Judge export + recall report | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 3 | **Fair** | ≥ 95% pass **and** judge false-pass vs human labels < 5% (E19) | Judge export + human sample labels | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 4 | **Format & limits** | 100% parse; ≥ 99% of summaries ≤ 80 words; zero banned/filler phrases (E16/E17/E26) | Code-check report | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 5 | **Robustness** | Long / messy / non-English (E23–E25) within 5 points of the clean baseline | Per-slice pass rates | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 6 | **Weighted failure points** | ≤ previous release — never regress | Total vs. previous (field above) | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 7 | **Judge calibration** | False-pass < 5% on the fresh human sample (E31) | Calibration report | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 8 | **Blind set integrity** | Re-run on this **exact** candidate; no rows added/removed; no tuning on blind rows since last run | Commit diff of blind CSV, run metadata | ⬜ PASS ⬜ FAIL ⬜ N/A |
| 9 | **Gate hygiene** | Drift monitors (E08, E29–E32) reviewed; any alerts triaged | Monitor screenshots / notes | ⬜ PASS ⬜ FAIL ⬜ N/A (monitors only) |

### Observed numbers (fill in)

| Metric | Observed | Bar | Status |
|--------|----------|-----|--------|
| Faithful | ____ / 40 (____%) | ≥ 95% | ⬜ PASS ⬜ FAIL |
| Complete | ____ / 40 (____%) | ≥ 90% | ⬜ PASS ⬜ FAIL |
| Fair | ____ / 40 (____%) | ≥ 95% | ⬜ PASS ⬜ FAIL |
| Format & limits | ____ / 40 (____%) | 100% parse · ≥ 99% ≤ 80 words | ⬜ PASS ⬜ FAIL |
| Robustness vs clean | Long ____ · Messy ____ · Non-English ____ (Δ vs clean) | within 5 pts | ⬜ PASS ⬜ FAIL |
| Weighted failure points | ________ | ≤ previous (________) | ⬜ PASS ⬜ FAIL |
| Judge false-pass rate | ____% | < 5% | ⬜ PASS ⬜ FAIL |

### Failures found (feed error analysis)

| Test id | Eval | Failure | Severity | Root cause (guessed) | New golden case? |
|---------|------|---------|----------|----------------------|------------------|
| ________ | ________ | ________ | High/Med/Low | ________ | Yes/No |
| ________ | ________ | ________ | High/Med/Low | ________ | Yes/No |

---

## Verdict

- Gate items failed (❌): ____ of ____ applicable
- Weighted failure points this run: ________ (previous: ________)
- **Overall: ⬜ GREEN — ship ⬜ RED — do not ship ⬜ INCONCLUSIVE (within margin — re-run)**

Sign-off (release owner): ____________________  Date: ____________________

---

## Before you ship (final checks)

- [ ] Blind (Test) set rows were **not** used to tune the prompt, rubric, or thresholds (PRD § 11.5).
- [ ] Every confirmed failure from this run was turned into a golden case and (where possible) a regression eval (FR-2.7).
- [ ] Gate run is logged with prompt/rubric/golden/judge versions (NFR-6).
- [ ] Human sample reviewed the **Fair** dimension — automated green alone never clears fairness (FR-6.5).

---

*Example (illustrative only — delete before use): Faithful 38/40 = 95% ✅ · Complete 36/40 = 90% ✅ · Fair 39/40 = 97.5% ✅ · Weighted points 4 ≤ previous 6 ✅ · False-pass 3% ✅ ⇒ GREEN.*