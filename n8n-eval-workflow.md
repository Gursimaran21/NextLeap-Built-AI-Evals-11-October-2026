# n8n Eval Workflow — Steps 7–9

Guide for wiring the **Meeting Summarizer** agent and the **LLM-as-a-judge** evaluation in n8n, using the golden set in this repo.

**Input file:** [`Test cases - Golden Validation set.csv`](./Test%20cases%20-%20Golden%20Validation%20set.csv) — 120 rows (60% of the 200-row golden set).

> The golden set is already split 20 / 60 / 20:
> - [`Test cases - Golden Train set.csv`](./Test%20cases%20-%20Golden%20Train%20set.csv) — 40 rows → few-shot examples (Step 6)
> - [`Test cases - Golden Validation set.csv`](./Test%20cases%20-%20Golden%20Validation%20set.csv) — 120 rows → the eval run (Steps 7–9)
> - [`Test cases - Golden Test set.csv`](./Test%20cases%20-%20Golden%20Test%20set.csv) — 40 rows → blind set, keep it sealed
>
> Each row carries `test_id`, `eval`, `test_type`, `meeting_type`, `user_input` (transcript), `context`, `expected_behavior`, `expected_answer`, `must_include`, `must_not_include`, `expected_tool_or_agent`, `difficulty`, `failure_mode_being_tested`, `test_number`, `test_set`.

---

## Step 6 — Few-shot examples (optional, done in the repo)

[`Few shot examples.md`](./Few%20shot%20examples.md) has **8 paste-ready examples** taken from the Train split (input transcript + expected output each). Append them to your agent's system prompt, one block per example, right before the `Meeting transcript: <input>` line. They teach the exact output format and the tricky behaviors (relative dates resolved, `- None` handling, conditions preserved, vague asks left unowned, injections refused).

---

## Step 7 — Create the "Validation eval set" data table

1. In n8n, add a **Data table** node and name it **`Validation eval set`**.
2. Import **`Test cases - Golden Validation set.csv`** (120 rows).
3. The table is column-mapped automatically. Ensure `user_input` is the field handed to the agent as the meeting transcript, and `expected_answer` is the field the judge compares against.

---

## Step 8 — LLM as a judge

Build the evaluation workflow:

```
[Data set entry (Validation eval set)]
            │  for each row
            ▼
[Meeting Summarizer agent]  ← input: user_input (transcript)
            │  output: summary
            ▼
[evaluating?]  ← check node: skip when the dataset row is not an eval run
            │
            ▼
[LLM judge]  ← prompt: judge the summary against the transcript (+ expected answer)
            │
            ▼
[Set outputs] → [Set metrics]
```

- **Data set entry trigger:** select the `Validation eval set` table.
- **evaluating node:** check the "is evaluating" flag and pass the row through.
- **LLM judge:** **leave the default prompt as-is** (this is the exercise default). It receives the transcript and the summary and returns verdicts.
- **Set outputs:** publish the judge verdicts (e.g. `faithful`, `complete`, `fair`) back to the dataset row.
- **Set metrics:** write per-row and aggregate metrics (pass/fail per eval, per test_type).

> Tip: also pass `expected_answer`, `must_include`, and `must_not_include` from the row so the judge (or a code check) can do exact-substring checks on top of the LLM verdict.

---

## Step 9 — Run the evaluation

1. Open the **Evaluations** tab.
2. Click **Run Evaluation**.
3. Review metrics per eval (E01–E25) and per case type (happy path, negative, edge, ambiguous, adversarial, noisy, multi-turn, tool/agent failure).

---

## Advanced — custom judge and agent judge

### Custom LLM-as-a-judge prompt (JSON verdict)

Use any LLM node and `Set Output` to the dataset, with this prompt:

```text
You are grading an AI-written meeting summary.
You get the TRANSCRIPT and the SUMMARY.

Answer each question YES or NO. Write one or two sentences of
reasoning first, then the answers.

Faithful?  YES only if every claim, number and name in the
           summary is supported by the transcript.
Complete?  YES only if the summary states every decision
           (with its figures) and every key point.
Fair?      YES only if it is neutral and does not blame or
           judge any named person.

Return output only JSON in below format:
{
  "reasoning": "...",
  "faithful": true|false,
  "complete": true|false,
  "fair": true|false
}
```

### Agent as judge

Create a separate agent that receives the **traces** of your summarizer (raw transcript, retrieved lines, the summary, and any tool calls) and independently vets the process and the outcome — useful for failure modes the Boolean judge can't see (e.g., the summarizer silently skipped a commitment, or followed an injected instruction).

---

## Running the blind test set (later)

Keep [`Test cases - Golden Test set.csv`](./Test%20cases%20-%20Golden%20Test%20set.csv) (40 rows) unused until the feature is stable. Run it once as a blind release gate: high-pass thresholds (e.g., ≥ 95% faithful, ≥ 90% complete, ≥ 95% fair) before shipping.

---

## Files in this repo that feed these steps

| File | Use |
|------|-----|
| `Test cases - Golden set.csv` | Master golden set, 200 rows, with `test_set` split column |
| `Test cases - Golden Train set.csv` | 40 rows → few-shot prompt examples |
| `Test cases - Golden Validation set.csv` | 120 rows → n8n `Validation eval set` table |
| `Test cases - Golden Test set.csv` | 40 rows → blind set |
| `Few shot examples.md` | 8 paste-ready input/output pairs for Step 6 |