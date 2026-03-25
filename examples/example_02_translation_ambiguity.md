# Example 02 — Translation Ambiguity Control

## Context
Machine Translation task involving semantic ambiguity in source text.
The objective is to preserve meaning without forcing premature disambiguation.

## Input
Source sentence:

"He saw her duck."

## Problem
- Sentence contains structural ambiguity
- Multiple valid interpretations exist
- Context is insufficient for disambiguation
- Risk of committing to incorrect meaning
- Forced translation may distort original intent

## UCOS Breakdown

### OBS (Observation)
- Lexical ambiguity: "duck" (noun vs verb)
- Pronoun reference: "her" (object vs possessive)
- No contextual cues available
- Sentence is syntactically valid under multiple parses

---

### INT (Interpretation)

#### Candidate Meanings
1. He observed a duck belonging to her  
2. He saw her lower her head (duck action)  
3. He witnessed her interacting with a duck  

#### Selected Meaning
- Ambiguity preserved (no single interpretation selected)

#### Selection Rationale
- No interpretation has sufficient contextual support  
- Multiple readings remain equally plausible  
- Forced resolution would introduce interpretive bias  
- Parallel interpretation maintained  

---

### UNC (Uncertainty)

- perceptual: 0 (text is clear)
- inferential: 3 (semantic ambiguity high)
- contextual: 3 (no disambiguating context)
- policy: 0

#### Dominant Axis
inferential

#### Resolution Condition
- Additional context (preceding or following sentence)
- Domain knowledge or discourse clarification

---

### DTR (Decision Trace)

1. OBS → ambiguity detected at lexical and syntactic levels  
2. INT → multiple valid parses identified  
3. UNC → inferential uncertainty exceeds safe resolution threshold  
4. Decision Gate → disambiguation blocked (non-expansive policy)  
5. Action → preserve ambiguity explicitly  
6. Audit → no semantic distortion introduced  

---

## Output

"It is unclear whether he saw her duck (the animal) or saw her duck (the action), as there is no contextual information available."

---

## Notes

- No forced disambiguation  
- Ambiguity is explicitly surfaced rather than hidden  
- Translation preserves interpretive openness  
- Output avoids semantic commitment beyond evidence  

---

## Key Insight

This example demonstrates how UCOS enforces:

- Controlled interpretation branching (INT)
- Suppression of premature semantic resolution
- Explicit handling of ambiguity in translation
- Faithfulness to source uncertainty

---

## Extended Insight (from real QA patterns)

- "Meaning ≠ Single Answer" must be respected  
- Ambiguity is a valid state, not an error  
- Over-resolution leads to hidden hallucination  
- Stable translation requires controlled interpretation, not selection  
