# Product Brief: Meeting Summarizer

## 1. What it is

Meeting Summarizer turns a meeting transcript into a short summary of what was decided and who needs to do what by when. Someone who missed the meeting, or left unsure what was agreed, gets the outcome in seconds without reading the transcript.

## 2. Problem

The decisions and commitments from a meeting are buried in the conversation. People forget them, remember them differently, or never hear about them. That leads to dropped follow-ups, duplicated work and arguments about who agreed to what. Manual notes are slow and few people volunteer to take them. Generic AI summaries read well but are often wrong where it matters most: the owner, the date and the numbers.

## 3. Users

| User | What they need |
|---|---|
| Meeting attendees | A reliable record of what they committed to |
| Absent teammates and managers | The decisions without the discussion |
| Customer-facing staff (support, sales) | A safe account of calls that may lead to messages sent to customers |

## 4. What the product does

**Input:** a meeting transcript, with speaker names and the list of attendees.

**Output:**
1. **Summary:** a short, plain-language account of the key points and decisions.
2. **Decisions:** what was agreed, with any figures that were stated.
3. **Action items:** for each one, an owner, a task and a due date.

## 5. Requirements: what it must do

- Capture every decision, including the numbers attached to it (prices, quantities, durations).
- Capture every action item someone committed to, and assign it to the person who committed.
- Convert relative dates ("by Thursday") into calendar dates, using the meeting date.
- Show "none" for the due date when no date was given, rather than guessing.
- Leave out anything that wasn't said in the meeting.

## 6. Requirements: how it must be

- **Accurate.** Every name, number and claim comes from the transcript. It never invents offers, commitments or figures.
- **Correctly attributed.** The owner is a real attendee who took the task, never a vague "Team" or "someone".
- **Neutral.** It never blames, judges or characterises a named person.
- **Concise.** The summary is no more than 80 words, with no filler and no recap of the whole conversation.
- **Predictable.** The output has the same structure every time, so other tools can rely on it.
- **Honest about gaps.** If an owner or decision is unclear, it says so instead of guessing.

Failures are not equal. An invented commitment, a wrong owner or unfair blame is serious, because people act on it. A clumsy sentence is minor. The product should be tuned to avoid the serious ones first.

## 7. Success looks like

- People trust the summary enough to skip the transcript.
- Owners and dates are correct in nearly every meeting.
- No invented commitments reach a customer.
- Fewer "I thought you were doing that" follow-ups.

## 8. Not in scope (for now)

Live audio capture and transcription, sending emails or tasks automatically, translation, memory across meetings, and calendar integration.

## 9. Risks

- **Trust.** One confident mistake, such as a wrong owner, can make people stop using it.
- **Customer exposure.** Summaries of customer calls get forwarded, so an invented concession has real cost.
- **Privacy.** Transcripts contain sensitive talk about people, pricing and hiring, so access and retention must be defined.
- **Harm to individuals.** A summary that frames a named person negatively can damage them.

## 10. Open questions

1. Who can see a summary: attendees only, or anyone invited?
2. Should sensitive meetings (customer, hiring) be reviewed by a person before the summary is shared?
3. What should it produce when a meeting has no clear decisions or owners?
4. Should it flag uncertainty, for example "owner unclear", in the output?
