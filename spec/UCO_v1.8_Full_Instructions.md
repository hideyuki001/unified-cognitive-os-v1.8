# Unified Cognitive OS v1.8 — Full Framework Instructions
## Translation & AI QA Edition | Judgment Decomposition Architecture

---

## VERSION LINEAGE

| Version | Core Metaphor | Tagline |
|---------|--------------|---------|
| v1.5 | Gate | Repair minimally or stop safely |
| v1.6 | Sensor | Make stopping easier by making instability visible |
| v1.7 | State Machine | Judgment is a state, not a label |
| v1.8 | Lens Stack | Judgment is a structured view, not a reflex |

**Design philosophy progression:**
- v1.5 → "Repair minimally, or stop safely."
- v1.6 → "Make stopping easier by making instability visible."
- v1.7 → "Preserve judgment as state, transport it safely, and hand it off properly."
- v1.8 → "Clarify what was actually observed before starting judgment."

---

## WHAT v1.8 ADDS / DOES NOT ADD

### ADDS
- Judgment Decomposition Layer (JDL): four sub-records — OBS / INT / UNC / DTR
- Multi-axis uncertainty vector (moving from a single score to a vector)
- signal_class taxonomy (a bridge from OBS to Red Flag detection)
- DTR gate trace (an auditable record of the judgment sequence)
- Cross-domain OBS portability (ASR observation records can be reused in translation review)

### DOES NOT ADD
- New Red Flags
- New Stopline verdict types
- New JRO operators
- Automation or autonomous AI judgment
- Quality optimization objectives

### PRESERVES (unchanged from v1.5)
- STOP > REPAIR priority
- Non-expansion guarantee
- One segment, one JRO
- Post-Repair Audit as a gate
- Validity of NONE
- Final human authority
- Decision ID format: `DEC-{YYYYMMDD}-{SEG_ID}-{SEQ}`

---

## LAYER STACK

```text
┌────────────────────────────────────────────────────────────────┐
│              HUMAN JUDGMENT  (Final Authority)                │
├────────────────────────────────────────────────────────────────┤
│  v1.7  HANDOFF LAYER      Escalation Packet · Caveat Form     │
│  v1.7  CONTEXT LAYER      Snapshot · Provenance · Variance    │
├────────────────────────────────────────────────────────────────┤
│  v1.8  DECOMPOSITION LAYER  OBS · INT · UNC · DTR  ◄ NEW      │
├────────────────────────────────────────────────────────────────┤
│  v1.6  STABILITY LAYER    JSI · Oscillation · Staleness       │
├────────────────────────────────────────────────────────────────┤
│  v1.5  DECISION LAYER     Red Flag → Stopline → JRO → Audit   │
└────────────────────────────────────────────────────────────────┘
```

The JDL sits above the v1.5 decision mechanism and below the v1.7 context and handoff layers. It does not replace any existing component. It provides typed pre-records that make upstream judgment traceable and downstream artifacts reusable.

---

## FULL OPERATIONAL WORKFLOW — v1.8

```text
PHASE 0   ── Assign Decision ID
          └── DEC-{YYYYMMDD}-{SEG}-{SEQ}

PHASE 0a  ── [NEW v1.8] Record OBS (Observation)
          └── Confirm signal_class before proceeding to Red Flag detection

PHASE 0b  ── [NEW v1.8] Record INT (Interpretation)
          └── Enumerate candidate meanings → select one → document rationale (obs_ref required)

PHASE 0c  ── [NEW v1.8] Record UNC (Uncertainty Vector)
          └── Score four axes → determine dominant_axis → check escalation_trigger
          └── If any axis is at or above escalation_trigger → STOP or DEFER is mandatory in PHASE 2

PHASE 1   ── Red Flag Detection       [v1.5 + v1.6 hesitation logging]
          └── Map OBS.signal_class to a Red Flag category
          └── If torn between two flags, use dual-flag notation

PHASE 2   ── Stopline Decision        [v1.5]
          │   STOP  ──► PHASE 2a + generate Handoff Packet  [v1.7]
          │   DEFER ──► PHASE 2a + generate Handoff Packet  [v1.7]
          └── ALLOW ──► PHASE 2a → PHASE 3
          └── stopline.reason must cite INT and/or UNC references

PHASE 2a  ── Generate Context Snapshot    [v1.7]
          └── The uncertainty field may be replaced by the UNC record (UNC is canonical)

PHASE 3   ── JRO Selection            [v1.5]
          └── If torn between JROs, record NONE(uncertainty) [v1.6]
          └── Add this gate step to DTR.trace

PHASE 4   ── Post-Repair Audit        [v1.5 + doubt_persists v1.6]
          └── Add this gate step to DTR.trace
          └── If doubt_persists=true, generate a Delivery Caveat [v1.7]

PHASE 5   ── Propagation Check        [v1.7]
          └── Select propagation_type based on DTR.terminal_state

PHASE 5a  ── [NEW v1.8] Complete DTR
          └── Finalize trace_complete: true/false
          └── If false, the Kernel raises a flag → UCO responds (escalation or caveat)

PHASE 6   ── Final Decision + Delivery
```

