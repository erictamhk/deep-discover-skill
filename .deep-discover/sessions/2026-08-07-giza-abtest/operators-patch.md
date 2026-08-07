# Generative Operators Patch (EXPERIMENTAL TREATMENT ONLY)

Apply this IN ADDITION to the base deep-discover skill. It REPLACES the assess logic (step 5) and the stop condition (step 6). The control run does NOT use this.

## Replace step 5 (Assess) with generative-operator assessment

After the verify step, before synthesis, apply ALL of the following operators to the current evidence pool. Each operator reads the evidence and produces NEW sub-questions. List every new sub-question explicitly in `assessments/round-N.md`, tagged with which operator produced it.

1. **Negation operator**: For every verified claim C, ask "What evidence would falsify C?" and "What is the strongest case against C?" → produce counter-search sub-questions.
2. **Pairwise conflict operator**: For every pair (EV-i, EV-j), ask "Do they conflict?" → produce resolution sub-questions. (Combinatorial — grows with pool.)
3. **Implication operator**: For every claim C, ask "What does C imply that we haven't verified?" → produce downstream sub-questions.
4. **Provenance operator**: For every source, ask "Who produced this, with what incentive, and what would they distort?" → produce meta-questions about source reliability.
5. **Absence operator**: For every sub-question, ask "What evidence would decisively answer this, and is it in the pool?" → produce gap-filling sub-questions.
6. **Boundary operator**: For every claim, ask "Under what conditions is this true/false?" → produce scope/condition sub-questions.
7. **Analogy operator**: "In which closest domain is this pattern known to differ?" → produce cross-domain sub-questions.

## Replace the stop condition with closure saturation

You stop ONLY when applying all operators to the current pool produces NO genuinely new sub-questions (the set of operator-produced questions equals the set already asked). This is the "generative closure is saturated" condition.

- A failed search is NOT a stop signal. It is evidence of absence: write it as an EV of type `claim` ("Searched X for Y, found nothing, effort Z"), then change strategy (different source type, domain, phrasing) and probe again.
- A claim that becomes a permanent dispute is NOT a dead end. Spawn the meta-question "why do the sources disagree?" (provenance, methodology, temporal context) and probe it at least once.
- Keep the base skill's hard safety ceilings (max 4 cycles, 400 EV cap) as absolute limits, but do NOT stop early on diminishing returns of claim-promotion. Instead measure diminishing returns on NEW SUB-QUESTIONS produced: if a cycle produces 0 new sub-questions from all operators, stop.
