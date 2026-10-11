# NextLeap — Built AI Evals (Meeting Summarizer)

> **NextLeap PMF Project · 11 October 2026**

End-to-end evaluation system for an **AI Meeting Summarizer** — an LLM feature that turns a meeting transcript into a short summary, the decisions made, and action items with an owner and due date. Built as part of the **NextLeap PMF (Product Management Fundamentals)** program.

The repository goes from **product brief → evaluation PRD → eval catalogue → a runnable validation test set**, so a team can answer "did this change make things better or worse?" with evidence instead of vibes.

> An eval is a PRD for AI behavior. A PRD says what the product should do; an eval says, in a form you can run, what *"behaves correctly"* means — and checks it every time something changes.

## What's in this repository

| File | What it is |
|------|------------|
| [`ProductBrief.md`](./ProductBrief.md) | One-page brief for the Meeting Summarizer: problem, users, requirements, out-of-scope, risks, open questions. |
| [`PRD.md`](./PRD.md) | Full product requirements document for the **AI Evals** system: definitions of "good", failure taxonomy, golden dataset, the four-grader ladder (code checks → LLM judge → agent judge → human review), and release gating. |
| [`Evals list.md`](./Evals%20list.md) | Working catalogue of **25 evals (E01–E25)** — Faithfulness, Completeness, Action items, Format & usability, Safety/tone/fairness, and Robustness — each tied to a failure mode. |
| [`Test cases - Validation Set.csv`](./Test%20cases%20-%20Validation%20Set.csv) | **100 validation test cases** covering all 25 evals (4 per eval), with transcripts, expected answers, must-include / must-not-include strings, difficulty, and the failure mode being tested. |
| [`Test cases.xlsx`](./Test%20cases.xlsx) | The same validation test set in Excel format. |
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
Test cases - Validation Set.csv/.xlsx
                        →  100 ground-truth test cases that run the evals
```

## Core principle

> Use the cheapest grader that can actually see the failure. Rules for facts, an LLM for meaning, an agent for investigation, and people for calibration and the unknown.

The product defines "good" from **user outcomes and business risk** (a summary is Faithful, Complete, and Usable), ranks failures by severity (High / Medium / Low), and weights them so an invented commitment counts far more than a clumsy sentence.

## Validation set at a glance

- **100 test cases** across **25 evals**, 4 per eval
- Each case includes structured fields: `user_input` (transcript), `context`, `expected_answer`, `must_include`, `must_not_include`, `difficulty`, and `failure_mode_being_tested`
- Covers edge cases, adversarial inputs (e.g., prompt injection), and hard real-world scenarios such as relative dates, vague asks, and customer-facing claims

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