**Decision Priority (unchanged):**
STOP > DEFER > ALLOW (with caveat) > ALLOW (clean)

---

## JUDGMENT DECOMPOSITION LAYER — v1.8

### Principles

Write OBS before INT. Write INT before UNC. Write UNC before the Stopline decision. Complete DTR only after reaching a terminal state. No skipping. No retroactive filling after the outcome is known.

**Mandatory rule:** If `stopline.reason` cites an INT or UNC reference that does not exist, the record is invalid.

---

### OBS — Observation Record

**Question: What was perceived, where, and with what level of perceptual confidence?**

```yaml
obs:
  obs_id:           # OBS-{decision_id}
  domain:           # asr / translation / ai_eval / agentic
  signal_class:     # see signal_class taxonomy below
  location_pointer: # segment_id + timestamp or span
  raw_evidence:     # verbatim excerpt or direct description (no interpretation)
  confidence:       # 0.0 – 1.0 (perceptual confidence only)
  perceptual_note:  # what made detection harder or easier
```

**Constraint:** `raw_evidence` must not contain interpretation. Statements such as "it sounded like ..." belong in INT.

---

### INT — Interpretation Record

**Question: What does the observation mean in context?**

```yaml
int:
  int_id:           # INT-{decision_id}
  obs_ref:          # link to OBS record (required)
  candidate_meanings:
    - label:        # short name for this reading
      rationale:    # why the evidence supports this reading
      strength:     # weak / moderate / strong
  selected_meaning:
    label:
    selection_rationale:   # must cite evidence; "sounds natural" is invalid
    weaker_interpretation_applied: true / false
  interpretation_conflict: true / false
  conflict_note:    # if true, describe the conflict
```

**Disallowed rationale strings (Kernel-detectable):**
- "sounds better"
- "more natural"
- "probably means" (without cited evidence)
- "client expects"

---

### UNC — Uncertainty Record

**Question: On which axis is the uncertainty located, how strong is it, and what would reduce it?**

```yaml
unc:
  unc_id:           # UNC-{decision_id}
  int_ref:          # link to INT record (required)
  axes:
    perceptual:     # 0–3 (can the audio/text be perceived correctly?)
    inferential:    # 0–3 (is the interpretation justified?)
    contextual:     # 0–3 (is there enough context?)
    policy:         # 0–3 (do the rules cover this case?)
  dominant_axis:    # axis driving the uncertainty most strongly
  resolution_condition: # what information would reduce uncertainty
  escalation_trigger:   # uncertainty level requiring STOP/DEFER (default: 2)
  jsi_hooks:            # link to v1.6 JSI fields (hesitation, doubt_persists, etc.)
```

**Score definitions:**
- 0 = no uncertainty
- 1 = minor (does not affect the decision)
- 2 = moderate (requires explicit logging)
- 3 = decision-blocking (STOP or DEFER required)

**Kernel enforcement rule:** If any axis is at or above `escalation_trigger`, the Stopline verdict must be either STOP or DEFER.

---

### DTR — Decision Trace Record

**Question: Which gates were traversed, and what determined the terminal state?**

```yaml
dtr:
  dtr_id:           # DTR-{decision_id}
  unc_ref:          # link to UNC record (required)
  trace:
    - step: 1
      gate:   # red_flag / stopline / jro_select / audit / propagation
      input:  # information entering this gate
      output: # decision or signal leaving this gate
      note:   # optional: hesitation or alternatives considered
  terminal_state:   # STOP / DEFER / ALLOW / ALLOW_WITH_CAVEAT
  trace_complete:   # true = all gates recorded / false = shortcut present
```

