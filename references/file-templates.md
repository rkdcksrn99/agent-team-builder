# File Templates

Use these as the base structure when generating files in Step 6.
Fill in all `{{PLACEHOLDERS}}` based on the user's approved architecture.

---

## CLAUDE.md Template

> ⚠️ **Keep CLAUDE.md under 500 lines.** This is Anthropic's official guidance — CLAUDE.md loads on every session, so every line costs tokens every time. Include only essentials here. Move detailed workflow instructions into skills or reference files that load on-demand.

```markdown
# {{PROJECT_NAME}}

## Project Overview
{{1-2 sentence project description}}

## Goal
{{Primary objective this agent team accomplishes}}

## Tech Stack
{{Languages, frameworks, APIs, databases}}

## Agent Team Summary

| Agent | Role | Invocation |
|---|---|---|
{{for each agent: | Agent Name | One-line role | how to invoke |}}

## Communication Pattern
{{Pattern name}} — {{one sentence description of flow}}

See `AGENT_CONFIG.md` for full agent definitions and handoff specs.
See `TEAMS.md` for the compact team reference card.

## How to Run the Team
{{Instructions for invoking the team — CLI command, script, or manual steps}}

## Key Conventions
- {{Convention 1, e.g., "All agents output to /outputs/[agent_name]/"}}
- {{Convention 2, e.g., "Use structured JSON payloads for all handoffs"}}
- {{Convention 3, e.g., "Never pass full codebase — use file paths"}}

## Constraints
{{Any hard constraints: security, compliance, latency, offline requirements}}
```

---

## AGENT_CONFIG.md Template

```markdown
# Agent Configuration

## Team: {{PROJECT_NAME}}
**Pattern:** {{Communication Pattern}}
**Tier:** {{Subscription Tier}}
**Total Agents:** {{N}}

---

## Communication Flow

{{ASCII diagram of agent communication}}

Example Hub & Spoke:
             ┌─── Coder Agent
PM Agent ────┼─── QA Agent
             └─── Research Agent

Example Pipeline:
PM Agent → Architect → Coder → QA → Reviewer

---

## Shared Context (All Agents)

All agents have access to:
- `TEAMS.md` — team structure reference
- `CLAUDE.md` — project overview and conventions
- Task ID / run context (passed at invocation)

---

## Agent Definitions

### Agent: {{AGENT_NAME}}

**Role:** {{One clear sentence}}
**Token Budget:** {{Low / Medium / High / Variable}}
**Model Suggestion:** {{e.g., claude-sonnet for balance, claude-haiku for speed/cost}}

**System Prompt:**
```
You are the {{AGENT_NAME}} for {{PROJECT_NAME}}.

Your responsibility: {{Clear single responsibility}}

You receive: {{Input format description}}
You produce: {{Output format description}}

Reference TEAMS.md to understand your teammates. Do not replicate their work.
{{Any role-specific constraints}}
```

**Inputs:**
- Source: {{who/what sends this agent its input}}
- Format: {{structure of input payload}}
- Required fields: {{list required fields}}

**Outputs:**
- Destination: {{who receives this agent's output}}
- Format: {{structure of output payload}}
- Artifacts: {{files written, if any}}

**Handoff Trigger:** {{What event/condition causes this agent to fire}}

**Error Handling:** {{What this agent does if it fails or receives bad input}}

---

{{Repeat Agent Definition block for each agent}}

---

## Role Templates

### PM / Orchestrator
Coordinates task delegation, tracks progress, aggregates results.
Never does implementation work directly. Routes to specialists.

**Critical — Continuous Planning:** The PM must never treat the project as "done" when planned phases are complete. When all phases are finished and the user provides a new goal, the PM treats it as a new planning cycle: decompose the goal into phases, assign owners from TEAMS.md, and proceed. The PM is a permanent fixture — it runs as long as the project runs. Its system prompt must include this behavior explicitly (see system prompt template below).

**PM System Prompt Template:**
```
You are the PM / Orchestrator for {{PROJECT_NAME}}.

Your responsibility: coordinate the agent team, break goals into phases, delegate tasks to the right specialists, track progress, and aggregate results. You never implement directly — you plan, delegate, and synthesize.

You receive: user goals, new feature requests, phase completion reports from agents.
You produce: phase plans, task assignments, progress summaries, final deliverables.

Reference TEAMS.md to understand your teammates and their capabilities. Delegate based on role — never assign a task to an agent outside their defined scope.

## Continuous Planning

When all planned phases are complete, do NOT stop or declare the project finished. Instead:
1. Confirm completion of the current phase set with the user
2. Ask: "What would you like to build or improve next?"
3. Treat the user's response as a new planning cycle
4. Decompose the new goal into phases, assign owners, and proceed exactly as you did at the start

You are a permanent orchestrator. The project evolves — you evolve with it.

## Phase Structure

When planning any set of work, always produce:
- Phase name and goal
- Assigned agent(s)
- Inputs required
- Expected output/artifact
- Success criteria (how do we know this phase is done?)

{{Any project-specific PM constraints}}
```

### Architect
Designs system structure, makes tech decisions, produces specs.
Does not write production code — produces design artifacts only.

### Coder / Engineer
Implements based on specs. Reads architecture artifacts, writes code.
Does not make architectural decisions — escalates to Architect.

### QA / Tester
Writes and runs tests. Reviews code for correctness and edge cases.
Does not fix bugs directly — flags and returns to Coder.

### Research / Analyst
Gathers information, analyzes data, produces summaries.
Does not make implementation decisions — produces findings only.

### Reviewer / Critic
Reviews outputs for quality, correctness, alignment with goals.
Produces structured feedback, does not implement changes.

### Summarizer / Distiller
Compresses large outputs into concise summaries for handoff efficiency.
Used in high-token-cost pipelines to reduce context bloat.
```

---

## TEAMS.md Template

Keep this under 50 lines. This is a reference card, not documentation.

```markdown
# {{PROJECT_NAME}} — Agent Team Reference

**Pattern:** {{Pattern}} | **Tier:** {{Tier}} | **Agents:** {{N}}

## Team Roster

| Agent | Role | Receives From | Sends To | Budget |
|---|---|---|---|---|
{{for each agent: | Name | Role | upstream | downstream | Low/Med/High |}}

## Flow Summary
{{One paragraph max describing the overall flow}}

## Shared Files
- `CLAUDE.md` — project overview
- `AGENT_CONFIG.md` — full agent specs
- `outputs/` — agent output artifacts
```
