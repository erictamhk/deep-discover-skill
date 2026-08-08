# Verification Report — control run (hypnosis state-vs-compliance)

**Global verifier verdict: PASS**
**Overall confidence: MEDIUM**

## Method
Three verifiers run in isolated roles against the evidence pool (25 EV files) and evidence_graph.json (8 claims): verifier-facts (per-claim), verifier-conflicts (cross-evidence), global-verifier (assembly check against report.md).

## (a) Assertions that passed
- All 8 claims in report.md cite at least one EV id, and all cited evidence is `verified` in the graph (C-0001 is `disputed` but the report explicitly marks it as such and softens its framing rather than asserting it as fact — compliant).
- Key load-bearing facts were independently re-verified against multiple locators:
  - Landry 2017 lingual-gyrus + heterogeneity: confirmed via 3 independent sources (PMID 28238944, ScienceDirect S0149763416306030, EuropePMC).
  - Braffman & Kirsch 1999 "substantial minority less suggestible post-induction": confirmed (PMID 10510510 abstract); the 54% figure is cited via Lynn 2023 (EV-0004).
  - SHIFT RCT (Faerman 2024): confirmed via 3 loci (Nature Mental Health s44220-023-00184-z, medRxiv 10.1101/2021.07.08.21260222, ClinicalTrials.gov NCT02969707). Design (preregistered, double-blind, RCT, N=80) and P-values verified verbatim.
  - Kirsch 2000 response-set quote: verbatim-accurate per EV-0003.
- No claim asserts a fact not traceable to a re-fetchable source locator.

## (b) Assertions that failed or were uncited
- None. No factual sentence in report.md lacks an EV citation. (Spot-checked the 5 numbered answer points + the established/contested table — all cite.)

## (c) Unresolved disputes / uncertainty marked in the report
- C-0001 (DISPUTED, MEDIUM): the report correctly does NOT use C-0001's "therefore a real altered state" framing as fact; instead it frames the brain-change finding as "real neurobiological change" and explicitly separates it from the contested "unique state" interpretation. The report's answer point #1 and the established/contested table both mark this distinction. COMPLIANT.
- C-0006 (genuinely unresolved: causal status of the state construct): explicitly reported as open in answer point #5 and the "真正未解" section. COMPLIANT.
- The induction-vs-suggestion methodological confound (EV-0013) is explicitly cited as the reason the stalemate persists. COMPLIANT.

## (d) Confidence rating justification
MEDIUM. The two extremes (pure compliance; unique-trance-required) being rejected is HIGH-confidence (multiple primary sources, converging). The "integrative position is consolidating" point (C-0004) is MEDIUM — these are active recent (2022–2024) efforts, not a settled consensus. The deepest question (causal status of the state construct) is, by the field's own admission (EV-0013), unresolved by current evidence; this caps overall confidence at MEDIUM. No HIGH-severity assertion failed the gate. The report does not overstate; it marks the genuine epistemic uncertainty as uncertainty rather than smuggling in a conclusion.

## Verifier conflicts surfaced
- The apparent C-0001 vs C-0002 conflict is a framing disagreement, not a factual contradiction (EV-0022). Severity MEDIUM. Resolved by reframing in the report.
- Moujaes 2023 (hypnosis neurally distinct from meditation/psychedelics) vs Lynn 2023 ("no robust neurophysiological evidence for a special state"): a MEDIUM interpretive tension. distinctiveness ≠ unique-trance-state. Reported honestly; does not downgrade either claim.

## Notes for the auditor (process honesty)
- This run used the OLD step 5/6 (no 7 operators, no closure-saturation). The convergence guard (OLD guard 5) caught one new sub-question at cycle 1 and pushed to cycle 2, which surfaced the SHIFT RCT — the strongest single piece of evidence in the pool. This is a genuine win for the OLD skill.
- The run STOPPED at cycle 2 because OLD guard 2 (claim-promotion count) fired: cycle 2 promoted only 1 claim (<2). This is the brittle dynamic the A/B test targets: a cycle that surfaces a single decisive RCT is high-value but gets read as "diminishing returns" under the claim-promotion metric.
- The 7 operators were NOT applied; only passive "did anything interesting come up?" discovery was used. Two known gaps (2023–2025 EEG primaries; clinical-implications angle) were never searched because nothing surfaced them. This is the premature-convergence risk the hypothesis predicts.
