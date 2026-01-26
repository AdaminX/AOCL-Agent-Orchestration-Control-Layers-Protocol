# ROADMAP — AOCL Protocol

AOCL (Agent Orchestration Control Layers) is intentionally small in v0.1.
This roadmap lists sensible additions once the core concepts prove out in real implementations.

---

## Status

- Current: **v0.1 (Experimental)**
- Goal: keep the core stable, add extensions as optional layers / stack formats / intent schemas.

---

## v0.1 (Current) — Minimal Viable Protocol

Delivered docs:
- `README.md`
- `docs/spec.md`
- `docs/aee-binding.md`
- `docs/stacks.md`
- `docs/observability.md`

Core features:
- Layer contract (decisions, context delta, actions, control flags)
- Pipeline or DAG stacks
- Bypass/branch semantics (audited)
- AEE binding via `aocl.*` intents and AEE `event`/`task`/`result`

---

## v0.2 — Schemas & Validators (High Leverage)

1) **JSON Schema for AOCL layer payloads**
- `schemas/aocl-layer-decision.schema.json`
- `schemas/aocl-run-summary.schema.json`

2) **Reference validators**
- TypeScript and Python validators for:
  - stack files
  - AOCL layer outputs
  - AOCL→AEE event payloads

3) **Canonicalization for digests**
- Define how to compute stable digests for:
  - `context_in/out`
  - delta patches
  - stack definitions

---

## v0.3 — Transport Bindings (Optional)

Add short docs for recommended bindings:
- HTTP (webhooks / REST)
- WebSocket (interactive sessions)
- NATS (low-latency events)
- Kafka (durable event logs)

Each binding doc should cover:
- where `corr` lives
- ordering expectations
- batching and backpressure
- idempotency guidance

---

## v0.4 — Control-Plane Hardening

1) **Bypass policy standard**
- stack-level `bypass_policy` schema
- role-based access + “never bypass” rules
- mandatory audit event shapes for allow/deny

2) **Branch selection standard**
- `aocl.stack.select` / `aocl.control.branch` payload schemas
- minimal expression language for `when` conditions (or a standard subset)

3) **HITL (Human-in-the-loop) handshake**
- a minimal approval protocol:
  - request approval
  - grant/deny
  - attach evidence refs
  - record approval in trace

---

## v0.5 — Observability Upgrades

1) **OpenTelemetry mapping**
- Map AOCL layers to OTel spans
- Recommended use of AEE `trace` field (`trace_id`, `span_id`)

2) **Replay tooling**
- “Re-run corr” with stored:
  - stack version
  - context deltas
  - refs and digests
- deterministic replays for debugging

3) **Reference UI**
- a minimal web panel that shows:
  - layer timeline
  - context diffs
  - branch map
  - delegations and results

---

## v0.6 — Reference Implementations

Provide tiny adapters:
- “AOCL runner” skeleton in Python
- “AOCL runner” skeleton in TypeScript
- examples integrating:
  - LangGraph
  - AutoGen
  - CrewAI
  - custom agent runtimes

Focus: demonstrate AOCL compliance and AEE audit emission.

---

## Future Ideas (Parked)

- Layer marketplaces / signed layer plugins
- Stack signing and provenance (supply chain)
- Multi-tenant policy packs (org-level AOCL stacks)
- Formal conformance test suite + certification
- More relationship docs (MCP/ACP/frameworks) as the ecosystem evolves
