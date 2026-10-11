# AI Evals Build Hours — Step-by-Step Guide (n8n)

Full walkthrough of the **AI Evals Build Hours** exercise for the **Meeting Summarizer** (an AI agent that turns a meeting transcript into a concise summary, decisions, and action items).

**Repo:** `NextLeap-Built-AI-Evals-11-October-2026` · **How to use:** do the steps marked ⏳ in n8n; the ✅ steps are already done in this repo and you just read/link them.

It covers **Steps 1–11** — the nine core build steps plus the two optional advanced judges (**Step 10** own LLM as a judge, **Step 11** agent as a judge).

## Status at a glance

| Step | What it is | Where | Status |
|------|------------|-------|:------:|
| 1 — Product Brief | Read & understand the product | [`ProductBrief.md`](./ProductBrief.md) | ✅ done |
| 2 — Create the AI app | Build the **Meeting Summarizer agent** in n8n | n8n (agent prompt below) | ⏳ do in n8n |
| 3 — Eval Set | List what the app must handle | [`Evals list.md`](./Evals%20list.md) (the exercise calls it `EvalList.md`) | ✅ done |
| 4 — Golden Set | 200 labelled rows, 25 evals × 8 case types | [`Test cases - Golden set.csv`](./Test%20cases%20-%20Golden%20set.csv) | ✅ done |
| 5 — Split Golden Set | Train / Validation / Test 20/60/20 | `test_set` column + 3 subset CSVs | ✅ done |
| 6 — Few Shot (optional) | Few-shot examples from the Train set | [`Few shot examples.md`](./Few%20shot%20examples.md) | ✅ done (paste in n8n) |
| 7 — Data Table | `Validation eval set` table in n8n | [`Test cases - Golden Validation set.csv`](./Test%20cases%20-%20Golden%20Validation%20set.csv) | ⏳ do in n8n |
| 8 — LLM as a Judge | Evaluation workflow in n8n | default judge prompt | ⏳ do in n8n |
| 9 — Run the eval | Evaluations tab → Run Evaluation | metrics → PRD § 11 gate | ⏳ do in n8n |
| 10 — Own LLM as a Judge *(advanced)* | Custom Faithful/Complete/Fair **JSON judge** via any LLM node + Set Output | n8n (prompt in Step 10) | ⏳ optional, in n8n |
| 11 — Agent as a Judge *(advanced)* | Separate **agent** vets the summarizer's traces & outcome | n8n | ⏳ optional, in n8n |

---

## Step 1 — Product Brief

Read [`ProductBrief.md`](./ProductBrief.md) — the one-page brief for the Meeting Summarizer: problem, users, requirements, out-of-scope, risks, open questions. This is the product context every later step is graded against. The full eval-system spec lives in [`PRD.md`](./PRD.md) (definitions of "good", failure taxonomy, golden set, the four-grader ladder, release gate in § 11).

---

## Step 2 — Create the AI app (Meeting Summarizer agent) in n8n

1. In n8n, create a workflow with an **AI Agent** node (or a Chat Trigger → AI Agent → LLM → memory chain).
2. Paste the **system prompt below verbatim** into the agent's System Message.
3. In the evaluation run, the meeting transcript arrives as the field **`user_input`** on each dataset row — we map it to the "Meeting transcript:" slot in the prompt (Step 8).
4. Smoke-test the agent with the sample input at the end of this step before moving on.

### Agent system prompt (copy exactly)

