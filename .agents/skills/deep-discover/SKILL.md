---
name: deep-discover
description: Verification-centric multi-agent research system for any question whose answer depends on evidence the model doesn't already reliably hold — any topic, any domain. Use whenever a single-pass answer would be unreliable: deep research, "is X true about Y", comparing options, auditing a claim, explaining a complex phenomenon, or any multi-faceted question where getting it wrong has consequences. Triggers on phrases like "deep research", "investigate", "verify whether", "find evidence", "what's the real story on", "what do we know about", or any question the user wants grounded in real evidence rather than a quick guess. Produces an answer plus an explicit evidence trail and verification notes.
---

# Deep Discover

A verification-centric multi-agent research system. You (the main session running this skill) are the **orchestrator**. You never answer the question yourself — you decompose it, dispatch **workers** to gather and synthesize evidence, dispatch **verifiers** (separate context, separate prompt) to check the workers' claims against the evidence, and only assemble a final report once the verification gate passes.

The invariant that makes this work: **the agent that produces a claim never grades it.** Verifiers run in isolated contexts with only the artifact + the evidence pool — never the worker's reasoning trace. This is the verifier pattern (generator-verifier loop) applied at orchestration level.

## When to run

This skill is for questions where a single-pass answer is risky — the cost of being wrong (giving bad advice, asserting a false technical claim, missing a contradiction) is higher than the cost of spending more tokens. For a one-line factual lookup or a trivial edit, just answer directly. For anything with multiple facets, conflicting sources, or real downstream impact, run the loop.

## The loop

### 0. Set up the session and run directory

**Two layers: session and run.** A *session* groups all the queries from one conversation. A *run* is one query's research. Multiple runs in a session are linked so follow-up queries can build on prior evidence.

#### Session layer

On the first deep-discover query in a conversation, create the session:

```
.deep-discover/sessions/<session-id>/
├── manifest.json          # the session manifest (see below)
└── runs/
    └── <run-slug>/        # one run dir per query (the {RUN_DIR} from before)
```

Generate `<session-id>` as a short slug from the date + first topic: e.g. `2026-07-16-freemasonry`. If the same session-id dir already exists, append a counter.

**The session manifest** (`sessions/<session-id>/manifest.json`) records every run in the session, in order, with how it relates to the others:

```json
{
  "session_id": "2026-07-16-freemasonry",
  "created": "ISO-8601",
  "runs": [
    {
      "run_id": "entered-apprentice",
      "slug": "entered-apprentice",
      "query": "do a deep research on freemason first degree",
      "grounding": "Masonic studies — ritual, history, esotericism, sociology",
      "started": "ISO-8601",
      "completed": "ISO-8601",
      "cycles": 1,
      "claims_verified": 14,
      "claims_disputed_final": 0,
      "builds_on": [],
      "related_to": [],
      "report_summary": "one-line summary of what the run found"
    },
    {
      "run_id": "fellowcraft",
      "query": "research the second degree",
      "builds_on": ["entered-apprentice"],
      "related_to": ["entered-apprentice"],
      "inherited_evidence": ["EV-0040", "EV-0041"],
      "report_summary": "..."
    }
  ]
}
```

Update the manifest **every time a run starts** (append the run entry with `started` timestamp) and **every time a run completes** (fill in `completed`, `cycles`, `claims_verified`, `report_summary`).

#### Run layer (within the session)

Each query gets its own run directory under `sessions/<session-id>/runs/<slug>/`:

```
sessions/<session-id>/runs/<slug>/
├── evidence/
│   ├── EV-0001.json
│   ├── EV-0002.json
│   └── ...
├── evidence_graph.json
├── plan.md
├── assessments/
│   ├── round-1.md
│   └── round-2.md
├── report.md
└── verification_report.md
```

Derive `<slug>` from the query (lowercase, hyphenated, max ~40 chars). If a slug collides within the session, append `-2`, `-3`.

#### Cross-run linking (the key addition)

When a new query in an existing session is **related to a prior run** (same topic, follow-up, or shares evidence), the new run can:

1. **Declare the relationship** in `plan.md`: "Builds on `<prior-run-slug>`. Inherits evidence EV-0040, EV-0041 (the 2-to-3 degree split). New sub-questions: ..."
2. **Link the evidence** by copying (not moving) specific EV files from the prior run's `evidence/` into the new run's `evidence/`, preserving their original IDs and adding an `inherited_from: "<prior-run-slug>"` field. This lets the new run's verifiers see the inherited evidence without re-searching.
3. **Record the link** in the manifest: `builds_on: ["<prior-run-slug>"]`, `inherited_evidence: ["EV-0040", ...]`.
4. **Reference the prior report** in the new report's confidence section if the new findings depend on or extend prior findings.

Do **not** blindly inherit all evidence — only the specific EV files the new run's decomposition actually needs. Inheriting everything defeats the isolation that makes verification clean.