**Handling `trace_complete: false`:** The Kernel raises a flag. UCO determines the response (escalation or Delivery Caveat).

---

## SIGNAL CLASS TAXONOMY — v1.8

`signal_class` is the bridge between OBS and the Red Flag system. It provides a cross-domain typed representation.

### Domain-specific signal_class list

#### ASR Domain

| signal_class | Description | Primary Red Flag |
|---|---|---|
| `overlap_boundary` | Start/end boundary of overlapping speech is unclear | RF-004 |
| `overlap_false_positive` | overlap tag is present, but there is no actual overlap | RF-001 |
| `backchannel_overlap` | backchannel/filler was recognized as overlap | RF-002 |
| `nonverbal_overlap` | nonverbal audio such as laughs/sighs is mixed into overlap | RF-006 |
| `unintelligible_low_info` | unintelligible content with low semantic contribution (candidate for deletion) | RF-011 |
| `unintelligible_partial` | only part is intelligible (core-token recovery may be possible) | RF-008 |
| `unintelligible_ghost` | unintelligible tag may be effectively empty/no real content | RF-001 |
| `trailing_uncertainty` | speech tail is unclear | RF-008 |
| `prefix_uncertainty` | speech onset is unclear | RF-008 |
| `speaker_boundary_violation` | speech is misattributed across segment boundaries | RF-014 |
| `hallucination_risk` | ASR may have generated unsupported lexical content | RF-001 |
| `phrase_recovery_candidate` | recoverable phrase may exist despite noise | RF-008 |
| `proper_noun_uncertainty` | unclear whether a proper noun is full form or abbreviated form | RF-008 |
| `low_confidence_token` | confidence is low for an individual token | RF-008 |

#### Translation Domain

| signal_class | Description | Primary Red Flag |
|---|---|---|
| `certainty_shift` | certainty is stronger or weaker than in the source (e.g. may → will) | MFLD (severity 2–3) |
| `scope_shift` | scope or applicability has broadened or narrowed | MFLD (severity 2–3) |
| `agency_shift` | actor or responsibility has shifted | RESP |
| `passive_active_shift` | passive↔active conversion affects responsibility attribution | RESP |
| `lexical_ambiguity` | local lexical branching in interpretation | AMB-1 |
| `structural_ambiguity` | sentence-level structural branching in interpretation | AMB-2 |
| `propagating_ambiguity` | ambiguity propagates across paragraph or document boundaries | AMB-3 |
| `proxy_judgment` | judgment replaced by surface-level signals such as "sounds natural" | PROXY |
| `terminology_inconsistency` | mismatch with approved terminology | MFLD (severity 1) |
| `attribution_without_evidence` | attribution added without source evidence | RESP |
| `clarification_expansion` | clarification materially expands scope | AMB-2 / MFLD-2 |

#### AI Evaluation Domain

| signal_class | Description | Primary Red Flag |
|---|---|---|
| `instruction_misalignment` | model output does not align with task instructions | PROXY |
| `confidence_overstatement` | model expresses high confidence without support | MFLD-2 |
| `hallucination_content` | content lacks factual support | RF-001 equivalent |
| `scope_overreach` | output extends beyond the requested scope | MFLD-3 |
| `refusal_appropriateness` | appropriateness of refusal is unclear | AMB-2 |
| `format_violation` | output violates required format | PROXY |
| `partial_completion` | task is only partially completed | AMB-1 |

#### Agentic Domain

| signal_class | Description | Primary Red Flag |
|---|---|---|
| `action_boundary_violation` | agent attempted an action outside the allowed boundary | RESP / MFLD-3 |
| `tool_call_uncertainty` | tool result is uncertain | AMB-2 |
| `state_ambiguity` | system state has multiple plausible interpretations | AMB-2 |
| `delegation_scope_unclear` | delegated task scope is unclear | AMB-3 |
| `irreversible_action_risk` | irreversible action may occur | RESP |

### signal_class selection rules

1. **Determine the domain first** (write `obs.domain` before anything else)
2. **If multiple signal_class values appear applicable, choose the weaker/more local one**
3. **If none applies, record `other:{brief_description}` and flag it as a candidate for taxonomy expansion**
4. **Only proceed to Red Flag detection (PHASE 1) after signal_class is fixed**

---

## RED FLAG SYSTEM (Inherited from v1.5)

### Categories

