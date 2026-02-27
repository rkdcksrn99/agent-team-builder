# Agent Team Builder

**A Claude Code skill for designing and scaffolding production-ready multi-agent AI systems.**

Agent Team Builder acts as an opinionated architecture consultant — it interviews you about your project, recommends a token-efficient agent roster tailored to your Claude subscription tier, defines explicit communication patterns and handoff contracts between agents, validates the architecture before committing to files, and generates a complete set of production-ready configuration documents.

---

## Overview

Multi-agent systems in Claude Code are powerful but easy to over-engineer. The most common failure modes — redundant agents, bloated context passing, vague handoff specs, and CLAUDE.md files that grow without bound — all stem from skipping the architecture step and jumping straight to implementation.

This skill enforces a structured 7-step workflow that mirrors how a professional systems architect would approach the problem: scope first, design second, generate files last.

---

## Key Design Principles

### Token Efficiency by Default

Every architectural decision this skill produces is evaluated against token cost. Agents receive exactly the context they need — no more. This is not just a best practice; it directly affects how long sessions stay coherent, how much a team costs to run, and how well agents perform on longer tasks.

Specific mechanisms enforced:

- **Subscription-aware team sizing** — team size is capped based on the user's Claude tier (Free/Pro → 2–3 agents, Team → 3–5, Enterprise/Max → 5–8)
- **Role-scoped system prompts** — each agent's prompt contains only its own role, inputs, and outputs; team context is loaded from `TEAMS.md` by reference, not duplicated inline
- **Structured JSON handoffs** — inter-agent payloads are defined explicitly with required fields, preventing agents from passing full conversation history or raw file dumps downstream
- **CLAUDE.md under 500 lines** — enforces Anthropic's official guidance; detailed workflow instructions are offloaded to on-demand reference files rather than loading into every session
- **Summary artifacts** — agents in high-budget roles produce a compact summary alongside full output, so downstream agents receive the summary by default and request full detail only when needed

### Explicit Handoff Contracts

Vague agent-to-agent connections ("Agent A feeds Agent B") are flagged and rejected. Every relationship in the generated architecture includes three things: a **trigger** (what event causes the handoff), a **payload** (the exact structured data passed, minimized to what the receiving agent actually needs), and an **error path** (what happens on failure). This makes the system debuggable and prevents silent failures in production runs.

### Consultant-Led, Not Menu-Driven

The skill is designed to lead with a recommendation rather than present open-ended choices. Users provide project context; the skill returns a concrete architecture proposal with reasoning. Users can override any decision, but the default is a specific point of view — which produces better outcomes than asking users to choose from an undifferentiated list of options.

---

## Workflow

The skill follows a 7-step process designed to prevent common failure modes in multi-agent system design:

| Step | Description |
|---|---|
| 1. Project Interview | Collects project goals, tech stack, subscription tier, priorities, risks, and timeline |
| 2. Architecture Recommendation | Proposes complexity tier, agent roster, token budgets, and efficiency rationale |
| 3. Fine-Tuning | User reviews and adjusts the roster; skill pushes back on redundancy |
| 4. Communication Patterns | Recommends Sequential, Hub & Spoke, Parallel, or Hybrid pattern with full handoff specs |
| 5. Team Validation | Validates each agent against a quality checklist before any files are generated |
| 6. File Generation | Produces `CLAUDE.md`, `AGENT_CONFIG.md`, and `TEAMS.md` from battle-tested templates |
| 7. Spawn Script (Optional) | Generates `spawn_team.sh` or `spawn_team.py` to bootstrap the team programmatically |

---

## Generated Outputs

| File | Purpose |
|---|---|
| `CLAUDE.md` | Top-level project instructions for Claude Code. Kept under 500 lines per Anthropic's token efficiency guidance. |
| `AGENT_CONFIG.md` | Master configuration — full agent definitions, system prompts, communication diagram, role templates, and shared context rules. |
| `TEAMS.md` | Compact team reference card (≤50 lines). The file agents load to understand team structure without duplicating full configs. |
| `spawn_team.sh` / `.py` | *(Optional)* Bootstrap script that spawns agents in the correct order with minimal structured context per agent. |

---

## Installation

### Via `.skill` file (recommended)

1. Download `agent-team-builder.skill` from the [Releases](../../releases) page
2. In Claude.ai, go to **Settings → Customize → Add Skill**
3. Upload the `.skill` file — all reference files are bundled inside

### Via folder

```bash
git clone https://github.com/rkldcksrn99/agent-team-builder.git
cp -r agent-team-builder ~/.claude/skills/
```

---

## Usage

Once installed, describe your project in Claude Code and the skill triggers automatically:

```
"Help me design a multi-agent team for my options flow analysis system"
"I want to build a Claude Code agent architecture for a data pipeline"
"Set up a multi-agent system for automated security scanning"
```

---

## Repository Structure

```
agent-team-builder/
├── SKILL.md                          # Core skill — 7-step workflow and principles
├── README.md                         # This file
├── LICENSE                           # MIT
├── .gitignore
└── references/
    ├── communication-patterns.md     # Pattern specs, decision tree, handoff templates
    ├── token-efficiency.md           # Token budgeting rules, waste patterns, payload templates
    ├── file-templates.md             # CLAUDE.md, AGENT_CONFIG.md, TEAMS.md templates
    └── spawn-templates.md            # spawn_team.sh and spawn_team.py boilerplate
```

Reference files use progressive disclosure — they are loaded by the skill only when the relevant step is reached, keeping base context lean.

---

## Supported Communication Patterns

| Pattern | Best For |
|---|---|
| Sequential Pipeline | Linear workflows where each step depends on the previous output |
| Hub & Spoke | A clear coordinator/PM role that delegates to specialists |
| Parallel Fan-Out | Independent subtasks that can run simultaneously |
| Hybrid | Complex projects with multiple phases requiring both patterns |

---

## Claude Subscription Tier Support

| Tier | Max Agents | Strategy |
|---|---|---|
| Free / Pro | 2–3 | Lean context, aggressive summarization, minimal handoff payloads |
| Team | 3–5 | Balanced separation of concerns, structured JSON payloads |
| Enterprise / Max | 5–8 | Full specialization supported, richer context permitted |

---

## License

MIT

---

## Contributing

Pull requests welcome. Areas of particular interest:

- Domain-specific agent role templates (security, data engineering, ML pipelines)
- Additional communication pattern variants
- Real-world `AGENT_CONFIG.md` examples from production projects
