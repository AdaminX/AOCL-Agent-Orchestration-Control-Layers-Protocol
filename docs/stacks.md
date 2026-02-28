# AOCL Stacks (v0.1)

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC 2119](https://www.rfc-editor.org/rfc/rfc2119).

This document defines:

1. A RECOMMENDED default AOCL layer taxonomy, and
2. A minimal stack definition format

AOCL stacks are intentionally simple in v0.1: they MAY be a **pipeline** (ordered list) or a **DAG** (nodes + edges).

---

## 1. Recommended Layer Taxonomy

AOCL does not require these names, but adopting a shared set makes stacks portable and easier to debug.

### Canonical Layers (Recommended)

- **L0.ingress.normalize**  
  Normalize incoming events (chat/webhook/email/alert), create run IDs, basic parsing.

- **L1.identity.scope**  
  Apply identity, permissions, secret scope, and redaction rules.

- **L2.route.smart**  
  Deterministic fast-path router (pattern match, cached answers, known commands).

- **L3.policy.gate**  
  Safety/compliance checks, tool/model restrictions, HITL requirements.

- **L4.plan.decompose**  
  Convert intent into structured objectives, decide if multi-step/multi-agent.

- **L5.context.retrieve**  
  Retrieve memory/files/RAG context; produce refs + bounded summaries.

- **L6.shape.rewrite**  
  Rewrite/structure into operational form (e.g., AEE tasks, tool plans, schemas).

- **L7.delegate.execute**  
  Delegate to agents/tools, manage dependencies/concurrency (runtime-specific).

- **L8.verify.check**  
  Verification/evals/consistency checks; evidence requirements.

- **L9.assemble.respond**  
  Assemble final response; redact; apply tone/formatting constraints.

- **L10.audit.writeback**  
  Persist trace summaries and permitted memory writeback.

---

## 2. Minimal Stack Definition Format

### 2.1 Pipeline Format (Simplest)

```json
{
  "stack_id": "default",
  "version": "0.1",
  "mode": "pipeline",
  "layers": [
    {"id": "L0.ingress.normalize", "ref": "builtin:l0.normalize", "enabled": true},
    {"id": "L1.identity.scope",    "ref": "builtin:l1.identity",  "enabled": true},
    {"id": "L2.route.smart",       "ref": "builtin:l2.router",    "enabled": true},
    {"id": "L3.policy.gate",       "ref": "builtin:l3.policy",    "enabled": true},
    {"id": "L4.plan.decompose",    "ref": "builtin:l4.plan",      "enabled": true},
    {"id": "L5.context.retrieve",  "ref": "builtin:l5.context",   "enabled": true},
    {"id": "L6.shape.rewrite",     "ref": "builtin:l6.shape",     "enabled": true},
    {"id": "L7.delegate.execute",  "ref": "builtin:l7.delegate",  "enabled": true},
    {"id": "L8.verify.check",      "ref": "builtin:l8.verify",    "enabled": true},
    {"id": "L9.assemble.respond",  "ref": "builtin:l9.respond",   "enabled": true},
    {"id": "L10.audit.writeback",  "ref": "builtin:l10.audit",    "enabled": true}
  ],
  "defaults": {
    "bypass_allowed_for_roles": ["admin"],
    "max_parallel_actions": 8,
    "timeout_ms": 60000
  }
}
```

**Notes:**

- `ref` is an implementation pointer (plugin ID, module path, container image, etc.)
- AOCL spec does not define `ref` resolution; it only standardizes stack structure + layer outputs

---

## 3. Branching Stacks (DAG Format)

Use `mode: "dag"` when you want conditional routes (alerts fast-path, restricted mode, deep-research).

```json
{
  "stack_id": "default-dag",
  "version": "0.1",
  "mode": "dag",
  "nodes": [
    {"id": "L0.ingress.normalize", "ref": "builtin:l0.normalize"},
    {"id": "L1.identity.scope",    "ref": "builtin:l1.identity"},
    {"id": "L2.route.smart",       "ref": "builtin:l2.router"},
    {"id": "L3.policy.gate",       "ref": "builtin:l3.policy"},
    {"id": "L5.context.retrieve",  "ref": "builtin:l5.context"},
    {"id": "L7.delegate.execute",  "ref": "builtin:l7.delegate"},
    {"id": "L9.assemble.respond",  "ref": "builtin:l9.respond"},
    {"id": "L10.audit.writeback",  "ref": "builtin:l10.audit"},

    {"id": "BR.realtime_alert",    "ref": "builtin:branch.alert_fast"},
    {"id": "BR.restricted_mode",   "ref": "builtin:branch.restricted"}
  ],
  "edges": [
    {"from": "L0.ingress.normalize", "to": "L1.identity.scope"},
    {"from": "L1.identity.scope",    "to": "L2.route.smart"},

    {"from": "L2.route.smart",       "to": "L9.assemble.respond", "when": "control.halt_pipeline == true"},
    {"from": "L2.route.smart",       "to": "L3.policy.gate",      "when": "control.halt_pipeline != true"},

    {"from": "L3.policy.gate",       "to": "BR.restricted_mode",  "when": "control.require_hitl == true || context.C5.Execution.mode == 'restricted'"},
    {"from": "L3.policy.gate",       "to": "L5.context.retrieve", "when": "control.require_hitl != true && context.C5.Execution.mode != 'restricted'"},

    {"from": "L5.context.retrieve",  "to": "L7.delegate.execute"},
    {"from": "L7.delegate.execute",  "to": "L9.assemble.respond"},
    {"from": "L9.assemble.respond",  "to": "L10.audit.writeback"}
  ],
  "defaults": {
    "max_parallel_actions": 8,
    "timeout_ms": 60000
  }
}
```

Notes:

* `when` is intentionally expressed as a string condition (implementation-defined). In v0.1, you can treat it as simple boolean expressions over `context` and `control_flags`.
* Branch nodes (`BR.*`) can themselves represent sub-stacks.

---

## 4. Example Variant Stacks (Recommended)

### 4.1 `realtime-alert` (Speed-First)

Typical flow:

- Normalize → identity → policy → delegate → respond → audit
- Skips heavy planning/context unless necessary

### 4.2 `deep-research` (Evidence-First)

Adds:

- Stronger context retrieval (multi-source)
- Verification/evals always-on
- Tighter evidence requirements

### 4.3 `restricted-textonly` (Safety-First)

Disables tool execution:

* policy gate forces text-only mode
* delegation becomes “analysis-only” or “ask user/HITL”

---

## 5. Bypass Rules (Stack-Level)

Stacks SHOULD declare bypass policy in config, even if enforcement is runtime-specific.

**Example:**

```json
{
  "bypass_policy": {
    "allowed_roles": ["admin"],
    "never_bypass": ["L1.identity.scope", "L3.policy.gate"],
    "audit_required": true
  }
}
```

**Recommendation:**

- Implementations SHOULD NOT allow bypass of identity + policy layers in v0.1
- If bypass is allowed, the orchestrator MUST emit `aocl.control.bypass` (see `docs/aee-binding.md`)

---

## 6. What Is Intentionally Omitted (v0.1)

- A formal expression language for `when`
- A standardized plugin registry for `ref`
- A standardized signature format for stack files

These belong in the roadmap once real implementations exist.