**AMB — Ambiguity Risk**  
Core question: Has the output become open to more reasonable interpretations than the source?
- AMB-1: local ambiguity
- AMB-2: sentence-level interpretive branching
- AMB-3: ambiguity that propagates beyond the sentence

**MFLD — Meaning / Force Drift**  
Core question: Has certainty, scope, or force shifted relative to the source?
Typical cases: may→will, specific subject→general subject, reportive→assertive tone

**RESP — Responsibility / Agency Shift**  
Core question: Has agency, responsibility, or attribution changed or become unclear?
Typical cases: actor disappears, actor appears, passive/active shift alters responsibility

**PROXY — Proxy Judgment Substitution**  
Core question: Has substantive judgment been replaced by surface-level signals or assumptions?
Typical case: "sounds natural" used as justification

### Severity

| Level | Range |
|--------|------|
| -1 | Local; unlikely to propagate |
| -2 | Affects sentence-level interpretation or decision boundary |
| -3 | Changes document-level meaning or downstream judgment |

If in doubt, choose the lower severity and record the hesitation in the note.

### Red Flag YAML

```yaml
red_flags:
  - type: AMB-2
    note: "Reason for the flag"
    hesitation_logged: false      # true if torn between two flags [v1.6]
    dual_flag_candidate: MFLD-2   # fill only if hesitation=true [v1.6]
    obs_ref:                      # [v1.8] linked OBS record
```

A Red Flag does not instruct repair. It only identifies the risk type. If no flag applies, leave the field empty. An empty field is still a valid judgment.

---

## STOPLINE SYSTEM (Inherited from v1.5)

### Meaning of each verdict

**STOP — Intervention prohibited**
- An irreversible risk exists
- The case exceeds reviewer authority
- Only repairs that change responsibility or scope are available
- → v1.7: generate a Handoff Packet

**DEFER — Judgment temporarily suspended**
- Context or information is insufficient
- Red Flags conflict
- Decision authority is unclear
- → v1.7: generate a Handoff Packet

**ALLOW — Limited intervention permitted**
- The Red Flag is understood
- Repair scope is minimal
- No responsibility or scope expansion is required
- → v1.7: a Context Snapshot must still be generated

### Stopline YAML

```yaml
stopline:
  verdict: STOP / DEFER / ALLOW
  reason:                         # required; must cite INT/UNC refs and describe risk, not preference
  int_ref:                        # [v1.8] cited INT record
  unc_ref:                        # [v1.8] cited UNC record
  timestamp:
  context_last_updated:
  staleness_threshold: 7_days     # [v1.6]
```

**Good example of `reason`:**
```text
reason: "INT-DEC-20260322-S042-001 contains two valid interpretations.
UNC-DEC-20260322-S042-001 has inferential=3, which blocks safe judgment, so STOP."
```

**Bad example of `reason`:**
```text
reason: "sounds better / more natural / client probably expects this"
```

---

## JRO SYSTEM (Inherited from v1.5, extended by v1.6)

| JRO | Name | Allowed | Prohibited |
|-----|------|---------|------------|
| JRO-1 | Clarification Without Expansion | Resolve local ambiguity already implied by the source | Adding certainty, new modifiers, or scope changes |
| JRO-2 | Scope Containment | Restore the original limitation or re-anchor a modifier | Generalization or adding examples |
| JRO-3 | Responsibility Neutralization | Rebalance attribution to match the source | Assigning responsibility or clarifying beyond the source |
| JRO-4 | Terminology Alignment | Restore consistent terminology or established equivalents | Redefining concepts or audience adaptation |
| NONE | No Repair Applied | Explicitly record that no safe repair exists | — |

**Usage rules (non-negotiable):**
- Maximum one JRO per repair
- If no JRO fits safely, choose NONE
- JRO may be selected only after an ALLOW verdict
- The selection rationale must be explainable in one sentence
- If torn between two JROs, neither is safely justified → choose NONE (uncertainty)

```yaml
repair_operation:
  jro_used: JRO-1 / NONE
  uncertainty_flag: false         # true if torn between JROs [v1.6]
  reason:
  before:
  after:
```

---

## POST-REPAIR AUDIT (Inherited from v1.5, extended by v1.6)

This is not quality review. It is a damage check.

