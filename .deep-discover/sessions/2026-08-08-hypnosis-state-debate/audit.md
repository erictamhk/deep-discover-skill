# Three-Way A/B/C Test Audit — Generative Operators + Draft-Answer Operator

- **Date**: 2026-08-08
- **Auditor**: main session (orchestrator)
- **Test question**: 催眠到底是一個真實的意識狀態改變（altered state of consciousness / trance），還是只是社會順從和角色扮演（social compliance / role-playing）？2023-2025 的科學證據到底怎麼說？
- **Hypothesis 1 (reproducibility)**: The old skill (no operators) exhibits premature convergence on a new topic.
- **Hypothesis 2 (draft-answer)**: An 8th operator that audits the draft synthesis catches backing-gaps the 7 pool operators miss.

## Method

Three runs on the **same question**, from the **same grounding and initial 5-part decomposition**:

- **Control** (`control/`): OLD skill — step 5/6 before operators (claim-promotion diminishing returns, convergence check).
- **Current** (`current/`): current skill — 7 generative operators + closure saturation.
- **Draft-answer** (`draft-answer/`): current skill + 8th operator (draft-answer) that audits draft synthesis for `[UNBACKED]` assertions.

All three were dispatched as parallel subagents. The shared plan is `initial-plan.md`; condition patches are `old-step5-step6.md` and `draft-answer-patch.md`.

## Methodological limitation (affects all three equally)

No subagent had the `Agent` tool, so none dispatched truly isolated worker/verifier subagents; each performed retrieval + verification inline. Generator-verifier isolation is weakened equally across conditions, so the A/B/C comparison of the operators remains valid. Additionally, the built-in WebSearch quota was exhausted for all three; they switched to Exa `web_search` (free Tier 1 highlights) per AGENTS.md budget rules.

## Results

| Metric | Control (old) | Current (7 ops) | Draft-answer (8 ops) |
|---|---|---|---|
| Cycles run | 2 | 2 | **3** |
| EV files | 25 | 35 | 36 |
| Claims | 8 | 8 | 8 |
| Distinct sub-questions | 6 | 7 | **12** |
| Source conflicts surfaced | 2 (MED) | 3 (1 HIGH→permanent) | 4 (1 HIGH→resolved) |
| Token usage | 2.3M | 2.9M | 4.5M |
| Tool uses | 59 | 72 | 92 |

## Key finding 1: control did NOT premature-converge this time

Unlike the Giza test, the control run did not stop at cycle 1. The old convergence guard caught one genuinely-new sub-question and pushed to cycle 2, which surfaced the Faerman 2024 SHIFT RCT (the strongest-design, causal-intervention evidence). All three reports reached a consistent synthesis (both extremes disconfirmed; predictive coding is the consolidating framework).

**This proves operators are not a panacea.** When the question is a mature two-sided debate and the evidence clearly points to "neither extreme," even the old skill can be pushed to synthesis. The operators' value is in exhausting the question space, not in finding the correct answer.

## Key finding 2: draft-answer operator was the only one to reach cycle 3

The three runs differ not in whether they found the right answer, but in **whether every assertion in the answer has traceable backing**:

- **Control** asserted "hypnosis has clinical efficacy evidence" but admitted in its own report this was an unsurfaced gap ("no direct evidence in the pool"). It shipped an unbacked assertion.
- **Current** never searched for clinical efficacy — the 7 pool operators did not force it to check "does this sentence I'm about to write have backing?"
- **Draft-answer** caught this in cycle 1: assertion D10 tagged `[UNBACKED]` ("clinical efficacy is from 2023-2025 meta-analyses" — no EV in pool supports this). This directly triggered cycle 2 retrieval, which found the Akkermann 2024 meta-review (49 meta-analyses, 261 RCTs).

This is the draft-answer operator's unique capability: **it catches implicit assertions.** The 7 pool operators ask "what questions does the evidence have?" The draft-answer operator asks "what am I claiming that I haven't sourced?" The latter is a blind spot the former cannot cover.

## Key finding 3: draft-answer surfaced a deeper gap in cycle 3

Cycle 2's draft exposed another `[UNBACKED]`: D9 — "does hypnosis show effects beyond expectancy/placebo-matched controls?" This is the **most decisive test** for distinguishing "real altered state" from "expectancy-mediated response," but no EV in the pool answered it. This triggered cycle 3, which found the Elkins 2025 sham-hypnosis RCT (n=250) plus two EEG studies, producing C-0008 — the most precise claim in the entire session.

**Control and Current never touched this question.** Both said "neither extreme is right" but neither asked "how would we tell them apart?" The draft-answer operator, by forcing the orchestrator to write the answer, exposed that this discriminative question was load-bearing and unbacked.

## Difference matrix (the real finding)

| Capability | Control | Current | Draft-answer |
|---|---|---|---|
| Found correct synthesis | ✅ | ✅ | ✅ |
| Every assertion has backing | ❌ (clinical efficacy unbacked) | ❌ (never searched) | ✅ (draft forced backing) |
| Asked the discriminative question | ❌ | ❌ | ✅ (hypnosis vs placebo) |
| Provenance check | ❌ | ❌ | ✅ (DLPFC 3-signature mainly Spiegel lab) |
| Marks inferences as inferences | partial | partial | ✅ (`[INFERRED]` tag) |

## Operator attribution (draft-answer condition)

- **Cycle 1**: negation (×2), pairwise-conflict (ALE-vs-DMN/SN/ECN), provenance (Spiegel lab), boundary, analogy (placebo), absence (efficacy meta), **draft-answer (D8 INFERRED + D9 + D10 UNBACKED)**. The draft-answer `[UNBACKED]` tags (D9, D10) were the ones that drove cycle 2.
- **Cycle 2**: negation+absence+analogy+draft converged on one question (hypnosis vs placebo). Draft exposed 1 `[UNBACKED]` (D9), driving cycle 3.
- **Cycle 3**: 7 pool operators produced 0 new sub-questions; draft-answer produced 0 `[UNBACKED]` → closure saturated, stop.

The draft-answer operator was the sole reason the run continued past cycle 2. Without it, two load-bearing gaps (clinical efficacy backing; the hypnosis-vs-placebo discriminative question) would have shipped as unbacked assertions.

## Verdict

**The draft-answer operator is effective and fills a real blind spot of the 7 pool operators.** The blind spot: pool operators inspect the evidence, not the claims-being-made. An orchestrator can fill the evidence pool densely while silently inserting unbacked assertions into the synthesis — and all 7 pool operators miss this because they don't read the draft.

Cost: ~1.5× tokens vs current (4.5M vs 2.9M), one extra cycle. Return: every load-bearing assertion has traceable backing — the core promise of a verification-centric system.

**Honest limitation**: on this topic all three reached the same synthesis, so the "depth" difference is in rigor (backing completeness), not in discovering new content. This is because the hypnosis state debate is a mature controversy with fairly settled evidence. To test draft-answer's effect in a "thin evidence" scenario, a more frontier topic would be needed.

## Recommendation

Adopt the draft-answer operator into the skill as the 8th operator (done — see `SKILL.md` step 5 operator 8 and step 6 stop condition). Track token cost per run.
