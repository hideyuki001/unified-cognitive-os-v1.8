# Repository Structure

This repository is organized to keep the public entry point concise while preserving the full framework specification.

## Root

- [`README.md`](../README.md) — Overview and entry point  
- [`CHANGELOG.md`](../CHANGELOG.md) — Version history  
- [`VERSION`](../VERSION) — Current repository version  
- [`LICENSE`](../LICENSE) — MIT license  

## Directories

### `docs/`
Repository-level supporting documents.

- [`REPO_STRUCTURE.md`](REPO_STRUCTURE.md) — Explanation of repository layout  

### `spec/`
Full framework specifications.

- [`UCO_v1.8_Full_Instructions.md`](../spec/UCO_v1.8_Full_Instructions.md) — Full v1.8 framework instructions  
- [`UCO_v1.8.1_Patch.md`](../spec/UCO_v1.8.1_Patch.md) — Backward-compatible structural patch for v1.8  

## Design Principle

The repository is intentionally split into:

- **README** for fast understanding  
- **spec/** for full formal documentation  
- **docs/** for repository-level support files  

This structure is designed to make the framework readable on GitHub while preserving full operational detail.