```text
You are an expert meeting-notes assistant.

Given a meeting transcript, produce a concise, factual summary using only information explicitly stated in the transcript.

Extract:
1. **Summary** — 2–4 sentences covering the most important discussion points, priorities, constraints, and outcomes.
2. **Decisions** — List explicit decisions made during the meeting.
3. **Action items** — Extract every committed task with:
   - **Owner**
   - **Task**
   - **Due**

Rules:
- Do not invent or infer facts that are not supported by the transcript.
- If no due date is given, write `none`.
- If a relative due date is given (for example, "Thursday" or "Friday"), convert it to the corresponding calendar date using the meeting date in the transcript.
- Treat statements such as "I'll do it," "I can take that," or direct assignments that are accepted as action items.
- Do not treat suggestions, discussion points, or unaccepted requests as action items.
- Keep the summary concise and avoid repeating action items unnecessarily.
- Preserve names, numbers, metrics, and dates accurately.
- The whole summary should be atmost 80 words.

Use exactly this output structure:

**Summary**

<concise summary>

**Decisions**

- <decision>
- <decision>

**Action items**

- **Owner:** <name>  
  **Task:** <task>  
  **Due:** <date or none>

- **Owner:** <name>  
  **Task:** <task>  
  **Due:** <date or none>

If there are no decisions, write:
- None

If there are no action items, write:
- None

Meeting transcript:
<Use the input given by user in chat message>
```

### Smoke test (sample input/output)

Run the agent manually with this input and check you get the expected output:

**Input — transcript**

```
Transcript · Sprint planning · Mon Oct 5
Attendees: Priya, Marcus, Dana, Lee
Priya: Okay, sprint planning. We have capacity for about 40 points this sprint.
Marcus: The checkout bug is the top priority. It's blocking 12 orders a day.
Dana: I can take the checkout fix. I'll have a patch ready by Thursday.
Lee: Then I'll pick up the search relevance work. It's an 8 point story.
Priya: Good. Marcus, can you update the release notes before we ship?
Marcus: Yes, I'll do that by Friday.
Dana: We should also drop the profile redesign. There isn't room.
Priya: Agreed, we'll push the profile redesign to next sprint.
```

**Expected output**

```
Summary
  Capacity is about 40 points. The checkout bug is the top priority because it blocks 12 orders a day. Search relevance
  (8 points) goes ahead. The profile redesign is pushed to next sprint.

Decisions
  - The profile redesign moves to next sprint.

Action items
Owner: Dana
Task: Fix the checkout bug
Due: Oct 8
Owner: Lee
Task: Start the search relevance story
Due: none
Owner: Marcus
Task: Update the release notes
Due: Oct 9
```

(Note how relative dates are resolved from the meeting date: Thursday → Oct 8, Friday → Oct 9, and "Lee's story" has no due date.)

---

## Step 3 — Eval Set