#### Context-window survival (for very long sessions)

If the session has many runs and the conversation context is getting long, the manifest is the durable summary. A new session can pick up where an old one left off by reading the manifest: it lists every run, its grounding, its report summary, and its relationships. This lets research continue across context resets without losing the thread.

Pass the absolute `{RUN_DIR}` path to every worker and verifier you dispatch.

Each evidence file is one atomic unit:

```json
{
  "id": "EV-0001",
  "source": { "type": "web|repo|doc|tool-run|reasoning", "locator": "url or file:line or tool call id" },
  "type": "raw-quote|summary|statistic|code|claim|hypothesis",
  "summary": "one-line what this evidence says",
  "content": "the actual text / snippet / figure, kept verbatim where it matters",
  "collected_by": "retrieval-worker-1",
  "supports": ["C-0001"],
  "contradicts": [],
  "confidence": "high|medium|low",
  "timestamp": "ISO-8601"
}
```

`evidence_graph.json` links claims to evidence:

```json
{
  "claims": {
    "C-0001": { "text": "X is true because Y", "supports_evidence": ["EV-0001"], "contradicts_evidence": ["EV-0007"], "status": "verified|disputed|unverified", "verifier_notes": "..." }
  },
  "links": [ { "from": "EV-0001", "to": "C-0001", "relation": "supports" } ]
}
```

**All agents write only to `evidence/`.** Workers append evidence. Verifiers append `verifier_notes` to existing claims in `evidence_graph.json` and may add new evidence of type `claim` (e.g. "this source actually says the opposite"). The orchestrator and verifiers read from the pool; workers do not read each other's traces.

Pad the EV id to 4 digits, increment atomically. Keep evidence small and atomic — one fact per file, not a dump.

### 1. Ground the query (before any worker is spawned)

This step exists because a single misread term sends the entire run down the wrong branch, and workers are isolated — once dispatched on the wrong interpretation, there is no cheap recovery. Verifiers can only check whether claims match evidence; they cannot check whether the evidence answers the question the user actually asked. So you resolve that *before* decomposition.

Do this:

1. **Read the query literally first.** Do not immediately reframe it in a domain you're familiar with. Every key term could have senses in domains you don't default to. Resist the pull to map ambiguous terms onto the field you're most comfortable in.
2. **List the key terms and their candidate senses.** For each term that has more than one plausible reading, write out the readings and which field each reading belongs to. Do not assume any term has an obvious sense — "model", "light", "force", "culture", "memory" all mean different things in different fields.
3. **Name the candidate domains** the question could live in, and note which signals in the query (phrasing, vocabulary, what the user likely cares about) point where. Treat any signal as a signal — don't weight the ones from your familiar domain more heavily.
4. **If the sense is still ambiguous after this, ask the user — one focused question, not a wall of options.** Use AskUserQuestion with the candidate senses as options. This is the only cheap place to correct course. Do not skip this even if you feel "fairly sure" — if there's more than one plausible sense and the cost of being wrong is a wasted multi-worker run, ask.
5. **Write the chosen interpretation to `{RUN_DIR}/plan.md` as an explicit grounding decision** before any decomposition: "Domain: X. Sense of term Y: Z. Senses considered and rejected: …". This is the contract the rest of the run operates under. If you later realize the grounding was wrong, stop the run and re-ground rather than silently continuing.

Only after the grounding decision is recorded do you proceed to decomposition. Never spawn workers on an unstated interpretation.

### 2. Decompose (you, the orchestrator)

Read the question. Use thinking to plan: what are the independent sub-questions, what would count as sufficient evidence, how many workers does this warrant (default 3–5, scale down for narrow questions), and what are the specific verification criteria each claim must meet.

Write a short plan to the run dir:

```
{RUN_DIR}/plan.md
```

containing: the original question, the grounding decision from step 1, the decomposed sub-questions, the worker assignments, the verification criteria, and a stop-condition budget (e.g. "max 4 cycles, stop when ≥80% of claims are verified and no HIGH-severity disputes remain"). This is your scratchpad — consult it between rounds so you don't drift. Write it to `{RUN_DIR}/plan.md`.

### 3. Dispatch workers (parallel)

Spawn worker subagents via the `Agent` tool **in a single message** so they run concurrently. Each worker gets:
- its role (retrieval / synthesis / reasoning — see `references/workers.md` for the full prompts)
- its specific sub-question
- the output format: append evidence files to `{RUN_DIR}/evidence/` and return a list of the EV ids it created

Workers do **not** see each other's prompts or outputs. They only see their own assignment. That isolation is what makes later verification meaningful.

Example dispatch (3 retrieval workers, 1 reasoning worker):

