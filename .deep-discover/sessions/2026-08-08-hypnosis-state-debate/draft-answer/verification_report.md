# Verification Report — Hypnosis State Debate (DRAFT-ANSWER condition)

## global-verifier verdict: **PASS** | Confidence: **Medium**

---

## Method
global-verifier reviewed the final draft report (`report.md`) against the complete evidence pool (`evidence/EV-0001..EV-0036`) and the evidence graph (`evidence_graph.json`). The verifier received only the report + evidence artifacts (per the verifier-isolation rule) — not the reasoning traces.

## (a) Assertions that passed
Every factual sentence in the report cites ≥1 EV id, and all cited evidence is `verified` (not `disputed`/`unverified`) in the graph:

- "方向一致但異質" — EV-0001, EV-0002, EV-0007, EV-0014 ✓
- "後部區域最可靠" — EV-0002, EV-0007, EV-0028 ✓
- "DLPFC 三特徵主要 Spiegel lab，未獨立乾淨重複" — EV-0029 ✓ (provenance claim, verified)
- "ALE vs FC 方法學差異" — EV-0028 ✓ (with one [INFERRED] bridge explicitly marked in draft-answer-round-3 as non-load-bearing)
- "純順從被排除" — EV-0017, EV-0008, EV-0013 ✓
- "r=0.67/0.82; 誘導增量小" — EV-0008, EV-0010, EV-0018 ✓
- "療效真實但適度; 49 元分析/261 RCT; 輔助止痛 n=6078" — EV-0024, EV-0025 ✓
- "sham ~80% 效果 (53.4% vs 40.9%)" — EV-0033 ✓
- "正式誘導不產生可靠 EEG 變化" — EV-0034 ✓
- "預期不預測減壓" — EV-0035 ✓
- "PC 匯合為理論性; 基礎 PC modest" — EV-0026, EV-0027 ✓
- "狀態/非狀態二分退休" — EV-0009, EV-0011, EV-0012, EV-0015, EV-0016 ✓

## (b) Assertions that failed or were uncited
**None.** No factual sentence lacks a citation. Two [INFERRED] synthesis bridges exist (D2's "ALE-vs-FC methodological" sub-clause; D8 the final conclusion framing) — both are explicitly flagged as non-load-bearing framing in `draft-answer-round-3.md`, not presented as new facts.

## (c) Unresolved disputes
- **No HIGH-severity disputes** remain. The cycle-1 C-0001 framing dispute was resolved by supersession (C-0001-REV).
- **One explicitly-reported open empirical question**: the size/mechanism of the hypnosis-specific effect beyond expectancy/placebo (C-0008). This is reported prominently in the report's TL;DR, section 5, and the unresolved-questions list — NOT hidden.
- The state-vs-nonstate dichotomy itself is reported as resolved-in-favor-of-retirement (C-0003), with the caveat that the replacement (PC framework) is theoretically appealing but not yet decisively tested (C-0007).

## (d) Confidence rating justification
**Medium.** The core conclusion (both extremes unsupported; expectancy contributes substantially; small real specific residual; PC convergence is theoretical) rests on 8 verified claims backed by 36 evidence units including 3 systematic reviews/meta-analyses (EV-0001, EV-0024, EV-0025), 1 ALE meta-analysis (EV-0028), and 1 sham-controlled RCT (EV-0033). Direction of findings is robust across independent groups. Confidence is held at Medium (not High) because: (1) heterogeneity is pervasive and explicitly limits definitive conclusions (EV-0001, EV-0014, EV-0024); (2) the load-bearing "hypnosis-specific beyond expectancy" evidence rests on few studies (EV-0033 single RCT, EV-0034 n=31, EV-0035 n=47); (3) the unifying PC framework is not yet decisively tested (EV-0026, EV-0027); (4) some neural-signature evidence is concentrated in one research group (EV-0029).

## FAIL check
No HIGH-severity assertion failed or was uncited. → **PASS.**

---

## Cycle/operator provenance (for the A/B-test auditor)
- Cycles run: 3 (of max 4)
- EV count: 36 (of max 400)
- Operators that fired and produced genuinely new sub-questions:
  - **Cycle 1:** Negation (×2), Pairwise (×1: ALE-vs-DMN/SN/ECN), Provenance (×1: Spiegel-lab), Boundary (×1), Analogy (×1: placebo), Implication (×1, overlapped D9), Absence (×1: efficacy meta), **draft-answer (×3: D8 INFERRED + D9 + D10 UNBACKED)**.
  - **Cycle 2:** Negation+Absence+Analogy+draft-answer converged on 1 (hypnosis-vs-placebo).
  - **Cycle 3:** 7 operators produced 0 new; draft-answer produced 0 [UNBACKED] → closure.
- The **draft-answer operator specifically** drove cycles 2 and 3 via [UNBACKED] tags D9/D10 (cycle 1) and the hypnosis-vs-placebo gap (cycle 2). Without it, the run would likely have stopped at cycle 2 with two load-bearing gaps (PC empirical status, efficacy meta-analysis) unaddressed.
