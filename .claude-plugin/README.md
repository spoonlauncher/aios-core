# AIOS Core - Claude Code Plugin

> AI Operating System with 12 specialized agents for the full software development lifecycle.

**Version**: 4.4.6
**Author**: SpoonLauncher
**License**: See repository root

## Overview

AIOS Core transforms Claude Code into a multi-agent development environment. Each agent is a specialist with distinct expertise, authority boundaries, and collaboration protocols. Agents never step outside their domain — they delegate to the right specialist.

## Quick Start

### Install

```bash
claude plugin add spoonlauncher/aios-core
```

### Use an Agent

Type `/agent-name` in Claude Code to activate any agent:

```
/dev          # Activate Dex, Full Stack Developer
/architect    # Activate Aria, System Architect
/qa           # Activate Quinn, QA Engineer
```

## Agents

| Skill | Agent | Role | Archetype |
|-------|-------|------|-----------|
| `/dev` | Dex | Full Stack Developer | Builder |
| `/architect` | Aria | System Architect | Visionary |
| `/qa` | Quinn | QA Engineer | Guardian |
| `/devops` | Gage | DevOps Specialist | Operator |
| `/pm` | Morgan | Product Manager | Strategist |
| `/po` | Pax | Product Owner | Prioritizer |
| `/sm` | River | Scrum Master | Facilitator |
| `/analyst` | Atlas | Business Analyst | Decoder |
| `/ux` | Uma | UX/UI Designer | Empathizer |
| `/data` | Dara | Database Architect | Sage |
| `/squad` | Craft | Squad Creator | Builder |
| `/orchestrate` | Orion | Master Orchestrator | Orchestrator |

### Utility Skills

| Skill | Purpose |
|-------|---------|
| `/doctor` | Framework diagnostics, health checks, validation |
| `/guide` | Onboarding, help, agent discovery |

## Authority Model

Every agent has strict boundaries. This prevents role confusion and ensures quality.

### Who Can Do What

| Action | Exclusive Agent |
|--------|----------------|
| Write/edit source code | `/dev` (and `/orchestrate` as master) |
| Git push, releases, CI/CD | `/devops` only |
| Architecture decisions | `/architect` only |
| Quality verdicts (PASS/FAIL) | `/qa` only |
| Create PRDs and epics | `/pm` only |
| Manage backlog priority | `/po` only |
| Create user stories | `/sm` only |
| Database migrations, RLS | `/data` only |

### Key Rules

- **No code without stories**: `/dev` requires a user story before implementation
- **Architecture first**: Writes to `supabase/functions/` and `supabase/migrations/` are blocked unless architecture docs exist
- **Git push is exclusive**: Only `/devops` can push — all other agents are blocked
- **QA never fixes code**: `/qa` identifies issues and reports them, never edits source
- **SM never modifies code**: `/sm` creates stories and facilitates, never touches source files
- **Delegation over emulation**: Agents suggest switching to the right specialist rather than doing another agent's job

## Tool Permissions

Each agent gets only the tools it needs:

| Agent | Read | Edit | Write | Bash | Glob | Grep | Task | WebFetch | WebSearch |
|-------|------|------|-------|------|------|------|------|----------|-----------|
| `/dev` | Y | Y | Y | Y | Y | Y | Y | Y | Y |
| `/architect` | Y | - | Y | - | Y | Y | Y | Y | Y |
| `/qa` | Y | - | Y | Y | Y | Y | Y | - | - |
| `/devops` | Y | - | Y | Y | Y | Y | Y | - | - |
| `/pm` | Y | - | Y | - | Y | Y | Y | Y | Y |
| `/po` | Y | - | Y | - | Y | Y | Y | - | Y |
| `/sm` | Y | - | Y | - | Y | Y | Y | - | - |
| `/analyst` | Y | - | Y | - | Y | Y | Y | Y | Y |
| `/ux` | Y | - | Y | - | Y | Y | Y | - | Y |
| `/data` | Y | - | Y | Y | Y | Y | Y | - | - |
| `/squad` | Y | - | Y | - | Y | Y | Y | - | - |
| `/orchestrate` | Y | Y | Y | Y | Y | Y | Y | Y | Y |

**Key**: Y = allowed, - = not available

## Governance Hooks

The plugin includes 6 governance hook scripts at `.claude/hooks/`. These enforce authority boundaries at the tool level.

### Hook Summary

| Hook | Trigger | Action |
|------|---------|--------|
| `enforce-architecture-first.py` | Write/Edit | Blocks code in `supabase/functions/` and `supabase/migrations/` without architecture docs |
| `enforce-git-push-authority.sh` | Bash | Blocks `git push` commands (only `/devops` should push) |
| `write-path-validation.py` | Write/Edit | Warns when documentation files are written to incorrect paths |
| `read-protection.py` | Read | Blocks partial reads on protected files (CLAUDE.md, settings.json, agent definitions) |
| `slug-validation.py` | Bash | Validates SQL INSERT/UPDATE slug values use snake_case |
| `sql-governance.py` | Bash | Blocks dangerous SQL (DROP TABLE, TRUNCATE, DELETE without WHERE) |

