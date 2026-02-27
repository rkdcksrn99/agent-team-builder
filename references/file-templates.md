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