```
Agent(subagent_type="general-purpose", prompt="<retrieval worker prompt for sub-question A>")
Agent(subagent_type="general-purpose", prompt="<retrieval worker prompt for sub-question B>")
Agent(subagent_type="general-purpose", prompt="<retrieval worker prompt for sub-question C>")
Agent(subagent_type="general-purpose", prompt="<reasoning worker prompt>")
```

Read `references/workers.md` before dispatching — it has the canonical worker prompts. Pass the relevant prompt body into each Agent call; don't paraphrase it away.

### 4. Verify (the gate — never skip)

After workers return, read the evidence pool. For every claim a worker asserted, run a verifier. Verifiers are spawned the same way (Agent tool, isolated context) but with verifier prompts from `references/verifiers.md`. Three verifier types:

- **verifier-facts** — for each specific claim, check it against the cited evidence. Does EV-0001 actually say what the claim says? Decompose hard claims into small source-checkable sub-questions (the verification-asymmetry principle: checking is easier than generating).
- **verifier-conflicts** — scan the whole pool for contradictions between evidence units or between workers' claims. Flag pairs where EV-X and EV-Y disagree.
- **global-verifier** — at the end, review the full draft against the full evidence set. Are there claims with no backing evidence? Are there evidence units no claim uses? Annotate risk and uncertainty.

The orchestrator **never trusts a worker's conclusion until a verifier signs off.** A worker saying "X is true" is a hypothesis (status `unverified`) until verifier-facts promotes it to `verified` or flags it `disputed`.

Verifiers write their findings back to the evidence pool — they append `verifier_notes` to claims in `evidence_graph.json` and may create new evidence of type `claim` (e.g. "EV-0001 is a blog post, not a primary source — downgrading confidence").

### 5. Assess (the cycle's decision point)

This is where the skill becomes a cyclic graph. The skill is **not a linear pipeline with a repair patch** — it is a Reason → Verify → Correct loop, where each cycle can re-decompose the question, not just patch broken claims. The verify step's output feeds back into decomposition, allowing the *act of researching* to surface new sub-questions that the original plan didn't anticipate. This is the difference between error-correction and genuine discovery.

After the verify step, before any synthesis, you (the orchestrator) read the evidence pool and the verifier verdicts and ask three questions explicitly:

1. **Are there open disputes?** Claims still `disputed` or `unverified` at HIGH severity need targeted correction. This is the repair path.

2. **Did the evidence surface new sub-questions that weren't in the original plan?** This is the discovery path — the one the previous linear version of this skill couldn't do. A verifier-facts check might reveal that a cited source actually points at a deeper question (e.g. "the paper says X is true *if* condition Y holds — but we never searched for whether Y holds"). A verifier-conflicts finding might reveal that two claims are in tension in a way that needs a new retrieval worker to resolve, not just a softening of one claim. The reasoning worker might have produced a claim that, now that you read it, depends on a sub-question no worker ever searched. **List these new sub-questions explicitly.**

3. **Is the evidence sufficient to answer the user's question, or is it thin in a way that isn't a dispute but a gap?** Thinness is not the same as a dispute. If three workers returned evidence about the ritual but the history angle has only one weak source, that's a gap. The linear skill would have shipped it as a low-confidence section; the cyclic skill dispatches a targeted worker to fill it.

Write the assessment to `{RUN_DIR}/assessments/round-N.md` (create the directory on first use), recording: which claims need correction, which new sub-questions were surfaced, which gaps need filling, and the current cycle count.

### 6. Cycle or stop (the loop with guards)

The loop is: **Assess → Re-decompose → Dispatch new/targeted workers → Verify → Assess again.** Each pass through this loop is one cycle. The loop is what makes this a cyclic graph (Reason → Verify → Correct), not a linear pipeline.

**Guards against infinite loops — mandatory, non-negotiable:**

1. **Max 4 cycles total** (including the initial dispatch as cycle 1). After cycle 4, you stop and synthesize with whatever you have, reporting open gaps and disputes as explicit uncertainty. Do not exceed 4 cycles regardless of how tempting the next search feels.

2. **Diminishing-returns guard.** Track the number of claims promoted from `unverified`/`disputed` to `verified` in each cycle. If a cycle promotes **fewer than 2 claims** or resolves **fewer than 1 HIGH-severity dispute**, stop — you have hit diminishing returns and another cycle is unlikely to help. Synthesize with current evidence.

3. **No-rework guard.** A claim that has been the target of correction in two consecutive cycles and is *still* `disputed` is declared a **permanent dispute** — stop trying to fix it. It becomes an explicit uncertainty in the final report, with the verifier notes attached. Do not spend a third cycle on it.

4. **Budget guard.** If the total evidence pool exceeds **400 EV files**, stop dispatching new workers regardless of cycle count. The pool is large enough; further growth will make verification harder, not easier. Synthesize.

