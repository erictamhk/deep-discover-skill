# Plan — Experimental Run (Generative Operators Patch)

## Original question
古埃及人到底怎麼建造吉薩大金字塔的？我們現在對它的建造方法、勞動力、和運輸技術，實際知道什麼？

## Grounding decision (from shared initial plan, used as-is)
- Domain: Egyptology / ancient construction archaeology.
- "建造方法" = construction methods (how blocks were moved and lifted).
- "勞動力" = labor force (size, composition, social organization).
- "運輸技術" = transport technology (quarry to site, lifting).
- The question asks what we ACTUALLY know — established consensus vs. contested vs. genuinely unknown.

## Initial decomposition (sub-questions, from shared plan)
1. What are the currently accepted theories for how the blocks were moved and lifted (external ramp, internal ramp, water transport, etc.)?
2. What does the archaeological evidence actually say (Merer papyrus, quarries, tool marks, the 2024 Nile branch discovery)?
3. What do we know about the labor force (size, slave vs. paid, social organization)?
4. What is the timeline and logistics of construction (block count, quarry-to-site transport)?
5. What is genuinely unknown / contested vs. established consensus?

## Worker assignments (cycle 1)
- Worker 1 (retrieval): sub-question 1 — construction/lifting theories
- Worker 2 (retrieval): sub-question 2 — archaeological evidence
- Worker 3 (retrieval): sub-question 3 — labor force
- Worker 4 (retrieval): sub-question 4 — timeline & logistics
- Worker 5 (reasoning): sub-question 5 — what's known vs unknown/contested

## Verification criteria
Each claim must cite specific evidence. Claims about "what is known" must distinguish established consensus from contested from unknown.

## Stop-condition budget (EXPERIMENTAL — operators patch)
- Use the CLOSURE-SATURATION stop condition: stop only when applying all 7 generative operators to the current pool produces NO genuinely new sub-questions.
- A failed search is evidence of absence (write as EV of type claim), not a stop.
- A permanent dispute spawns the meta-question "why do sources disagree?" and gets probed.
- Hard ceilings: max 4 cycles, 400 EV cap. Do NOT stop on claim-promotion diminishing returns; measure diminishing returns on NEW SUB-QUESTIONS produced (0 new sub-questions from all operators → stop).

## Cycle log
- Cycle 1: initial 5-worker dispatch + verify + assess (all 7 operators). 10 new sub-questions surfaced.
- Cycle 2: 5 targeted workers (ScanPyramids, granite beams, labor across periods, harbor, Houdin provenance). 4 new sub-questions surfaced.
- Cycle 3: 3 targeted workers (NFC function, hydraulic lift, cross-cultural). 4 new sub-questions surfaced.
- Cycle 4: 3 targeted workers (NFC extent, harbor transport-vs-lift, non-Egyptologist provenance). 0 new sub-questions → closure saturated. STOP.

## Final state
- Cycles run: 4 (hard ceiling reached).
- Evidence pool: EV-0001..EV-0038 (38 files).
- Claims: C-0001..C-0011, all verified.
- Stop reason: closure-saturation (0 new sub-questions from all 7 operators) + 4-cycle hard ceiling.
- Report: report.md. Verification: verification_report.md (PASS, confidence medium).
