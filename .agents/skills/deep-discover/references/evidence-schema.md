# Evidence pool schema

The evidence pool is an append-only store. All agents write here; only verifiers and the orchestrator read across the whole pool. Keep evidence atomic — one fact per file.

## Directory layout

```
{RUN_DIR}/
├── plan.md                 # orchestrator's scratchpad
├── evidence/
│   ├── EV-0001.json
│   ├── EV-0002.json
│   └── ...
├── evidence_graph.json     # claims + links, mutated by verifiers
├── report.md               # final report (written last)
└── verification_report.md  # global verifier's output
```

## Evidence unit (one file per EV id)

```json
{
  "id": "EV-0001",
  "source": {
    "type": "web | repo | doc | tool-run | reasoning",
    "locator": "https://example.com/...  OR  path/to/file.py:42  OR  tool-call-id"
  },
  "type": "raw-quote | summary | statistic | code | claim",
  "summary": "one-line gloss of what this evidence says",
  "content": "the actual text. Verbatim for quotes/code/numbers; can be a restatement for summaries.",
  "collected_by": "retrieval-worker-1 | synthesis-worker-1 | reasoning-worker-1 | verifier-facts-1 | verifier-conflicts-1",
  "supports": ["C-0001"],
  "contradicts": [],
  "confidence": "high | medium | low",
  "timestamp": "2026-07-16T12:00:00Z"
}
```

Field rules:

- **`id`** — zero-padded to 4 digits, monotonically increasing. Scan the directory before writing to find the next free number.
- **`source.type`** — `web` (fetched URL), `repo` (file in the codebase), `doc` (local doc / README), `tool-run` (output of a command or tool call), `reasoning` (inferred, not directly observed — use sparingly, this is the weakest evidence).
- **`source.locator`** — must be precise enough that a verifier can re-fetch or re-read the same thing. A bare domain is not a locator. A URL with fragment, or `file:line`, is.
- **`type`** — `raw-quote` (verbatim text from a source), `summary` (a restatement produced by a synthesis worker), `statistic` (a number with units), `code` (a snippet), `claim` (an assertion made by a reasoning worker or a verifier — claims live in the pool too, and are what the graph tracks).
- **`content`** — for `raw-quote`, `statistic`, and `code`, keep verbatim. For `summary`, a clean restatement is fine. For `claim`, the precise assertion.
- **`supports` / `contradicts`** — ids of claims (C-####) this evidence bears on. Can be empty. A `claim`-type unit's own id is what goes in the graph, not here.
- **`confidence`** — high = primary source, direct quote, or re-verified locator; medium = reputable secondary source; low = blog, forum, inference, or stale source.

## Claim id convention

Claims use a separate id space: `C-0001`, `C-0002`, etc. A claim is created as a `claim`-type evidence unit (so it has both an EV id and a C id — the EV id is the file, the C id is the logical claim). In practice: a reasoning worker creates `EV-0031.json` with `type: "claim"` and registers `C-0001` in the graph pointing at `EV-0031`. Keep the two id spaces separate so evidence and claims don't collide.

## evidence_graph.json

```json
{
  "claims": {
    "C-0001": {
      "text": "X is true because Y",
      "evidence_file": "EV-0031",
      "supports_evidence": ["EV-0001", "EV-0003"],
      "contradicts_evidence": ["EV-0007"],
      "status": "verified | disputed | unverified",
      "verifier_notes": "EV-0001 is the primary paper, figure confirmed. EV-0007 is a 2019 blog post predating the paper — not a real contradiction, downgraded.",
      "verified_by": "verifier-facts-1"
    }
  },
  "links": [
    { "from": "EV-0001", "to": "C-0001", "relation": "supports" },
    { "from": "EV-0007", "to": "C-0001", "relation": "contradicts" }
  ]
}
```

`status` transitions:
- a reasoning worker creates a claim with `status: "unverified"`.
- `verifier-facts` sets it to `verified` or `disputed` and fills `verifier_notes` + `verified_by`.
- `verifier-conflicts` can downgrade `verified` → `disputed` if it finds a cross-evidence contradiction.
- the orchestrator never sets `status` directly — only verifiers do.

## Worked example

Question: "Is framework X faster than framework Y on workload Z?"

`EV-0001` (retrieval worker, web, raw-quote): the framework X benchmark page, verbatim row for workload Z, latency 12ms.
`EV-0002` (retrieval worker, web, raw-quote): the framework Y benchmark page, verbatim row for workload Z, latency 18ms.
`EV-0003` (retrieval worker, web, summary): a third-party blog comparing X and Y, noting X is faster but cautioning the benchmark config differs.
`EV-0031` (reasoning worker, claim, registered as `C-0001`): "X is faster than Y on workload Z (12ms vs 18ms)."
`EV-0050` (verifier-conflicts, claim): "EV-0003 cautions the configs differ — the 12ms vs 18ms comparison may not be apples-to-apples."
`C-0001` ends `disputed` with verifier notes pointing at EV-0050.

Final report says: "X reports 12ms vs Y's 18ms on workload Z (EV-0001, EV-0002), but the comparison is disputed because the benchmark configurations differ (EV-0050). Confidence: medium." That's a verified-centric answer — it states what the evidence supports and where the uncertainty is, rather than asserting "X is faster."