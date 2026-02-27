# Communication Patterns Reference

## Pattern 1: Sequential Pipeline

```
Agent A → Agent B → Agent C → Agent D
```

**When to use:**
- Linear workflows where each step depends on the previous
- Data transformation pipelines (ingest → process → analyze → report)
- Code generation workflows (plan → code → test → review)

**Handoff structure:**
```
trigger: Agent A completes its output artifact
payload: { output_artifact, metadata, errors[] }
next: Agent B receives payload as its primary input
```

**Pros:** Simple, predictable, easy to debug
**Cons:** Slow (sequential), one failure blocks the whole chain
**Token cost:** Low — each agent only sees its slice of context

**Error handling:** If Agent N fails, halt pipeline and return error with stage label.
Optionally allow retry with exponential backoff before halting.

---

## Pattern 2: Hub & Spoke (Orchestrator)

```
         ┌─── Specialist A
PM Agent ─── Specialist B
         └─── Specialist C
```

**When to use:**
- Projects with a clear coordinator role (PM, Architect, Orchestrator)
- Tasks that need to be delegated and results aggregated
- When specialists need different context but share a common goal

**Handoff structure:**
```
trigger: PM Agent determines task requires specialist
payload: { task_description, relevant_context_only, expected_output_format }
return: Specialist returns { result, confidence, issues[] } to PM Agent
```

**Pros:** Clear authority, easy to add/remove specialists
**Cons:** PM Agent becomes a bottleneck; can accumulate large context
**Token cost:** Medium — PM Agent context grows with each specialist result

**Error handling:** PM Agent retries failed specialist once, then either skips or escalates
to user depending on task criticality.

---

## Pattern 3: Parallel Fan-Out

```
Orchestrator
├─── Agent A (runs simultaneously)
├─── Agent B (runs simultaneously)
└─── Agent C (runs simultaneously)
        ↓
    Aggregator Agent
```

**When to use:**
- Independent subtasks that don't depend on each other
- Research/analysis tasks that can be parallelized
- When speed matters more than cost

**Handoff structure:**
```
trigger: Orchestrator splits work into N independent tasks
payload: { task_slice, shared_context (minimal) }
aggregation: All results collected, passed to Aggregator with { results[], original_goal }
```

**Pros:** Fast, scalable
**Cons:** Higher token cost (shared context duplicated N times), aggregation complexity
**Token cost:** High — context is replicated per parallel agent

**Error handling:** Aggregator handles partial results. Define minimum viable result
threshold (e.g., "proceed if at least 2/3 agents succeed").

---

## Pattern 4: Hybrid

```
Planner → [Fan-Out: Research A, Research B] → Synthesizer → [Pipeline: Coder → QA]
```

**When to use:**
- Complex projects with both independent and dependent stages
- When some phases benefit from parallelism and others need sequential ordering
- Recommended for "Full" complexity tier projects (6-8 agents)

**Design rules for Hybrid:**
1. Map out all tasks first, identify which are independent vs dependent
2. Group independent tasks into parallel fan-out stages
3. Connect stages with sequential handoffs
4. Minimize context that crosses stage boundaries (use summaries, not full outputs)

**Token cost:** High — design carefully. Use `TEAMS.md` as the shared reference
instead of passing full agent configs between stages.

---

## Choosing a Pattern: Quick Decision Tree

```
Is there a clear coordinator/PM role?
├── Yes → Hub & Spoke (or Hybrid if some tasks are independent)
└── No → Are tasks independent of each other?
    ├── Yes → Parallel Fan-Out
    └── No → Sequential Pipeline
```

If the project has 2+ distinct phases with different structures → **Hybrid**.

---

## Handoff Payload Best Practices

Always define payloads explicitly. Avoid passing full file contents when a summary suffices.

| Instead of... | Pass... |
|---|---|
| Full codebase | File paths + function signatures |
| Full conversation history | Summarized decision log |
| All previous agent outputs | Only the output the next agent needs |
| Raw API responses | Parsed, structured subset |

**Rule:** If the payload is > 2000 tokens, ask "can this be summarized?" before passing it.