```yaml
post_repair_audit:
  confidence_increased: false     # did the repair make the statement sound more certain?
  responsibility_shifted: false   # did agency, responsibility, or attribution change?
  scope_expanded: false           # did scope, applicability, or generalization expand?
  intent_altered: false           # did communicative intent shift?
  new_stopline_triggered: false   # did the repaired output create a new Stopline concern?
  audit_passed: false
  doubt_persists: false           # does doubt remain even after passing the audit? [v1.6]
  doubt_note:
  auditor_note:
```

All fields are binary (`true`/`false`). If any field is `true`, the repair fails. That is a valid detection outcome, not an operational mistake. If `doubt_persists: true` and delivery is still approved, generate a Delivery Caveat. Audit pass ≠ delivery obligation.

---

## STABILITY LAYER — JSI (Inherited from v1.6)

```yaml
jsi:
  hesitation_present: true / false
  decision_conflict_present: true / false
  doubt_persistence_present: true / false
  weaker_interpretation_applied: true / false
  revisit_needed: true / false

  instability_sources:
    - overlap_boundary
    - lexical_uncertainty
    # etc.

  resolution_pattern:
    - conservative_selection
    - structure_preservation

  interpretation: |
    Describe convergence or unresolved instability in 1–2 sentences.
```

**JSI decision rules:**
- Logged hesitation weakens intervention; it does not justify intervention
- `hesitation_present = true` + hesitation between JROs → `NONE(uncertainty)`
- `doubt_persistence_present = true` + audit pass → Delivery Caveat

---

## CONTEXT TRACKING LAYER (Inherited from v1.7)

```yaml
context_snapshot:
  context_id:          # CTX-{date}-{seg}-{seq}
  decision_id:
  created_at:
  reviewer:

  target:
    artifact_type:     # segment / doc / batch
    location_pointer:
    version_hash:

  task_intent:
    user_intent_summary:
    task_goal:
    success_definition:

  risk_posture:
    risk_appetite:     # strict / balanced / permissive
    harm_profile:      # privacy / safety / legal / reputation / financial
    must_not_do:
    acceptable_tradeoff:

  preconditions:
    assumed_context:
    external_dependencies:
    assumed_audience:

  rules_applied:
    guideline_refs:    # [ rule_id_1, rule_id_2 ]
    exceptions_invoked:

  uncertainty:
    # [v1.8] UNC is canonical. The following fields remain for backward compatibility.
    level:             # 0 (none) to 3 (decision-blocking) ← may be replaced by UNC dominant score
    drivers:           # ← may be replaced by UNC.dominant_axis
    what_would_reduce_it:  # ← may be replaced by UNC.resolution_condition
    unc_ref:           # [v1.8] recommended: UNC-{decision_id}

  propagation_hooks:
    propagation_type:  # NONE / CONSTRAINS / CONTAMINATES / DEPENDS_ON
    target_decision_id:
    constraint_note:
```

---

## HANDOFF LAYER (Inherited from v1.7)

### Escalation Handoff Packet

```yaml
handoff_packet:
  meta:
    packet_id:         # HO-{date}-{seg}-{seq}
    created_at:
    decision_id:
    trigger:           # STOP / DEFER
    urgency:           # immediate / standard / low

  context_ref:
    context_id:
    decision_id:
    dtr_ref:           # [v1.8] add DTR-{decision_id}

  item:
    description:
    location_pointer:
    primary_flags:
    current_verdict:   # STOP / DEFER
    reason:

  authority_request:
    requesting:        # what decision is needed
    authority_required:
    named_owner:       # specific person or role

  options:
    option_A:
      description:
      pros:
      cons:
      risks:
    option_B:
      description:
      pros:
      cons:
      risks:

  evidence_pointers:
    - obs_ref:         # [v1.8] link to OBS record
    - int_ref:         # [v1.8] link to INT record

  interim_action:
    handling:          # STOP / DEFER / ALLOW_WITH_CAVEAT
    downstream_constraints:

  resolution_log:
    receiver_decision:
    rationale:
    closed_at:
```

### Resume Conditions

A stopped or deferred judgment may resume only if all of the following are true:
- A human decision owner is explicitly identified by name and role
- The reasoning can be explained to a third party
- Required review or validation steps have been explicitly restored
- Not resuming remains a valid option

If even one condition is unmet, resumption is prohibited. Issue a new Handoff Packet instead.

---

## MASTER TEMPLATE — UCO v1.8 Complete Workflow

