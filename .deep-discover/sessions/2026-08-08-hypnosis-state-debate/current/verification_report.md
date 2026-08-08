# Verification Report (global-verifier) — Hypnosis state/non-state run

## Verdict: PASS
## Confidence: MEDIUM-HIGH

## Method
Reviewed `{RUN_DIR}/report.md` against the full evidence pool (`{RUN_DIR}/evidence/`, EV-0001..EV-0035) and `{RUN_DIR}/evidence_graph.json` (8 claims: C-0001..C-0008). Checked: (a) every factual assertion cites ≥1 EV id and the cited EV is `verified`; (b) no evidence unit was silently dropped; (c) all `disputed`/`unverified` claims are explicitly marked uncertain in the report.

## (a) Assertions that PASSED
Every load-bearing assertion in report.md cites EV ids, and all cited claims are `verified` in the graph:
- "measurable brain changes ARE real" → C-0001 (verified), EV-0001/0002/0003/0004/0019. PASS.
- "no hypnosis-specific unique fingerprint" → C-0002 (verified), EV-0005/0006/0007/0008/0009/0016/0020. PASS.
- "strict trance-required hypothesis ruled out" → C-0005 (verified), EV-0008/0016. PASS.
- "DLPFC inhibition causally increases hypnotizability beyond expectancy" → C-0007 (verified), EV-0029/0030/0031. PASS. Note EV-0030 explicitly states expectancy did not differ between rTMS conditions — directly supports "beyond expectancy."
- "49 meta-analyses / 261 RCTs, 99.2% positive, hypnotizability moderator r=0.24" → C-0008 (verified), EV-0033. PASS.
- "predictive coding dissolves dichotomy" → C-0004 (verified-but-scoped). Report correctly marks this as "a set of proposals with testable-but-unconfirmed predictions" — scope is explicit. PASS.
- "Wagstaff 2026 no necessary contradiction" → EV-0034. PASS.

## (b) Evidence units with no claim reference
- EV-0001..EV-0020 (raw retrieval), EV-0021..EV-0026 (reasoning-worker claims), EV-0027/0028 (verifier outputs), EV-0029..EV-0031 (cycle-2 retrieval), EV-0032/0035 (cycle-2 claims), EV-0033/0034 (cycle-2 retrieval) — all are referenced by ≥1 claim in the graph or in the report's evidence list. No silent drops. The raw retrieval EVs (e.g., EV-0001, EV-0002) are referenced via the claims they support (C-0001 etc.).

## (c) Unresolved disputes explicitly marked
- The HIGH interpretive conflict (state vs epiphenomenal reading of same imaging) is explicitly marked in report.md under "Contested" and under "Genuinely unresolved / declared permanent dispute per guard 3, redirected." PASS.
- C-0004's scoped status is explicitly flagged ("a theoretical-trend claim, not an established causal fact"). PASS.
- C-0006's open axes (causal-vs-epiphenomenal, trait-vs-state, which-mechanism) are explicitly listed as "Genuinely unresolved." PASS.

## Failures
None at HIGH severity. No uncited assertions. No unverified claim stated as fact.

## Confidence justification
The synthesis answer (dichotomy is false; both extremes disconfirmed; brain changes real and clinically efficacious; specific mechanism underdetermined) rests on a broad, convergent, mostly-primary-source evidence base spanning 2023-2026, including a preregistered RCT (Faerman SHIFT), a preregistered Stroop study (Dienes), and a 49-meta-analysis meta-review (Kekecs/Angle). The main reason confidence is MEDIUM-HIGH rather than HIGH: (1) the predictive-coding synthesis is theoretically attractive but empirically under-tested; (2) the specific-mechanism question is genuinely underdetermined by current evidence (the leading theories make overlapping predictions on tested manipulations); (3) the neuroimaging literature has acknowledged heterogeneity (ALE meta-analysis did not confirm predicted networks). These are reported as explicit uncertainty, not hidden.