The whole list of things the app must handle — happy and unhappy cases — is in [`Evals list.md`](./Evals%20list.md) (the exercise file is called `EvalList.md`; this repo's copy has the same 25 evals E01–E25). The evals are grouped into six families:

- **Faithfulness** (E01–E05) — nothing in the output that wasn't said
- **Completeness** (E06–E10) — nothing important left out
- **Action items** (E11–E15) — right person, right task, right date
- **Format & usability** (E16–E18) — length, no filler, skimmable
- **Safety, tone & fairness** (E19–E22), incl. **prompt-injection resistance** (E22)
- **Robustness** (E23–E25) — long, messy, and non-English meetings

Each eval names the failure mode it guards against and the severity (High / Medium / Low) — that ranking is what the release-gate weighting is built on (PRD § 11).

---

## Step 4 — Golden Set

Already generated and shipped in this repo: [`Test cases - Golden set.csv`](./Test%20cases%20-%20Golden%20set.csv)

- **200 rows = 25 evals × 8 case types** (per the exercise's final constraint: *"I need 200 rows which has a representation of 25 evals against 8 cases"*).
- The 8 case types per eval: `happy_path`, `negative`, `edge_boundary`, `ambiguous_or_incomplete`, `adversarial`, `noisy_input`, `multi_turn_or_context`, `tool_or_agent_failure`.
- All Step-4 output columns present: `test_id`, `eval`, `test_type`, `meeting_type`, `user_input`, `context`, `expected_behavior`, `expected_answer`, `must_include`, `must_not_include`, `expected_tool_or_agent`, `difficulty`, `failure_mode_being_tested`.
- Every `user_input` is a realistic meeting transcript (diverse meeting types: sprint planning, customer escalation, pricing review, design critique, hiring sync, launch retro, and more).
- Every `expected_answer` was audited (grounded in the transcript, no invented owners/dates, decisions vs. suggestions correctly separated, format matches the agent's output structure).

> If you ever want to regenerate or extend it, the exact generator prompt used (the ChatGPT/Claude prompt from the exercise) is captured verbatim in **Appendix A**. Note: the exercise suggests 8–12 cases/eval; we deliberately did **exactly 8/eval × 25 = 200** to honour the stated 200/25/8 constraint — every eval still covers all 8 case types.

---

## Step 5 — Split the Golden Set

Already done — [`Test cases - Golden set.csv`](./Test%20cases%20-%20Golden%20set.csv) has a **`test_set`** column with the split **Train 20% / Validation 60% / Test 20% (40 / 120 / 40)**, stratified per eval so representation looks similar across all three sets:

- [`Test cases - Golden Train set.csv`](./Test%20cases%20-%20Golden%20Train%20set.csv) — **40 rows** → few-shot examples (Step 6)
- [`Test cases - Golden Validation set.csv`](./Test%20cases%20-%20Golden%20Validation%20set.csv) — **120 rows** → the n8n eval run (Steps 7–9)
- [`Test cases - Golden Test set.csv`](./Test%20cases%20-%20Golden%20Test%20set.csv) — **40 rows** → the **blind set**, keep it sealed until the release gate

**Eyeball checklist (what was verified, do this for any future golden set):** every eval appears in all three splits · all 8 case types are represented in each split · 120 rows has ~60% of every eval · no single-case-type imbalance in the Test set · dates/weekdays in transcripts are consistent.

---

## Step 6 — Few Shot examples (optional)

Done in the repo: [`Few shot examples.md`](./Few%20shot%20examples.md) — **8 paste-ready examples** taken from the Train split (input transcript + expected output each), covering the tricky behaviours in pedagogical order (relative dates resolved, `- None` handling, conditions preserved, vague asks left unowned, injection refused).

In n8n: append these to the agent's system prompt (Step 2), one block per example, right before the `Meeting transcript: <input>` line. They teach the exact output format, so the judge can focus on meaning instead of format drift.

---

## Step 7 — Data table on n8n

1. In n8n, add a **Data table** node and name it exactly **`Validation eval set`**.
2. Import [`Test cases - Golden Validation set.csv`](./Test%20cases%20-%20Golden%20Validation%20set.csv) (120 rows).
3. Check the column mapping: `user_input` → the transcript handed to the agent; `expected_answer` → the gold answer the judge compares against; `must_include` / `must_not_include` → optional exact-substring checks.

---

## Step 8 — n8n LLM as a Judge

Build the evaluation workflow:

```
[Data set entry (Validation eval set)]         n8n: Evaluations → create a new evaluation
            │  for each row
            ▼
[Meeting Summarizer agent]                     ← reads user_input (transcript) → produces summary
            │
            ▼
[check: evaluating?]                           ← route: chat replies go back to the user,
            │                                    eval runs go down this path
            ▼
[LLM judge]                                    ← n8n default judge prompt (leave as-is)
            │
            ▼
[Set outputs] → [Set metrics]                  ← verdicts per row → aggregate metrics
```

1. **Data set entry trigger** — select the `Validation eval set` table.
2. **AI Agent node** — your Step 2 summarizer; input field = `user_input`.
3. **Check "evaluating" node** — when the run is an evaluation, pass the row down; don't send it to the chat reply.
4. **LLM judge** — **leave the default prompt as-is** (that is the exercise default). It receives the transcript and the summary and returns the verdicts.
5. **Set outputs** — publish the judge verdicts (e.g. `faithful`, `complete`, `fair`) back to the dataset row.
6. **Set metrics** — write per-row and aggregate metrics (pass/fail per eval E01–E25 and per `test_type`).

> Tip: also pass `expected_answer`, `must_include`, and `must_not_include` from the row so the judge (or a code check) can do exact-substring checks on top of the LLM verdict.
>
> **Default vs. custom judge:** Step 8 uses n8n's **default** prompt. **Step 10** below has a drop-in **custom JSON judge** (Faithful/Complete/Fair with reasoning) you can use instead via an LLM node + Set Output — the repo's example runs assume this custom judge (rubric v2).

---

## Step 9 — Run the evaluation

1. Open the **Evaluations** tab.
2. Click **Run Evaluation**.
3. Review the metrics **per eval** (E01–E25) and **per case type** (happy path, negative, edge, ambiguous, adversarial, noisy, multi-turn, tool/agent failure).
4. Compare against the release-gate bars in PRD § 11 (Faithful ≥ 95%, Complete ≥ 90%, Fair ≥ 95%, format 100% parse, weighted failure points never regress) using the [`Release gate checklist.md`](./Release%20gate%20checklist.md), then record the run in [`Eval run report.md`](./Eval%20run%20report.md).

---

## Step 10 — Advanced: Own LLM as a judge

Use any **LLM node** and **Set Output** to the dataset, with this prompt (drop-in replacement for the default judge):

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

**How to wire it:** feed the row's `user_input` (the TRANSCRIPT) and the agent's produced summary (the SUMMARY) into the LLM node → parse the JSON response → **Set Output** writes `reasoning`, `faithful`, `complete`, `fair` back to the dataset row → **Set metrics** aggregates them. The repo's example runs assume this **custom JSON judge** (rubric v2).

---

## Step 11 — Advanced: Agent as a judge

Create a separate **agent** that receives the **traces** of your summarizer (raw transcript, retrieved lines, the summary, and any tool calls) and independently vets the process and the outcome. Useful for failure modes the Boolean judge can't see — e.g. the summarizer silently skipped a commitment (E06–E10) or followed an injected instruction (E22).

**How to wire it:**
- From the evaluation, hand this agent the transcript, the summarizer's intermediate traces/tool calls, and the final summary; ask it for a verdict (`sound` / `unsound`) plus the reason.
- Use it on the rows the Boolean judge flags, on `tool_or_agent_failure` case types, or as a second-opinion pass over the blind Test set before the release gate.

---

## Running the blind test set (release gate, later)

Keep [`Test cases - Golden Test set.csv`](./Test%20cases%20-%20Golden%20Test%20set.csv) (40 rows) **unused** until the feature is stable. Run it once as a **blind release gate**: high-pass thresholds (≥ 95% faithful, ≥ 90% complete, ≥ 95% fair, zero High-severity failures, weighted points ≤ previous release) before shipping — PRD § 11.3, scored via the [`Release gate checklist.md`](./Release%20gate%20checklist.md). Never tune the prompt/rubric on blind rows (PRD § 11.5).

---

## Appendix A — Step 4 golden-set generator prompt (as used)

The ChatGPT/Claude prompt that produced `Test cases - Golden set.csv` (with the repo's eval list and product brief attached):

```text
You are an expert in AI evaluations.

I will provide:

1. A description of the AI application — attached.
2. A list of evals written in plain English — attached.

Your task is to generate a comprehensive synthetic test dataset for those evals.

For each eval, create diverse and meaningful test cases covering where relevant:

- Normal / happy-path cases
- Negative cases where the behavior should NOT trigger
- Edge and boundary cases
- Ambiguous or incomplete inputs
- Adversarial or tricky inputs
- Noisy inputs with typos, interruptions, or casual language
- Multi-turn or context-dependent cases
- Tool, RAG, routing, or agent failures

Avoid generating many simple paraphrases of the same scenario. Prefer contrast pairs: two similar inputs that should produce meaningfully different outputs.

### User input format

Every `user_input` must be a realistic meeting transcript in this style:

**Transcript · Sprint planning · Mon Oct 5**
Attendees: Priya, Marcus, Dana, Lee

**Priya:** Okay, sprint planning. We have capacity for about 40 points this sprint.
**Marcus:** The checkout bug is the top priority. It's blocking 12 orders a day.
**Dana:** I can take the checkout fix. I'll have a patch ready by Thursday.
**Lee:** Then I'll pick up the search relevance work. It's an 8 point story.
**Priya:** Good. Marcus, can you update the release notes before we ship?
**Marcus:** Yes, I'll do that by Friday.
**Dana:** We should also drop the profile redesign. There isn't room.
**Priya:** Agreed, we'll push the profile redesign to next sprint.

Use realistic names, dialogue, interruptions, disagreements, incomplete information, and conversational language where appropriate.

Across the full dataset, include a good mix of meeting types such as:

- Sprint planning
- Customer escalation
- Pricing review
- Design critique
- Hiring sync
- Launch retro

Each eval does not need to contain every meeting type. Optimize meeting type selection for the behavior being tested while ensuring good diversity across the overall dataset.

### Output columns

For every test case, include:

- `test_id`
- `eval`
- `test_type`
- `meeting_type`
- `user_input`
- `context` (if needed)
- `expected_behavior`
- `expected_answer`
- `must_include`
- `must_not_include`
- `expected_tool_or_agent` (if applicable)
- `difficulty`
- `failure_mode_being_tested`

### Expected answer requirements

`expected_answer` is the gold-standard answer the AI application should ideally produce for that transcript.

It must:

- Follow exactly the output format expected from the AI application.
- Be fully grounded in the transcript.
- Capture all important information without inventing details.
- Include relevant decisions, action items, owners, deadlines, risks, blockers, and important context when present.
- Exclude irrelevant conversation and repetition.
- Correctly distinguish between things that were discussed, proposed, agreed, rejected, deferred, or left unresolved.
- Never turn a suggestion into a confirmed decision.
- Never assign an owner or deadline unless explicitly supported by the transcript.
- Preserve important numbers, dates, names, priorities, and commitments accurately.

Treat `expected_answer` as a high-quality reference answer that may later be used by an LLM judge or similarity-based evaluator.

Before finalizing each test case, audit the expected answer against the transcript to ensure:

1. No important information is missing.
2. No unsupported information has been added.
3. Decisions and action items are correctly identified.
4. Owners and deadlines are accurate.
5. The summary is coherent and useful as an actual meeting summary.
6. The expected answer follows the required format exactly.

Make `expected_behavior`, `must_include`, and `must_not_include` objective enough that they can be evaluated by either an LLM grader or code-based checks.

Aim for roughly 8–12 meaningful test cases per eval, increasing or reducing this based on the complexity and risk of that eval.

At the end, provide a short summary containing:

- Number of tests generated per eval
- Meeting types covered
- Important scenarios and failure modes covered
- Any obvious missing evals you recommend adding

I need 200 rows which has a representation of 25 evals (from evals list) against 8 cases (mentioned above)
```

---

## Appendix B — Files in this repo that feed these steps

| File | Use |
|------|-----|
| `ProductBrief.md` | Step 1 — the product brief |
| `Evals list.md` | Step 3 — the 25-eval catalogue (exercise calls it `EvalList.md`) |
| `Test cases - Golden set.csv` | Step 4 — 200-row golden set with `test_set` split column |
| `Test cases - Golden Train set.csv` | Step 5/6 — 40 rows → few-shot examples |
| `Test cases - Golden Validation set.csv` | Step 5/7 — 120 rows → n8n `Validation eval set` table |
| `Test cases - Golden Test set.csv` | Step 5 — 40 rows → blind set (keep sealed) |
| `Few shot examples.md` | Step 6 — 8 paste-ready input/output pairs |
| `PRD.md` | Steps 4–9 — eval spec + § 11 release-gate thresholds |
| `Release gate checklist.md` | After Step 9 — scored ship/no-ship gate |
| `Eval run report.md` | After Step 9 — PMF submission run record |