```yaml
# ═══════════════════════════════════════════════════════════════
# UCO v1.8 — Unified Cognitive OS
# Translation & AI QA Edition  |  Judgment Decomposition Architecture
# ═══════════════════════════════════════════════════════════════

decision_id:         # DEC-{YYYYMMDD}-{SEG_ID}-{SEQ}
created_at:
reviewer:

# ─── PHASE 0a: Observation [v1.8] ────────────────────────────
obs:
  obs_id:            # OBS-{decision_id}
  domain:            # asr / translation / ai_eval / agentic
  signal_class:      # choose from signal_class taxonomy
  location_pointer:
  raw_evidence:      # verbatim excerpt only (no interpretation)
  confidence:        # 0.0 – 1.0
  perceptual_note:

# ─── PHASE 0b: Interpretation [v1.8] ─────────────────────────
int:
  int_id:            # INT-{decision_id}
  obs_ref:           # obs_id above
  candidate_meanings:
    - label:
      rationale:
      strength:      # weak / moderate / strong
  selected_meaning:
    label:
    selection_rationale:   # must cite evidence
    weaker_interpretation_applied: false
  interpretation_conflict: false
  conflict_note:

# ─── PHASE 0c: Uncertainty [v1.8] ────────────────────────────
unc:
  unc_id:            # UNC-{decision_id}
  int_ref:           # int_id above
  axes:
    perceptual:      # 0–3
    inferential:     # 0–3
    contextual:      # 0–3
    policy:          # 0–3
  dominant_axis:
  resolution_condition:
  escalation_trigger: 2
  jsi_hooks:

# ─── PHASE 1: Red Flag Detection ──────────────────────────────
red_flags:
  - type: AMB-?      # AMB / PROXY / MFLD / RESP + severity 1-3
    note:
    hesitation_logged: false
    dual_flag_candidate:   # fill only if hesitation=true
    obs_ref:               # [v1.8]

# ─── PHASE 2: Stopline Decision ───────────────────────────────
stopline:
  verdict: STOP / DEFER / ALLOW
  reason:            # must cite INT/UNC refs
  int_ref:           # [v1.8]
  unc_ref:           # [v1.8]
  timestamp:
  context_last_updated:
  staleness_threshold: 7_days

# STOP/DEFER → generate Handoff Packet (see template below)
# handoff_packet_id:   # HO-{date}-{seg}-{seq}

# ─── PHASE 2a: Context Snapshot [v1.7] ────────────────────────
context_snapshot:
  context_id:        # CTX-{date}-{seg}-{seq}
  task_intent:
  risk_posture:      # strict / balanced / permissive
  harm_profile:
  preconditions:
  rules_applied:
  uncertainty:
    unc_ref:         # [v1.8] UNC is canonical
    level:           # backward compatibility
    drivers:         # backward compatibility
    what_would_reduce_it:  # backward compatibility
  propagation_type:  # NONE / CONSTRAINS / CONTAMINATES / DEPENDS_ON
  propagation_target:
  propagation_note:

# ─── PHASE 3: Repair Operation (ALLOW only) ──────────────────
repair_operation:
  jro_used: JRO-? / NONE
  uncertainty_flag: false
  reason:
  before:
  after:

# ─── PHASE 4: Post-Repair Audit (repair only) ────────────────
post_repair_audit:
  confidence_increased: false
  responsibility_shifted: false
  scope_expanded: false
  intent_altered: false
  new_stopline_triggered: false
  audit_passed: false
  doubt_persists: false
  doubt_note:
  auditor_note:

# doubt_persists=true + delivery approved → generate Delivery Caveat
# caveat_id:           # CAV-{date}-{seg}-{seq}

# ─── PHASE 5: Propagation Check [v1.7] ────────────────────────
propagation_check:
  propagation_emitted: false
  type:              # CONSTRAINS / CONTAMINATES / DEPENDS_ON
  target_decision_id:
  note:

# ─── PHASE 5a: DTR Completion [v1.8] ──────────────────────────
dtr:
  dtr_id:            # DTR-{decision_id}
  unc_ref:           # unc_id above
  trace:
    - step: 1
      gate:          # red_flag / stopline / jro_select / audit / propagation
      input:
      output:
      note:
  terminal_state:    # STOP / DEFER / ALLOW / ALLOW_WITH_CAVEAT
  trace_complete:    # true / false

# ─── PHASE 6: Final Decision ──────────────────────────────────
final_decision:
  deliverable: YES / NO
  escalation_required: YES / NO
  caveat_attached: YES / NO
  note:
```

---