### Hook Configuration

Hooks are project-level configuration, not bundled in the plugin. Add them to your `.claude/settings.json`:

```json
{
  "hooks": {
    "PreToolUse": [
      {
        "matcher": "Write|Edit",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"${CLAUDE_PROJECT_DIR:-.}/.claude/hooks/enforce-architecture-first.py\"",
            "timeout": 10
          },
          {
            "type": "command",
            "command": "python3 \"${CLAUDE_PROJECT_DIR:-.}/.claude/hooks/write-path-validation.py\"",
            "timeout": 10
          }
        ]
      },
      {
        "matcher": "Bash",
        "hooks": [
          {
            "type": "command",
            "command": "bash \"${CLAUDE_PROJECT_DIR:-.}/.claude/hooks/enforce-git-push-authority.sh\"",
            "timeout": 10
          },
          {
            "type": "command",
            "command": "python3 \"${CLAUDE_PROJECT_DIR:-.}/.claude/hooks/slug-validation.py\"",
            "timeout": 10
          },
          {
            "type": "command",
            "command": "python3 \"${CLAUDE_PROJECT_DIR:-.}/.claude/hooks/sql-governance.py\"",
            "timeout": 10
          }
        ]
      },
      {
        "matcher": "Read",
        "hooks": [
          {
            "type": "command",
            "command": "python3 \"${CLAUDE_PROJECT_DIR:-.}/.claude/hooks/read-protection.py\"",
            "timeout": 10
          }
        ]
      }
    ]
  }
}
```

**Requirements**: Python 3.x and Bash must be available in your PATH.

## Typical Workflows

### Start a New Feature

```
/pm           # Define requirements, create PRD
/architect    # Design the architecture
/sm           # Break into user stories
/dev          # Implement the code
/qa           # Review and test
/devops       # Push and deploy
```

### Research Phase

```
/analyst      # Market research, competitive analysis
/pm           # Synthesize into product requirements
/ux           # User research and wireframes
```

### Database Work

```
/architect    # Define data architecture
/data         # Create schemas, migrations, RLS policies
/dev          # Implement data access layer
```

### Multi-Agent Coordination

```
/orchestrate  # Plan and coordinate across agents
```

## Agent Collaboration Protocol

Agents communicate through delegation, not emulation:

- When `/dev` encounters an architecture question, it suggests: "Switch to `/architect` for this decision"
- When `/qa` finds a bug, it reports the issue — it never fixes the code
- When `/pm` needs market data, it suggests: "Switch to `/analyst` for research"
- `/orchestrate` (Orion) coordinates multi-agent workflows and tracks progress

## IDS (Incremental Development System)

The orchestrator manages the IDS — a framework for component reuse:

```
Need a component?
  |
  v
IDS CHECK -> Exists?
  |            |
  No          Yes -> Can REUSE as-is?
  |            |         |
  v           Yes       No -> Can ADAPT?
CREATE        |         |        |
  |          REUSE     Yes      No
  |           |       ADAPT   CREATE
  v           v         v       v
Register   Done    Modify   Register
```

Commands: `ids check`, `ids impact`, `ids register`, `ids health`, `ids stats`

## Project Structure

```
aios-core/
  .claude-plugin/
    plugin.json              # Plugin manifest
    README.md                # This file
  skills/
    dev/SKILL.md             # Full Stack Developer
    architect/SKILL.md       # System Architect
    qa/SKILL.md              # QA Engineer
    devops/SKILL.md          # DevOps Specialist
    pm/SKILL.md              # Product Manager
    po/SKILL.md              # Product Owner
    sm/SKILL.md              # Scrum Master
    analyst/SKILL.md         # Business Analyst
    ux/SKILL.md              # UX/UI Designer
    data/SKILL.md            # Database Architect
    squad/SKILL.md           # Squad Creator
    orchestrate/SKILL.md     # Master Orchestrator
    doctor/SKILL.md          # Diagnostics utility
    guide/SKILL.md           # Onboarding utility
  .claude/
    hooks/                   # Governance hook scripts
    settings.json            # Hook configuration
```

## Contributing

1. Fork the repository
2. Create a feature branch
3. Follow the agent authority model in your changes
4. Submit a pull request

## Links

- **Repository**: [github.com/spoonlauncher/aios-core](https://github.com/spoonlauncher/aios-core)
- **Original**: [github.com/SynkraAI/aios-core](https://github.com/SynkraAI/aios-core)
