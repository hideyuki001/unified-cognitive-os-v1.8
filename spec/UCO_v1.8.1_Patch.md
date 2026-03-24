# UCO v1.8.1 — Structural Patch
## Additive Defect Fix | Backward-Compatible with v1.5–v1.8

---

## Overview

This patch resolves three structural defects identified in UCO v1.8.

- No architectural changes are introduced.
- All existing records from v1.5–v1.8 remain valid.

This document must be read in conjunction with:
`UCO_v1.8_Full_Instructions.md`

In case of conflict, this patch takes precedence.

---

## Defects Addressed

- DTR was not verifiable when `trace_complete: false`
- UNC escalation logic applied a uniform threshold across all axes
- `signal_class: other:` lacked lifecycle governance

---

## Unchanged Principles

- OBS → INT → UNC → DTR ordering
- STOP > REPAIR priority
- Kernel enforces structure / UCO enforces meaning
- Red Flag / Stopline / JRO system unchanged
- JSI mechanisms (v1.6)
- Handoff Packet structure (v1.7)
- Human authority remains the final decision layer

---

# Patch 1 — DTR: `missing_gates`

## Problem

When `trace_complete: false`, incompleteness could be detected but not specified.

Result:
- Missing information was not explicit
- Handoff recipients could not resolve gaps deterministically

## Fix

Introduce `missing_gates` into the DTR schema.

### Constraint

- REQUIRED when `trace_complete: false`
- MUST be `[]` when `trace_complete: true`

### Enforcement

- If `trace_complete: false` AND `missing_gates: []`
  → Kernel rejects (`dtr_must_not:incomplete_without_missing_gates`)
- If `trace_complete: true` AND `missing_gates` is non-empty
  → Kernel rejects (contradiction)

  # Patch 2 — UNC: Axis-Specific Escalation

## Problem

A single escalation threshold was applied across all uncertainty axes.

This assumption is invalid:

- Perceptual uncertainty → often resolvable
- Policy uncertainty → requires escalation at low levels

Uniform thresholds created operational ambiguity.

## Fix

Replace scalar `escalation_trigger` with axis-specific thresholds.

### Default Thresholds

perceptual: 3
inferential: 2
contextual: 2
policy: 1


### Enforcement

For each axis:


if axes.{axis} >= escalation_trigger.{axis}
→ Stopline must be STOP or DEFER


- Each axis is evaluated independently
- Any single breach triggers escalation

---

# Patch 3 — Signal Class Governance

## Problem

`signal_class: other:` acted as an escape hatch without lifecycle control.

Result:
- Unbounded accumulation
- Gradual degradation of type safety

## Fix

Introduce **Signal Class Candidate Log (Template 6)**

### Design Principle

- Logging mechanism (non-blocking)
- Kernel emits signals
- Humans make classification decisions

### Trigger Rule

occurrence_count >= 3
→ status: under_review
→ reviewer decision required
