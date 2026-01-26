# AOCL Protocol — Agent Orchestration Control Layers

AOCL is a protocol for running modern agent swarms through **layered control pipelines**.

It standardizes *how an orchestrator processes an incoming event* (user message, alert, webhook, cron tick) by passing it through ordered layers such as:

- Smart routing
- Policy & safety gating
- Context/memory integration
- Rewriting/structuring
- Delegation to agents/tools
- Verification
- Response assembly

**AOCL is not a runtime.**  
It does not prescribe how to execute tools, schedule workers, or build agents. It standardizes the *control flow and the audit trail*.

---

## Relationship to AEE (Agent Envelope Exchange)

AOCL is designed to run alongside **AEE**.

- **AEE** = the minimal envelope for agent↔agent / human↔agent messages (identity + intent + causality).
- **AOCL** = the orchestrator’s layered control pipeline that decides what to do.

**Key idea:** AOCL makes layers auditable by emitting **AEE envelopes for layer activity**, not just for agent hops.

**AEE repo:** https://github.com/AdaminX/AEE-Agent-Envelope-Exchange

---

## Mental Model

Think of AOCL like an **OSI-style layer stack**, but for agent orchestration:

```
Ingress → Identity/Scope → Smart Router → Policy Gate → Context → Rewrite → Delegate → Verify → Assemble
```

AOCL supports:

- **Pipelines** (simple ordered layers)
- **Graphs** (branching and conditional paths)
- **Bypass** (skip layers under explicit control + audit)
- **Parallel** (optional: run some layers concurrently)

---

## AOCL Outputs (Audit-First)

Each layer produces:

- A decision summary (what happened and why)
- A context delta (what changed)
- Optional actions (delegate work, call tools, ask for HITL approval)

These are emitted as **AEE `event`/`stream` envelopes** using `aocl.*` intents.  
See: `docs/aee-binding.md`

---

## Quick Example (High Level)

1. A human sends an AEE `task` to the orchestrator: `ops.backup.status.check`
2. AOCL runs the request through layers:
   - L2 smart router decides whether to fast-path
   - L3 policy gate applies safety/tool constraints
   - L5 context layer retrieves memory/files
   - L7 delegate layer assigns a worker agent
3. AOCL emits AEE `event` envelopes for each layer activation/decision
4. A worker returns an AEE `result`
5. AOCL assembles the final response (and logs the trace)

---

## Documentation

- `docs/spec.md` — Core AOCL concepts and layer contract
- `docs/aee-binding.md` — How AOCL uses AEE without changing AEE
- `docs/stacks.md` — Default layer stacks + simple stack definition format
- `docs/observability.md` — Tracing and “see layers activate”
- `ROADMAP.md` — Planned extensions and future docs

---

## Status

**Experimental (v0.1)**  
AOCL is intended to stay small: stable core semantics, with extensibility through stack definitions and intent schemas.
