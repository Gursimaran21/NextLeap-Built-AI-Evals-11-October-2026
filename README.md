# NextLeap Built — AI Evals

> **NextLeap PMF Project · 11 October 2026**

A **Product Requirements Document (PRD)** for **AI Evals** — a repeatable evaluation system for LLM features.

This repository was created as a NextLeap **PMF (Product Management Fundamentals) project**. It contains a complete, runnable-in-spirit PRD for an AI evaluation system, built from the [Evals Class](https://evals-for-everyone.vercel.app/) *Meeting summarizer* case study. It walks a team from defining "good" through error analysis, a golden dataset, a ladder of graders (code checks → LLM judge → agent judge → human review), a 32-eval catalogue, release gating, and production monitoring — so wrong-but-green outputs never ship.

> An eval is a PRD for AI behavior. A PRD says what the product should do; an eval says, in a form you can run, what *"behaves correctly"* means — and checks it every time something changes.

The reference application is a **Meeting Summarizer**: a feature that reads a meeting transcript and writes a short summary plus action items (owner + due date). Because people who missed the meeting act on the output, the system is designed to catch failures that read beautifully but are wrong.

## Contents

| File | Description |
|------|-------------|
| [`PRD.md`](./PRD.md) | The full product requirements document. |
| [`README.md`](./README.md) | This overview. |
| [`LICENSE`](./LICENSE) | MIT license. |

## What the PRD covers

- **Define "good"** from user outcomes and business risk (Faithful · Complete · Usable).
- **Error analysis** → a named, severity-ranked failure taxonomy.
- **Golden dataset** of hard, stratified, adversarial cases with known-good answers.
- **A ladder of graders**: code checks → LLM judge → agent judge → human review, each seeing what the cheaper rungs cannot.
- **Match each failure** to the cheapest grader that detects it on live traffic.
- A **32-eval catalogue** with severity, lead grader, and pass bars (13 release-blocking).
- **Run → Read → Fix → Add** loop, release gating, and production monitoring/recalibration.

## Core principle

> Use the cheapest grader that can actually see the failure. Rules for facts, an LLM for meaning, an agent for investigation, and people for calibration and the unknown.

## Project status

- **Type:** PRD / evaluation specification
- **Reference application:** Meeting Summarizer (LLM feature)
- **Status:** Draft for review · v1.0 · 2026-10-11
- **Versioned in git** with full history (`main`).

## Source

Built from the [Evals Class](https://evals-for-everyone.vercel.app/) **Meeting summarizer** case study — <https://evals-for-everyone.vercel.app/meeting-summarizer/foundations>.

All data on the source site is synthetic and generated for teaching. In the meeting summarizer case study, code checks run live while LLM, agent, and human verdicts are scripted.

## License

[MIT](./LICENSE) © 2026 Gursimaran21.