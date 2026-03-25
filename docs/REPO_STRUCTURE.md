# Repository Structure

This repository is structured to balance **clarity, accessibility, and completeness**.

The public entry point remains concise, while full specifications and practical examples are preserved in dedicated directories.

---

## 📂 Root

- [`README.md`](../README.md) — Overview and entry point  
- [`CHANGELOG.md`](../CHANGELOG.md) — Version history  
- [`VERSION`](../VERSION) — Current repository version  
- [`LICENSE`](../LICENSE) — MIT license  

---

## 📁 Directories

### `spec/`

Full framework specifications and formal definitions.

- [`UCO_v1.8_Full_Instructions.md`](../spec/UCO_v1.8_Full_Instructions.md)  
- [`UCO_v1.8.1_Patch.md`](../spec/UCO_v1.8.1_Patch.md)  

👉 Complete operational logic and system behavior

---

### `examples/`

Practical implementations of UCOS across ambiguity types.

- [`example_01_asr_uncertainty_preservation.md`](../examples/example_01_asr_uncertainty_preservation.md)  
- [`example_02_translation_ambiguity.md`](../examples/example_02_translation_ambiguity.md)  
- [`example_03_llm_hallucination_control.md`](../examples/example_03_llm_hallucination_control.md)  

👉 Real-world evaluation behavior

---

### `docs/`

Repository-level supporting documents.

- [`REPO_STRUCTURE.md`](REPO_STRUCTURE.md)  

👉 Internal documentation and repository explanation

---

## 🧠 Design Principles

This repository is structured into three layers:

### 1. Entry Layer (README)

- Fast understanding  
- Conceptual overview  

---

### 2. Specification Layer (`spec/`)

- Formal definitions  
- Full architecture  
- Operational rules  

---

### 3. Execution Layer (`examples/`)

- Real-world cases  
- Ambiguity handling  
- Reproducible decisions  

---

## 🎯 Why this structure

AI frameworks often fail because:

- Concepts are not operationalized  
- Examples lack formal grounding  
- Documentation is fragmented  

UCOS separates:

👉 Concept (README)  
👉 Specification (spec)  
👉 Execution (examples)  

This ensures:

- Readability  
- Precision  
- Reproducibility  

---

## 🔍 Navigation Guide

If you're new:

→ Start with [`README.md`](../README.md)

If you want full specification:

→ Explore [`spec/`](../spec/)

If you want practical understanding:

→ Explore [`examples/`](../examples/)
