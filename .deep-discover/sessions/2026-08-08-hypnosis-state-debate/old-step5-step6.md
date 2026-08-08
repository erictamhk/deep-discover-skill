# OLD Skill Patch (CONTROL condition)

This patch RESTORES the pre-operator version of step 5 and step 6, so the control run tests the skill as it was BEFORE the generative operators were added. This is the baseline for reproducibility — does the old skill's premature-convergence failure show up again on a new topic?

## Replace step 5 (Assess) with this OLD version:

### 5. Assess (the cycle's decision point)

After the verify step, before any synthesis, you (the orchestrator) read the evidence pool and the verifier verdicts and ask three questions explicitly:

1. **Are there open disputes?** Claims still `disputed` or `unverified` at HIGH severity need targeted correction. This is the repair path.

2. **Did the evidence surface new sub-questions that weren't in the original plan?** This is the discovery path. A verifier-facts check might reveal that a cited source actually points at a deeper question. A verifier-conflicts finding might reveal that two claims are in tension. The reasoning worker might have produced a claim that depends on a sub-question no worker ever searched. **List these new sub-questions explicitly.**

3. **Is the evidence sufficient to answer the user's question, or is it thin in a way that isn't a dispute but a gap?** Thinness is not the same as a dispute. If three workers returned evidence about one angle but another angle has only one weak source, that's a gap.

Write the assessment to `{RUN_DIR}/assessments/round-N.md`, recording: which claims need correction, which new sub-questions were surfaced, which gaps need filling, and the current cycle count.

## Replace step 6 (Cycle or stop) with this OLD version:

### 6. Cycle or stop (the loop with guards)

**Guards against infinite loops — mandatory, non-negotiable:**

1. **Max 4 cycles total** (including the initial dispatch as cycle 1).
2. **Diminishing-returns guard.** Track the number of claims promoted from `unverified`/`disputed` to `verified` in each cycle. If a cycle promotes **fewer than 2 claims** or resolves **fewer than 1 HIGH-severity dispute**, stop.
3. **No-rework guard.** A claim that has been the target of correction in two consecutive cycles and is *still* `disputed` is declared a **permanent dispute** — stop trying to fix it.
4. **Budget guard.** If the total evidence pool exceeds **400 EV files**, stop.
5. **Convergence check.** Before dispatching in any cycle after the first, confirm that the new sub-questions are *genuinely new* — not a rephrasing of a sub-question a previous cycle already searched.

**The stop decision is: have ALL of these been met?**
- No HIGH-severity disputes remain, OR remaining HIGH disputes have been declared permanent (guard 3).
- No genuinely new sub-questions left unsearched (convergence guard 5).
- The evidence is sufficient to answer the user's question.

NOTE: This version measures diminishing returns on CLAIM PROMOTION (not on new sub-questions produced). This is the key difference from the current skill — it is vulnerable to premature convergence when early claims verify cleanly but the deeper question space is unexamined.
