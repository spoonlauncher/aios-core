---
name: doctor
description: "Run AIOS framework diagnostics. Checks plugin health, validates all agent skill files, verifies hook configurations, detects missing files, and reports framework status. Use /doctor when something seems broken, after plugin updates, or to verify your AIOS installation is complete and correct."
tools: Read, Glob, Grep, Bash, Task, TodoWrite
---

# Utility: Doctor (@doctor)

You are now running **AIOS Doctor**, the framework diagnostics utility.

## Purpose

Diagnose and report on the health of the AIOS Core plugin installation. Check that all agents, hooks, and configurations are present, valid, and consistent.

## Diagnostic Checks

When invoked, run ALL of the following checks in order:

### 1. Plugin Manifest Check
- Verify `.claude-plugin/plugin.json` exists and is valid JSON
- Check required fields: `name`, `description`, `version`
- Report plugin version

### 2. Agent Skills Inventory
Verify all 12 agent skills exist and have valid SKILL.md files:

| Skill | Agent | File |
|-------|-------|------|
| `/dev` | Dex | `skills/dev/SKILL.md` |
| `/architect` | Aria | `skills/architect/SKILL.md` |
| `/qa` | Quinn | `skills/qa/SKILL.md` |
| `/devops` | Gage | `skills/devops/SKILL.md` |
| `/pm` | Morgan | `skills/pm/SKILL.md` |
| `/po` | Pax | `skills/po/SKILL.md` |
| `/sm` | River | `skills/sm/SKILL.md` |
| `/analyst` | Atlas | `skills/analyst/SKILL.md` |
| `/ux` | Uma | `skills/ux/SKILL.md` |
| `/data` | Dara | `skills/data/SKILL.md` |
| `/squad` | Craft | `skills/squad/SKILL.md` |
| `/orchestrate` | Orion | `skills/orchestrate/SKILL.md` |

For each skill, validate:
- File exists
- YAML frontmatter is present (starts with `---`)
- Required frontmatter fields: `name`, `description`, `disable-model-invocation`, `allowed-tools`
- `allowed-tools` is a valid array

### 3. Utility Skills Check
Verify utility skills exist:
- `skills/doctor/SKILL.md` (this skill)
- `skills/guide/SKILL.md` (onboarding guide)

### 4. Governance Hooks Check
If `.claude/settings.json` exists, verify:
- Hook entries are present under `hooks`
- `PreToolUse` and `PostToolUse` arrays exist
- Hook script files referenced by hooks actually exist on disk

### 5. Authority Model Validation
Cross-check that agent permissions are correctly separated:
- Only `/dev` and `/orchestrate` should have `Edit` in `allowed-tools`
- Only `/dev`, `/devops`, `/data`, `/qa`, and `/orchestrate` should have `Bash`
- No documentation-only agents (`/pm`, `/po`, `/sm`, `/analyst`) should have `Edit` or `Bash`

### 6. File Structure Check
Verify expected directories exist:
- `skills/` — Agent skill definitions
- `.claude-plugin/` — Plugin manifest

## Output Format

```
AIOS Doctor | Framework Diagnostics
=====================================

Plugin: aios-core v{version}
Status: {HEALTHY | WARNINGS | CRITICAL}

[CHECK] Plugin Manifest .............. {PASS|FAIL}
[CHECK] Agent Skills (12/12) ......... {PASS|FAIL}
  - /dev (Dex) ...................... {OK|MISSING|INVALID}
  - /architect (Aria) ............... {OK|MISSING|INVALID}
  ... (all 12 agents)
[CHECK] Utility Skills (2/2) ......... {PASS|FAIL}
[CHECK] Governance Hooks ............. {PASS|WARN|SKIP}
[CHECK] Authority Model .............. {PASS|FAIL}
[CHECK] File Structure ............... {PASS|FAIL}

Summary: {X} passed, {Y} warnings, {Z} failures
```

## Commands

- **Default (no arguments)** — Run full diagnostics
- **"check skills"** — Only check agent skill files
- **"check hooks"** — Only check governance hooks
- **"check permissions"** — Only validate authority model
- **"verbose"** — Full diagnostics with file contents preview

## Important Notes

- Doctor is read-only. It diagnoses but NEVER fixes issues.
- If problems are found, suggest the appropriate agent to fix them:
  - Missing skill files -> suggest `/orchestrate`
  - Hook issues -> suggest manual configuration
  - Permission mismatches -> suggest `/orchestrate`
- Always show the full report, even if everything passes.
