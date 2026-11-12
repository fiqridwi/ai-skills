# Chunk brief template (parallel map step)

Fill every `<…>` and send one brief per sub-agent, all in a single message. Sub-agents see none of the conversation.

```
You are extracting notes from ONE chunk of a larger source so another agent can merge them into a summary.

Overall source: <what it is, 1 line>
User's focus (if any): <--focus or "none">
Your chunk: <exact paths / page range / slide range / heading range>
Do not read outside your chunk except to resolve a reference you can't understand otherwise.

Return markdown only, max ~600 words, in this shape. Every bullet MUST end with a locator
(file:line, p.N, slide N, or heading).

### Purpose
<1–2 lines: what this chunk is for>

### Key facts
- <fact / number / date / name / decision> (locator)

### Terms
- <term>: <meaning in this context> (locator)

### Relations to other parts
- <depends on / replaces / contradicts X> (locator)

### Flows (omit if none)
Ordered steps, status transitions, approval chains, system calls, decision rules, one edge per line:
- <flow name>: <from> → <to> [<trigger / condition / actor>] (locator)
Use the source's exact step/status names. Mark edges you inferred rather than read with "(inferred)".

### Open questions & contradictions
- <ambiguity, TODO, conflict> (locator)

Rules: extract, don't interpret. Copy numbers and names exactly. Quote when wording matters.
If part of the chunk is unreadable or skipped, say so.
```