## UCO vs. KERNEL — ROLE BOUNDARY (v1.8)

| Responsibility | Owner in v1.8 | Rationale |
|------|----------------|------|
| OBS schema validation (`signal_class` in allowed set, `confidence` within [0,1]) | Kernel | Deterministic type check; no judgment required |
| INT candidate meaning generation | UCO protocol | Requires domain knowledge; Kernel cannot enumerate meanings |
| INT selection enforcement (reject "sounds better" style reasoning) | Kernel | Pattern match against prohibited rationale strings |
| UNC axis scoring (0–3 on each axis) | Reviewer via UCO form | Requires calibrated human judgment; Kernel only checks range validity |
| UNC escalation-trigger enforcement | Kernel | Deterministic threshold logic: any axis ≥ trigger → STOP/DEFER required |
| DTR trace completeness check | Kernel | Deterministic: all required gates must be present in the trace |
| DTR shortcut detection (`trace_complete: false`) | Kernel → flag, UCO → response | Kernel detects; UCO decides the response |
| Cross-domain OBS reuse (portability) | UCO protocol | Domain mapping rules belong to UCO, not Kernel logic |

**Boundary principle:**
- **The Kernel enforces structure; UCO enforces meaning.**
- Any check executable without understanding semantic content belongs to the Kernel.
- Any decision requiring knowledge of what an interpretation means in context belongs to UCO.
- **The Kernel does not read `INT.selected_meaning`.** It reads `OBS.signal_class`, `UNC.axes[*]`, and `DTR.trace_complete`. Semantic content remains fully inside UCO territory.

---

## STABILITY PRINCIPLES (Inherited from v1.6, enforced in v1.8)

| Principle | Meaning | Decision rule |
|------|------|-----------|
| Hesitation is information | If torn between flags, both risks matter. If torn between JROs, neither repair is safely justified | Hesitation weakens intervention; it never justifies stronger intervention |
| Prefer the weaker interpretation | Choosing AMB-2 over MFLD-3 preserves ambiguity; choosing DEFER preserves decision space | If interpretations differ in strength, choose the weaker one |
| "Sounds good" is a red signal | Fluency-based repair without structural justification is a PROXY failure | If fluency is the main rationale, the verdict is STOP |
| Scope expansion starts early | Asking "is it X or Y?" already narrows interpretive space when both remain viable | If a forced choice is required between equally viable interpretations, STOP |
| Responsibility visibility > superficial accuracy | Delivering "actor unclear" is safer than inferring an unsupported actor | Never make the actor more explicit in repair than the source allows |
| Audit pass ≠ proof of safety | A repair can pass all checks and still remain unstable | Audit pass is necessary, not sufficient |
| OBS before INT | Confusing observation with interpretation causes downstream failure | [v1.8] INT without prior OBS is invalid |
| UNC is not a single score | Different axes require different handling | [v1.8] identify `dominant_axis` before deciding the Stopline verdict |

---

## WHEN TO GENERATE EACH ARTIFACT (Quick Reference)

| Trigger condition | Artifact to generate |
|------------|------------|
| Any Stopline verdict | Context Record Sheet (including OBS/INT/UNC refs) |
| STOP or DEFER | Escalation Handoff Packet (including OBS/INT refs) |
| oscillation_count ≥ 2 | Oscillation Case Log |
| New composite flag pattern observed | Flag Overlap Library entry |
| doubt_persists=true + delivery approved | Delivery Caveat |
| trace_complete: false | Kernel flag → UCO decides response |
| Any UNC axis ≥ escalation_trigger | STOP or DEFER required |

---

## CLOSING NOTE

```text
v1.5:  Repair minimally or stop safely.
v1.6:  Make stopping easier by making instability visible.
v1.7:  Stop safely and hand the judgment off intact.
v1.8:  Know what you actually observed before you decide anything.

Judgment is not one act. It is four acts.

Observation is what was perceived. Interpretation is what was inferred.
Uncertainty is how much that inference can be trusted.
Decision is what was done about it.

Once those are collapsed into one step, judgment becomes hard to debug,
hard to reproduce, and unsafe to delegate—even when it sounds confident.
```

---

**Status: v1.8 DESIGN COMPLETE**  
**Integration Confidence: 93%**  
**Philosophy Preserved: restraint-first, non-expansion, human authority**  
**Next Implementation Priority: signal_class Kernel validation (OBS schema enforcement)**
