# Plan — Control run

## Original question
古埃及人到底怎麼建造吉薩大金字塔的？我們現在對它的建造方法、勞動力、和運輸技術，實際知道什麼？

## Grounding decision (from shared initial plan, used as-is)
- Domain: Egyptology / ancient construction archaeology.
- "建造方法" = construction methods (how blocks were moved and lifted).
- "勞動力" = labor force (size, composition, social organization).
- "運輸技術" = transport technology (quarry to site, lifting).
- The question asks what we ACTUALLY know — established consensus vs. contested vs. genuinely unknown.

## Decomposition (sub-questions)
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

## Stop-condition budget (BASE skill step-6 guards)
- Max 4 cycles total.
- Diminishing returns: stop if a cycle promotes <2 claims or resolves <1 HIGH-severity dispute.
- No-rework: a claim disputed for 2 consecutive correction cycles becomes a permanent dispute.
- 400 EV cap.
- Convergence: no re-dispatch of an already-searched sub-question.

## Operational note
The Agent subagent tool is not available in this environment, so worker retrieval and verifier checking are performed inline by the orchestrator, writing evidence to the pool exactly per the schema. This weakens the skill's generator/verifier isolation guarantee; the limitation is disclosed honestly in the final report.
