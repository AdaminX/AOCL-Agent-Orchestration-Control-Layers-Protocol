# AOCL Protocol Spec (v0.1)

AOCL (Agent Orchestration Control Layers) standardizes how an orchestrator processes an incoming event by passing it through **layers** that may transform context, apply policies, route work, and delegate tasks to agents/tools.

AOCL is designed to be **runtime-agnostic** and **framework-agnostic**.

- AOCL **does not** define how agents are built, how tools execute, or how workers are scheduled.
- AOCL **does** define a layered control pipeline (or graph) and the minimum contract each layer must follow.
- AOCL is intended to be **auditable** by emitting layer activity as **AEE envelopes** (see `docs/aee-binding.md`).

---

## 1. Terminology

### Intent (Input)
A triggering event such as:
- user message
- monitoring alert
- webhook / API call
- scheduled tick

AOCL does not constrain the event format, but commonly it will arrive as an **AEE `task` envelope**.

### Orchestrator
The component that runs AOCL stacks. It may be a single process or distributed.

### Layer
A named unit of control logic that takes an input state and produces:
- a possibly modified state
- decisions and/or actions
- control flags that influence routing (branch/bypass/halt)

Layers can be deterministic code, LLM calls, or hybrids.

### Stack
A configured sequence or graph of layers used for a class of tasks (e.g., `default`, `realtime-alert`, `deep-research`).

### Context Bundle
A structured state object passed through AOCL layers.

**Design goal:** context should remain inspectable and replayable while avoiding bloated “hidden state”.

---

## 2. Core Design Principles

1. **Layered control**  
   Work flows through ordered layers with clear responsibilities.

2. **Branchable, bypassable**  
   AOCL supports alternate paths and skipping layers — but **decisions must be auditable**.

3. **Delta-first context**  
   Layers should emit **context deltas** (patches) and refs/digests rather than re-sending huge objects.

4. **Audit-first**  
   AOCL is intended to produce a trace where you can answer:
   - what happened?
   - why?
   - what changed?
   - who/what approved bypasses?
   - what evidence supports the result?

---

## 3. Context Bundle (Recommended Partitions)

AOCL does not mandate a single schema, but recommends a partitioned bundle to keep control clean:

- **C0 Event**: Source, timestamp, channel, correlation IDs, attachments
- **C1 Identity & Scope**: User/org, roles, permissions, secret scope, redaction rules
- **C2 Task**: Goal, constraints, definition of done, priority
- **C3 Memory/Knowledge**: Retrieved notes, RAG hits, file refs, summaries
- **C4 Policy**: Safety/compliance rules, allowed tools, model restrictions
- **C5 Execution**: Budgets, timeouts, concurrency, tool registry, model routing
- **C6 Audit**: Log level, required evidence, compliance checkpoints
