# A/B Test Audit — Generative Operators for deep-discover

- **Date**: 2026-08-07
- **Auditor**: main session (orchestrator)
- **Test question**: 古埃及人到底怎麼建造吉薩大金字塔的？我們現在對它的建造方法、勞動力、和運輸技術，實際知道什麼？
- **Hypothesis**: Adding seven generative operators + a closure-saturation stop condition to the deep-discover skill prevents premature convergence and produces materially deeper discovery than the base skill.

## Method

Two runs of the deep-discover loop on the **same question**, starting from the **same grounding and the same initial 5-part decomposition** (so the only variable is the discovery/stop logic):

- **Control** (`control/`): base skill, step 5/6 exactly as written.
- **Experimental** (`experimental/`): base skill + generative-operators patch (7 operators + closure-saturation stop).

Both runs were dispatched as parallel subagents. The shared initial plan is `initial-plan.md`; the experimental treatment is `operators-patch.md`.

## ⚠️ Methodological limitation (affects both runs equally)

Neither subagent had the `Agent` tool available, so neither could dispatch truly isolated worker/verifier subagents; each performed retrieval + verification inline as the orchestrator. This **weakens the generator-verifier isolation invariant for both runs equally**. The A/B comparison of the operators therefore remains valid (the variable is unchanged), but the absolute quality of both runs is lower than a true multi-agent deep-discover run would be. A follow-up in an environment with the `Agent` tool is needed to confirm the operators' effect on a fully isolated pipeline.

## Results

| Metric | Control | Experimental |
|---|---|---|
| Cycles run | **1** (stopped at cycle 1) | **4** (closure saturated) |
| EV files | 23 | 38 |
| Claims | 5 | 11 |
| Distinct sub-questions explored | 5 (initial only) | **16** |
| Source conflicts surfaced | 0 HIGH / 1 LOW | **3** (timeline, ramp feasibility, internal ramp) |
| Challenges dominant "ramp" narrative | yes | yes (stronger) |
| Final confidence | medium–high | medium |
| Acknowledges uncertainty | yes | yes (more precise) |
| Token / tool usage | 1.3M / 45 | 4.2M / 81 |

## Key finding: control exhibits the predicted premature convergence

The control stopped after cycle 1 and **rationalized the stop via the convergence guard**, dismissing three real sub-questions as "refinements, not load-bearing":

> "(a) Is the Ahramat-branch finding fully accepted? … non-load-bearing. (b) Was the labor force corvée or paid? … a refinement, not a new load-bearing branch. … DECISION: STOP after cycle 1."

The experimental run **proved those sub-questions were load-bearing** — they materially changed the answer. This is exactly the failure mode the operators are designed to fix: the base skill's convergence guard is a self-fulfilling prophecy, because the orchestrator has no mechanism forcing it to look, so it rationalizes unexamined questions as "refinements."

## Load-bearing content the experimental found that the control missed

1. **ScanPyramids Big Void / North Face Corridor** (EV-0028, EV-0033, EV-0036) — direct archaeological check for a ramp/lifting mechanism. Absent from all 23 control EV files.
2. **Heaviest granite beams (40–70 t, 43 over 60 t)** (EV-0029) — "no ramp model explains how they were lifted to 43–65 m." The strongest counter-evidence to "ramps are the answer." Not in control.
3. **"Not slaves" has a 4th-Dynasty boundary** (EV-0030) — Middle Kingdom pyramids used foreign (Canaanite) labor. Control stated "not slaves" as flat consensus; experimental correctly scoped it.
4. **Harbor 7 m flood-rise was for transport, not a hydraulic lift** (EV-0037) — a common misconception the control did not clarify.
5. **"Mystery solved" headlines are overstated** (EV-0038) — not addressed by control.
6. **Provenance of alternative theories** (Houdin's commercial interest, non-Egyptologist authors) — not addressed by control.

## Operator attribution (experimental)

All seven operators produced genuinely new sub-questions:
- **Negation** → ScanPyramids direct evidence, North Face Corridor endoscopy, "what lies beyond the NFC"
- **Pairwise conflict** → ramp feasibility (Nature 2025 vs Brichieri-Colombi 2015), timeline (10 vs 20 yr), NFC function, harbor transport-vs-lift
- **Implication** → harbor/canal excavation evidence, hydraulic-lift theory
- **Provenance** → Houdin theory, non-Egyptologist theories
- **Absence** → decisive evidence for the lifting method (repeatedly probed, reported as genuine absence)
- **Boundary** → "not slaves" across periods, heaviest beams
- **Analogy** → cross-cultural ramp/lifting comparison (Inca, Müller-Römer)

## Stop-condition behavior (experimental)

The run did **not** stop on claim-promotion diminishing returns. It stopped only when applying all seven operators produced **0 genuinely new sub-questions** (closure saturation in cycle 4), which coincided with the 4-cycle hard ceiling. Failed searches (no direct ramp evidence) were treated as evidence of absence and re-probed across cycles, not as stop signals. No permanent dispute arose, so the "why do sources disagree?" redirect was not triggered.

## Verdict

**The generative operators are effective and target the diagnosed failure mode.** The control's early stop was not because "the evidence was sufficient" — it was because the base skill had no mechanism forcing it to look, and the convergence guard let it rationalize unexamined questions as refinements. The experimental run's operators forced it to ask "what would falsify this, who has incentive, under what conditions, where is the gap" for every claim, so the closure only saturated at cycle 4 and surfaced six load-bearing findings the control never touched.

**Cost**: the experimental run used ~3.2× the tokens. This is a depth-vs-cost tradeoff, not a defect, but it should be tracked.

## Recommendation

Adopt the generative operators + closure-saturation stop condition into the skill (done — see `SKILL.md` step 5/6). Track token cost per run. Re-run the A/B in an environment with the `Agent` tool to confirm the effect on a fully isolated multi-agent pipeline.
