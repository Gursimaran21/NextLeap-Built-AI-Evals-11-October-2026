# NextLeap — Built AI Evals (Meeting Summarizer)

> **NextLeap PMF Project · 11 October 2026**

End-to-end evaluation system for an **AI Meeting Summarizer** — an LLM feature that turns a meeting transcript into a short summary, the decisions made, and action items with an owner and due date. Built as part of the **NextLeap PMF (Product Management Fundamentals)** program.

The repository goes from **product brief → evaluation PRD → eval catalogue → a runnable validation test set**, so a team can answer "did this change make things better or worse?" with evidence instead of vibes.

> An eval is a PRD for AI behavior. A PRD says what the product should do; an eval says, in a form you can run, what *"behaves correctly"* means — and checks it every time something changes.

## What's in this repository

**Start here:** [`Submission cover page.md`](./Submission%20cover%20page.md) — a one-pager for NextLeap reviewers: the exercise, the artifacts, the demo run, and how to verify.

| File | What it is |
|------|------------|
| [`Submission cover page.md`](./Submission%20cover%20page.md) | **One-page orientation for NextLeap reviewers** — how this repo answers the exercise, the pipeline, the (illustrative) headline result, and how to reproduce it. |
| [`ProductBrief.md`](./ProductBrief.md) | One-page brief for the Meeting Summarizer: problem, users, requirements, out-of-scope, risks, open questions. |
| [`PRD.md`](./PRD.md) | Full product requirements document for the **AI Evals** system: definitions of "good", failure taxonomy, golden dataset, the four-grader ladder (code checks → LLM judge → agent judge → human review), and release gating. |
| [`Evals list.md`](./Evals%20list.md) | Working catalogue of **25 evals (E01–E25)** — Faithfulness, Completeness, Action items, Format & usability, Safety/tone/fairness, and Robustness — each tied to a failure mode. |
| [`Test cases - Golden set.csv`](./Test%20cases%20-%20Golden%20set.csv) | **200-row golden (labelled) test set**: 25 evals × 8 case types, with transcripts, contexts, gold answers, must-include / must-not-include strings, and a `test_set` split column (Train / Validation / Test). |
| [`Test cases - Golden Train set.csv`](./Test%20cases%20-%20Golden%20Train%20set.csv) | The **Train split (40 rows, 20%)** — kept for few-shot examples in the agent prompt. |
| [`Test cases - Golden Validation set.csv`](./Test%20cases%20-%20Golden%20Validation%20set.csv) | The **Validation split (120 rows, 60%)** — imported into n8n as the `Validation eval set` data table. |
| [`Test cases - Golden Test set.csv`](./Test%20cases%20-%20Golden%20Test%20set.csv) | The **Test split (40 rows, 20%)** — the blind set, kept sealed until release time. |
| [`Few shot examples.md`](./Few%20shot%20examples.md) | **8 paste-ready few-shot examples** (input transcript + expected output) picked from the Train split for the agent prompt. |
| [`n8n-eval-workflow.md`](./n8n-eval-workflow.md) | **Full Build Hours walkthrough (Steps 1–9)**: product brief → agent system prompt → golden set → split → few shots → data table → LLM judge → run evaluation. |
| [`Release gate checklist.md`](./Release%20gate%20checklist.md) | **Scored, fill-in-the-blanks release gate** — run the PRD § 11 thresholds on the blind set, record evidence, and get a GREEN / RED / INCONCLUSIVE verdict before shipping. |
| [`Release gate checklist - Example filled.md`](./Release%20gate%20checklist%20-%20Example%20filled.md) | **Illustrative filled example** of the release gate — fabricated numbers for the blind re-run (VAL-004 → **GREEN → ship**), consistent with the demo run report. Replace with real results. |
| [`Eval run report.md`](./Eval%20run%20report.md) | **PMF submission run report** — paste the n8n results into this template: aggregate + per-eval + per-case-type results, failure analysis, judge calibration, and sign-off. |
| [`Eval run report - Example filled.md`](./Eval%20run%20report%20-%20Example%20filled.md) | **Illustrative filled example** of the run report — internally consistent (fabricated) numbers showing a completed run: validation 120 → fixes → blind gate GREEN. Replace with real results before submitting. |
| [`Test cases - Validation Set.csv`](./Test%20cases%20-%20Validation%20Set.csv) | Earlier **100 validation test cases** covering all 25 evals (4 per eval). |
| [`Test cases.xlsx`](./Test%20cases.xlsx) | The same 100-case validation test set in Excel format. |
| [`README.md`](./README.md) | This overview. |
| [`LICENSE`](./LICENSE) | MIT license. |

## How the pieces fit together

```
ProductBrief.md          →  what the product is and what "good" means
        ↓
PRD.md                   →  the eval system spec, built from failure modes
        ↓
Evals list.md            →  E01–E25: each failure → a named, testable eval
        ↓
Test cases - Golden set.csv
                         →  200 labelled cases (25 evals × 8 case types)
        ↓  split 20 / 60 / 20 (stratified per eval)
Train (40)  →  Few shot examples.md  →  agent prompt (Step 6)
Validation (120) →  n8n "Validation eval set" table → LLM judge → metrics (Steps 7–9)
Test (40)   →  blinded until release gate
```

## Core principle

> Use the cheapest grader that can actually see the failure. Rules for facts, an LLM for meaning, an agent for investigation, and people for calibration and the unknown.

The product defines "good" from **user outcomes and business risk** (a summary is Faithful, Complete, and Usable), ranks failures by severity (High / Medium / Low), and weights them so an invented commitment counts far more than a clumsy sentence.

## Golden set at a glance

- **200 test cases** across **25 evals**, 8 per eval — one row per case type: happy path, negative, edge/boundary, ambiguous/incomplete, adversarial, noisy input, multi-turn/context, and tool/agent failure
- Each case includes structured fields: `user_input` (transcript), `context`, `expected_behavior`, `expected_answer`, `must_include`, `must_not_include`, `expected_tool_or_agent`, `difficulty`, and `failure_mode_being_tested`
- Pre-split **Train 40 (20%) / Validation 120 (60%) / Test 40 (20%)**, stratified per eval so all three sets look alike (every eval appears in every split)
- The earlier `Test cases - Validation Set.csv` / `.xlsx` (100 cases, 4 per eval) remains as a compact companion set

## Project status

- **Type:** Product brief + evaluation PRD + validation artefacts
- **Reference application:** Meeting Summarizer (LLM feature)
- **Status:** Draft for review · v1.0 · 2026-10-11
- **Repository:** public on GitHub, `main` branch, versioned with git
- GitHub description: *"Built AI Evals in NextLeap via n8n.io"*

## Source

The eval methodology is based on the [Evals Class](https://evals-for-everyone.vercel.app/) **Meeting summarizer** case study — <https://evals-for-everyone.vercel.app/meeting-summarizer/foundations>. All data on the source site is synthetic and generated for teaching.

## License

[MIT](./LICENSE) © 2026 Gursimaran21.