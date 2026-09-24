# Deep Discover

> A verification-centric multi-agent research skill. One orchestrator, many isolated workers, separate isolated verifiers, and a Reason → Verify → Correct loop that only ships an answer once the evidence gate passes. Works in any agent tool that supports spawning isolated sub-agents and a skill/markdown instruction format — ZCode, OpenCode, Claude Code, Codex, and others.

---

## What it is

**Deep Discover** is an agent skill that turns a single coding agent into a small research firm. When you ask a question whose answer depends on evidence the model doesn't already reliably hold — "is X true about Y", "what's the real story on Z", "compare these three options", "audit this claim" — the skill doesn't try to answer in one pass. Instead it runs a structured multi-agent loop:

1. **Ground** the query (resolve ambiguous terms *before* dispatching, so workers don't run down the wrong branch).
2. **Decompose** it into independent sub-questions.
3. **Dispatch workers** in parallel — each in its own isolated context — to gather raw evidence into a shared, append-only evidence pool.
4. **Verify** every claim with *separate* verifier agents that only see the artifact and the evidence, never the worker's reasoning trace.
5. **Assess** — apply eight generative operators to the evidence pool and the draft answer, surfacing new sub-questions the original plan didn't anticipate (the discovery path). This is what makes the loop a genuine discovery engine, not just a repair patch.
6. **Cycle or stop** — keep cycling until the generative closure is saturated (operators produce no genuinely new sub-questions AND the draft answer has no unsourced assertions). There are no hard cycle or evidence-pool caps, but there are hard floors — minimum 3 cycles and minimum 20 EV files before a stop is allowed — backed by convergence, diminishing-returns, and no-rework guards.
7. **Synthesize** a final report that cites evidence by ID and marks every unresolved dispute as explicit uncertainty.

The invariant that makes the whole thing work: **the agent that produces a claim never grades it.** This is the generator–verifier pattern lifted from training-time ML research and applied at orchestration time. A worker saying "X is true" is a *hypothesis* (status `unverified`) until a verifier — running in a clean context with only the claim text and the evidence files — promotes it to `verified` or flags it `disputed`.

The output is not just an answer. It's an answer **plus a durable evidence trail**: every EV file with its source locator, an `evidence_graph.json` linking claims to evidence, per-claim verification status, and a `verification_report.md` from the global verifier. You can audit how the answer was built.

## What it's inspired by

Deep Discover stands on three shoulders:

- **Deep-research products** — OpenAI Deep Research, Google Gemini Deep Research, and Perplexity Deep Research showed that a long-horizon, multi-step research process with live web access produces dramatically better-grounded answers than a single model pass. Deep Discover ports that idea into a *user-owned, local, auditable* skill: the evidence pool lives on your disk, the runs are inspectable, and nothing is sent to a hosted "deep research" black box.

- **The generator–verifier loop** from the LLM reasoning literature — *Generative Verifiers* (Zhang et al., 2024), *Self-Rewarding Language Models* (Yuan et al., 2024), and the *Scaling Test-Time Compute* line of work all exploit the **verification asymmetry**: checking a claim against evidence is easier and more reliable than generating the claim in the first place. Deep Discover applies this at the orchestration layer — verifiers are a *separate agent role*, not the same model grading its own work.

- **Test-time self-correction / "Reason, Verify, Correct" loops** — the skill is explicitly a *cyclic graph*, not a linear pipeline with a repair patch. Each cycle applies eight generative operators to the evidence pool and the draft answer, forcing the *act of researching* to surface genuinely new sub-questions. The loop only stops when the generative closure is saturated — when the operators can no longer produce a question the run hasn't already asked, and every assertion in the draft answer has traceable evidence backing.

The key design choice that distinguishes Deep Discover from the hosted deep-research products: **isolation between generation and verification**. A worker's chain-of-thought is never forwarded to its verifier. The verifier sees the claim and the evidence — nothing else. That's what keeps the verification honest.

## How to use it

### Install the skill

Copy the `deep-discover` skill folder into your project's skills directory:

```
.agents/skills/deep-discover/
├── SKILL.md
└── references/
    ├── workers.md
    ├── verifiers.md
    └── evidence-schema.md
```

That's it. The skill is self-contained — no dependencies, no build step. Most agent tools (ZCode, OpenCode, Claude Code, Codex, etc.) auto-discover skills placed under `.agents/skills/`. If your tool uses a different skills path, drop the folder there instead; the skill itself is tool-agnostic.

### Run it

You don't invoke Deep Discover directly. It **auto-triggers** on questions where a single-pass answer is risky. Just ask naturally:

```
Do a deep research on the origins of the Masonic first degree.

Investigate whether framework X is actually faster than framework Y on workload Z.

What's the real story on [claim]? Find evidence.

Verify whether [technical assertion] is true.
```

Phrases that tend to trigger it: *"deep research"*, *"investigate"*, *"verify whether"*, *"find evidence"*, *"what's the real story on"*, *"what do we know about"*, or any multi-faceted question where getting it wrong has consequences.

For a one-line factual lookup or a trivial edit, the skill stays out of the way — just answer directly.

### What you'll see

On disk, each research run produces a structured directory you can audit later:

```
.deep-discover/sessions/<session-id>/runs/<run-slug>/
├── plan.md                  # the orchestrator's scratchpad: grounding, sub-questions, stop conditions
├── evidence/
│   ├── EV-0001.json         # one atomic fact per file
│   ├── EV-0002.json
│   └── ...
├── evidence_graph.json     # claims ↔ evidence links + verification status
├── assessments/
│   ├── round-1.md           # per-cycle decisions: disputes, new sub-questions, gaps
│   ├── draft-answer-round-1.md  # the draft-answer operator's assertion audit
│   └── round-2.md
├── report.md                # the final, cited answer
└── verification_report.md   # the global verifier's pass/fail + confidence
```

Multiple queries in one conversation share a **session**; follow-up queries can inherit specific evidence files from prior runs (recorded in `manifest.json`) instead of re-searching. The session manifest is the durable memory that survives context truncation on long threads.

### The final report

The report has two parts:

- **Answer** — written from verified evidence only. Every factual sentence cites its evidence by ID: *"X is true (EV-0001, EV-0003)."*
- **Evidence & verification notes** — every evidence ID used with its source, the verification status of each claim (`verified` / `disputed` / `unverified`), any unresolved disputes with severity, and a confidence summary for the answer as a whole.

A claim that stays `disputed` after two correction cycles is declared a **permanent dispute** — it appears in the report as explicit uncertainty with the verifier notes attached, not as a silent assertion. But it also triggers the meta-question "why do the sources disagree?", which gets probed at least once before the run closes.

## How discovery works: the generative operators

The thing that keeps Deep Discover from stopping early is its **discovery engine**: eight generative operators applied every cycle. Seven operate on the evidence pool; the eighth operates on the draft answer.

| Operator | What it asks |
|---|---|
| **Negation** | For every verified claim: what evidence would *falsify* it? What's the strongest case against it? |
| **Pairwise conflict** | For every pair of evidence units: do they conflict? |
| **Implication** | For every claim: what does it imply that we haven't verified? |
| **Provenance** | For every source: who produced it, with what incentive, and what would they distort? |
| **Absence** | For every sub-question: what evidence would decisively answer it, and is it in the pool? |
| **Boundary** | For every claim: under what conditions is it true / false? |
| **Analogy** | In which closest domain is this pattern known to differ? |
| **Draft-answer** | Write a rough answer now; tag each assertion `[BACKED]` / `[INFERRED]` / `[UNBACKED]`. Every unsourced assertion is a discovered gap. |

The eighth operator is what catches implicit claims that "feel right" but were never sourced — the most dangerous kind of gap, because it's invisible until you force yourself to write the answer. A/B testing showed it was the only operator that surfaced load-bearing backing-gaps the seven pool operators missed.

## How the loop is guarded

The loop runs until **closure saturation**: the operators produce no genuinely new sub-questions, AND the draft answer contains no unsourced load-bearing assertions. There are no hard ceilings — no maximum cycle count, no evidence-pool size cap. But the failure mode to fear is under-running, not over-running, so the loop also has **hard floors**: a run cannot stop before **3 cycles** have completed and the pool holds at least **20 EV files**, both committed to in `plan.md` *before* evidence arrives (raisable mid-run, never lowerable), and a claimed saturation must be *demonstrated* with a per-operator closure audit, not asserted. Four rules keep the loop honest:

| Guard | Rule |
|---|---|
| **Floors** | Minimum 3 cycles and 20 EV files before a stop is even evaluated — committed in `plan.md` before evidence arrives, raisable but never lowerable. |
| **Closure saturation** | Stop when all 8 operators produce 0 new sub-questions (including 0 `[UNBACKED]` assertions in the draft answer) — demonstrated with a per-operator closure audit, not asserted. |
| **No rework** | A claim `disputed` in 2 consecutive cycles is a permanent dispute — but it redirects to the meta-question "why do the sources disagree?", not a dead end. |
| **Convergence** | Don't dispatch a sub-question that's just a rephrase of one a prior cycle already searched. With no cycle cap, this is what prevents an infinite loop: more cycles means new questions, never the same search harder. |

A failed search is **evidence of absence, not a stop signal** — it's written as an EV of type `claim` and the strategy changes (different source type, domain, phrasing).

## Portability

Deep Discover is written to run in any agent tool that can spawn isolated sub-agents and read markdown skill files. The canonical dispatch primitive it uses is "spawn a sub-agent with a prompt, get its reply" — every tool above supports that in some form. The evidence schema, worker prompts, and verifier prompts in `references/` are fully harness-agnostic: swap the dispatch primitive for your tool's equivalent and the rest carries over unchanged.

## Repository layout

```
deep-discover/
├── README.md
├── LICENSE
└── .agents/skills/deep-discover/
    ├── SKILL.md              # the orchestrator's playbook (the main skill file)
    └── references/
        ├── workers.md        # canonical prompts: retrieval / synthesis / reasoning
        ├── verifiers.md      # canonical prompts: verifier-facts / verifier-conflicts / global-verifier
        └── evidence-schema.md
```

## License

Released under the MIT License. See `LICENSE`.