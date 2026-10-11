# Submission Cover Page — NextLeap: Built AI Evals

**Reference app:** Meeting Summarizer (LLM feature) · **Project:** Product Management Fundamentals (PMF) · **Date:** 11 October 2026 · **Author:** Gursimaran21

> **One line:** this repo builds a *run-your-own-evals* system for an AI Meeting Summarizer — a golden set, a clean train/validation/test split, few-shot examples, an n8n evaluation workflow, release-gate thresholds, and the reports that turn a run into a ship/no-ship decision.

An **eval** is a PRD for AI behaviour: it turns *"behaves correctly"* into a form you can **run** on every prompt/model change, so the team answers *"did this change make things better or worse?"* with evidence instead of vibes.

---

## How this repo answers the exercise

| Exercise step | What you'll find here |
|---------------|-----------------------|
| 1. Pick a product & an LLM feature | [`ProductBrief.md`](./ProductBrief.md) — Meeting Summarizer: problem, users, requirements, out-of-scope. |
| 2. Define "good" as evals | [`PRD.md`](./PRD.md) — § 5–7: success metrics, failure taxonomy, the four-grader ladder (code → LLM → agent → human). |
| 3. Build a golden set | [`Test cases - Golden set.csv`](./Test%20cases%20-%20Golden%20set.csv) — **200 labelled rows = 25 evals × 8 case types** (happy path → tool/agent failure), with transcripts, contexts, gold answers, `must_include` / `must_not_include`, difficulty, and failure mode. |
| 4. Split it | `test_set` column: **Train 40 (20%) / Validation 120 (60%) / Test 40 (20%)**, stratified per eval — every eval appears in every split, so the sets look alike. |
| 5. Few-shot examples | [`Few shot examples.md`](./Few%20shot%20examples.md) — **8 paste-ready input/output pairs** (Step 6) picked from the Train split. |
| 6. n8n evaluation workflow | [`n8n-eval-workflow.md`](./n8n-eval-workflow.md) — **Steps 7–9**: import the Validation table, wire the LLM judge (`faithful` / `complete` / `fair` JSON verdicts), and run the evaluation. |
| 7. Metrics, thresholds & gating | [`PRD.md`](./PRD.md) § 11 — release-gate metrics; [`Release gate checklist.md`](./Release%20gate%20checklist.md) — the scored gate; [`Eval run report.md`](./Eval%20run%20report.md) — the run record for submission. |

---

## The pipeline at a glance

```
ProductBrief → PRD (§5–7 define "good") → Evals list (E01–E25)
        ↓
Golden set (200 rows · 25 evals × 8 case types)
        ↓  split 20/60/20, stratified per eval
Train (40) ──────→ Few shot examples.md ──→ agent prompt  (Step 6)
Validation (120) → n8n "Validation eval set" table → LLM judge → metrics  (Steps 7–9)
Test (40) ───────→ kept blind → Release gate (PRD §11) → GREEN / RED before ship
```

**Core principle:** use the cheapest grader that can actually see the failure — rules for facts, an LLM for meaning, an agent for investigation, and people for calibration.

---

## Headline result (illustrative — see note)

Two example files show a **complete, internally consistent run** so you can see how the artifacts look filled in:

> **Validation run (VAL-003, 120 rows):** Faithful 98.3% · Complete 95.8% · Fair 99.2% — but **8 failing rows, 6 High-severity** on blocking evals (injection-paraphrase, wrong owner, exact-figure claims) → fixes in prompt v3.1.
> **Blind gate re-run (VAL-004, Test 40): GREEN → ship** — Faithful 95% · Complete 90% · Fair 97.5% · weighted failure points **4 ≤ previous 6** · judge false-pass **3%**.

⚠️ **All numbers above are fabricated for demonstration** ([`Release gate checklist - Example filled.md`](./Release%20gate%20checklist%20-%20Example%20filled.md), [`Eval run report - Example filled.md`](./Eval%20run%20report%20-%20Example%20filled.md)). The **real run is pending** — it requires Steps 6–9 in an n8n instance (below). Blank templates are ready to take the real numbers.

---

## How to verify / reproduce

1. **Open the repo** — every deliverable links above; the dataset checks out at 200 rows / 25 evals / 8 types, with a valid 40/120/40 split.
2. **Run the n8n evaluation** — follow [`n8n-eval-workflow.md`](./n8n-eval-workflow.md) Steps 6–9: paste the 8 few-shots, import `Test cases - Golden Validation set.csv` as the eval-set table, wire **Data set → Evaluating → LLM judge → Set outputs/Set metrics**, and click **Run Evaluation**.
3. **Gate it** — paste the per-row verdicts into [`Release gate checklist.md`](./Release%20gate%20checklist.md) (blind Test set, 40 rows) and the narrative into [`Eval run report.md`](./Eval%20run%20report.md).
4. **Hygiene** — the Test set stays sealed: no tuning the prompt/rubric on blind rows (PRD § 11.5).

---

## Status

- **Done (16 artifacts):** product brief, eval PRD, 25-eval catalogue, 200-row golden set + 40/120/40 split, few-shots, n8n guide, release gate + templates, demo-filled examples, MIT license, public git repo with versioned history.
- **Owned by the candidate (me):** the actual n8n run (Steps 6–9) with real numbers, then sign-off on the gate.
- **Repo:** [github.com/Gursimaran21/NextLeap-Built-AI-Evals-11-October-2026](https://github.com/Gursimaran21/NextLeap-Built-AI-Evals-11-October-2026) · branch `main` · MIT.

*Full file map and project status: [`README.md`](./README.md). Methodology source: [Evals Class — Meeting summarizer](https://evals-for-everyone.vercel.app/meeting-summarizer/foundations) (synthetic teaching data).*