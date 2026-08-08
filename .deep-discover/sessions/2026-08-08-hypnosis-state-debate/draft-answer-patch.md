# Draft-Answer Operator Patch (DRAFT-ANSWER condition ONLY)

This patch is applied ON TOP OF the current skill (which already includes the 7 generative operators and closure-saturation stop). It adds ONE additional operator — the draft-answer operator — inspired by analyzing the MiroThinker reasoning trajectory, where the act of writing a deliverable (a party pre-talk script) forced the agent to discover its own knowledge gaps (it wrote the script, then had to chase citations for each claim it had made).

The hypothesis: writing a draft answer surfaces gaps that the 7 evidence-pool operators miss, because a draft answer makes implicit assertions explicit — and each assertion that lacks backing evidence is a discovered gap.

## When to apply the draft-answer operator

Apply it ONCE PER CYCLE, after the 7 generative operators in step 5 but BEFORE the stop decision in step 6. It is an 8th operator that operates on the draft synthesis, not the evidence pool.

## The draft-answer operator

1. **Write a rough draft of the final report's answer section** — not the full report, just the core argument: 5-10 sentences that state what the evidence currently supports, what it contests, and what it leaves open. Write this directly to `{RUN_DIR}/assessments/draft-answer-round-N.md`.

2. **For each assertion in the draft, ask: "What is my evidence for this?"** Tag each assertion with:
   - `[BACKED]` — cites specific EV ids that support it.
   - `[INFERRED]` — a bridge between two backed points, but no single EV states it directly. This is a candidate new sub-question: "Is this inference actually supported?"
   - `[UNBACKED]` — no evidence in the pool supports this. This is a discovered gap.

3. **Treat every `[INFERRED]` and `[UNBACKED]` tag as a new sub-question** from the draft-answer operator. Add them to the cycle's assessment alongside the 7 operators' outputs. They are dispatched as targeted retrieval workers in the next cycle.

4. **The stop condition gains one clause:** closure is not saturated if the draft answer contains any `[UNBACKED]` assertions. (A small number of `[INFERRED]` bridges may be acceptable if they are non-load-bearing, but `[UNBACKED]` load-bearing assertions always trigger another cycle, unless the 4-cycle hard ceiling is reached.)

## Why this is different from the absence operator

The absence operator asks "what evidence would decisively answer this sub-question" — it operates on the questions you already know you have. The draft-answer operator operates on assertions you didn't know you were making — it catches the implicit claims that slip into a synthesis because they "feel right" but were never actually sourced. These are the most dangerous gaps because they are invisible until you force yourself to write the answer.
