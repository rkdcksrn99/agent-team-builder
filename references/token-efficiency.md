# Token Efficiency Guide for Agent Teams

The goal: **every agent gets exactly what it needs, nothing more.**

---

## Subscription Tier Budgets

Use these as guardrails when designing teams:

| Tier | Recommended Max Agents | Context Strategy |
|---|---|---|
| Free / Pro | 2-3 | Lean context, summarize aggressively |
| Team | 3-5 | Balanced, structured payloads |
| Enterprise / Max | 5-8 | Richer context OK, still avoid redundancy |

---

## The 5 Token Efficiency Rules

### Rule 1: No Full-Context Passing
Never pass an agent's full output to the next agent unless required.
Always ask: "What is the minimum information Agent B needs to do its job?"

**Bad:** Pass 10,000 token codebase to QA Agent
**Good:** Pass function signatures + changed files list + test requirements

### Rule 2: Use TEAMS.md as Shared Reference
All agents should reference `TEAMS.md` to understand the team structure.
This prevents duplicating agent descriptions in every system prompt.
TEAMS.md should stay under 50 lines — it's a reference card, not a manual.

### Rule 3: Summarize Before Handing Off
If an agent produces a large output, have it produce a **summary artifact** alongside
the full output. The next agent in the pipeline receives the summary by default;
it can request the full output only if needed.

```
Agent output structure:
{
  "summary": "2-5 sentence summary of what was done and key findings",
  "output_ref": "path/to/full_output.md",
  "key_data": { ...only the fields the next agent needs... }
}
```

### Rule 4: Role-Scoped System Prompts
Each agent's system prompt should only contain:
- Its own role and responsibilities
- Its expected input format
- Its expected output format
- Reference to TEAMS.md for team context (not inline team descriptions)

Do NOT include: other agents' full system prompts, full project history,
complete codebase context unless the agent's role explicitly requires it.

### Rule 5: Lazy Context Loading
Agents should load detailed context only when needed:
- PM Agent doesn't need the full codebase — it needs the task list
- QA Agent doesn't need the architecture doc — it needs the code + test requirements
- Architect Agent doesn't need runtime logs — it needs requirements

---

## Token Budget Labels

Use these labels when defining agents in AGENT_CONFIG.md:

| Label | Approx Context Usage | Typical Roles |
|---|---|---|
| `Low` | < 2K tokens per invocation | Router, Dispatcher, Summarizer |
| `Medium` | 2K–8K tokens per invocation | PM, QA, Researcher |
| `High` | 8K–32K tokens per invocation | Coder, Architect, Analyzer |
| `Variable` | Depends on task size | Multi-purpose agents |

---

## Common Token Waste Patterns (Avoid These)

1. **The Echo Agent** — an agent that just repeats the previous agent's output with minor edits. Merge it.
2. **The Librarian Agent** — an agent that holds all project context and is queried by everyone. Use a shared file instead.
3. **The Over-briefed Agent** — every agent gets the full project spec. Use role-scoped context instead.
4. **The Chatty Handoff** — agents pass natural language summaries when a structured JSON payload would be 5x smaller.
5. **The Redundant Reviewer** — two agents both reviewing the same output with no differentiated lens. Merge or specialize.

---

## Context Passing Template (Efficient)

Use this structure for inter-agent payloads:

```json
{
  "from_agent": "coder",
  "to_agent": "qa",
  "task_id": "feature-x",
  "summary": "Implemented feature X. Modified files: auth.py, routes.py. Added 3 new functions.",
  "artifacts": ["src/auth.py", "src/routes.py"],
  "flags": ["needs_edge_case_testing", "new_dependency_added"],
  "full_output_ref": "outputs/coder_feature_x.md"
}
```

The receiving agent reads `summary` and `flags` first. Only reads `artifacts` or
`full_output_ref` if its task requires it.