5. **Convergence check.** Before dispatching in any cycle after the first, confirm that the new sub-questions from the Assess step are *genuinely new* — not a rephrasing of a sub-question a previous cycle already searched. If you can't articulate how the new search differs from a previous one, don't dispatch. This catches the failure mode where the orchestrator keeps searching "the same thing but harder" instead of recognizing the evidence isn't out there.

**The stop decision is: have ALL of these been met?**
- No HIGH-severity disputes remain, OR remaining HIGH disputes have been declared permanent (guard 3).
- No genuinely new sub-questions left unsearched (convergence guard 5).
- The evidence is sufficient to answer the user's question at the confidence the evidence supports — thinness in non-load-bearing areas is acceptable and reported as such.

If yes → proceed to synthesis (step 7). If no → proceed to the next cycle: re-decompose (add the new sub-questions from the assessment to the plan), dispatch targeted workers for the gaps/disputes/new-questions, verify the new and re-corrected claims, then assess again.

**What a cycle looks like in practice:**

- **Cycle 1** (always): the initial dispatch from step 3 + the first verify from step 4 + the first assess from step 5.
- **Cycle 2+**: dispatch only the targeted workers identified by the assess step (the specific claims needing correction, the specific new sub-questions, the specific gaps). Do not re-dispatch workers whose evidence already passed verification. Preserve all earlier evidence — the pool is append-only and cumulative. Re-verify only the new and re-corrected claims (don't re-run verifier-facts on claims already verified). Then assess again.

Each cycle should be smaller and more targeted than the one before. If a cycle is as large as the initial dispatch, something is wrong — you're re-searching instead of searching the frontier.

### 7. Synthesize the final report

Only after the gate passes (or the budget forces a stop) do you write the report. It has two parts:

**Answer** — the actual response to the user's question, written from verified evidence only. Every factual sentence cites its evidence by id, e.g. "X is true (EV-0001, EV-0003)."

**Evidence & verification notes** — an explicit list:
- every evidence id used, with its source and a one-line summary
- the verification status of each claim (verified / disputed / unverified)
- any unresolved disputes or gaps, with severity
- a confidence summary for the answer as a whole

Write the report to `{RUN_DIR}/report.md` and also surface it to the user in your reply. The on-disk report is the durable artifact; the reply is the user-facing summary. When the report is done, **update the session manifest** with the run's `completed` timestamp, `cycles`, `claims_verified`, and `report_summary`.

## How to think about failure

- A worker that returns a confident conclusion with no evidence ids is not done — re-dispatch or discard.
- A verifier that says "looks good" without citing specific evidence is not a verification — re-run with a stricter prompt.
- A dispute that survives 2 consecutive correction cycles is a permanent dispute (guard 3). Report it, don't hide it, and stop trying to fix it.
- If you catch yourself writing the answer before the cycle's stop conditions are met, stop. You're the orchestrator, not a worker.
- If you catch yourself writing a factual sentence in the report that doesn't cite an EV id, stop. The report is built only by citing evidence from the graph — no imported-from-memory assertions. The global verifier will fail you for this; better to catch it yourself.
- If you realize mid-run that the grounding decision (step 1) was wrong — the evidence coming back is answering a question the user didn't ask — stop the run and re-ground. Do not silently pivot.
- If you find yourself dispatching a worker for a sub-question that a previous cycle already searched, stop and check the convergence guard (step 6, guard 5). "The same thing but harder" is not a new sub-question. The evidence may simply not exist, and that's an explicit uncertainty, not a reason for another cycle.
- If you hit cycle 4 (guard 1), the 400-EV cap (guard 4), or the diminishing-returns threshold (guard 2), stop. These guards are not suggestions. They exist because the alternative is an infinite loop that burns budget without improving the answer. A stopped run with explicit uncertainty is better than a run that never ends.
- If a new query in a session is related to a prior run, **declare the relationship** in `plan.md` and the session manifest (`builds_on`, `related_to`) before dispatching. Don't silently re-search evidence a prior run already gathered — inherit the specific EV files and build on them.
- If you're starting a query and a prior run in the same session already covered the topic, check whether this is a genuine new question or a re-run. If it's a follow-up, inherit evidence and link the runs. If it's a re-run, ask the user whether to overwrite or create a `-2` run.
- If the session has many runs and the context is getting long, read the session manifest to recover the thread — it lists every run, its grounding, and its summary. The manifest is the durable memory that survives context truncation.

## Reference files

Read these on demand when you reach that stage:

- `references/workers.md` — canonical prompts for retrieval, synthesis, and reasoning workers. Read before dispatching workers.
- `references/verifiers.md` — canonical prompts for verifier-facts, verifier-conflicts, and global-verifier. Read before dispatching verifiers.
- `references/evidence-schema.md` — full schema and worked examples for the evidence pool and graph. Read if you're unsure how to structure an evidence unit or a claim.