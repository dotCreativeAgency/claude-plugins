# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin that provides an automated frontend development pipeline for React + MUI + TypeScript projects. The plugin consists of 5 specialized agents that work in a mandatory automated chain with enforced review cycles, a `/chain` command for explicit chain enforcement, plus a comprehensive MUI/React best practices skill.

**Repository Structure:**
- `/agents/` - 5 specialized agent definitions (architect, reviewer, debugger, refactorer, chrome-devtools-analyzer)
- `/commands/` - Slash commands (`/chain` for mandatory chain enforcement)
- `/skills/mui-react-development/` - MUI v7 + React patterns skill with reference documentation
- `/.claude-plugin/plugin.json` - Plugin metadata and configuration

## Plugin Architecture

### Agent Pipeline Flow

The agents form an automated chain that processes frontend components from creation through quality assurance:

```
frontend-architect (creates component)
    ↓
frontend-reviewer (scores code quality 0-100) ←─────────────────┐
    ↓                                                            │
    ├─> DONE (score ≥ 80, LOC ≤ 150)                            │
    ├─> frontend-debugger (score < 80 / CRITICAL/HIGH) ─────────┤ re-review
    └─> frontend-refactorer (LOC > 150 / maintainability < 15) ─┘ (max 2 cycles)

chrome-devtools-analyzer (independent, invoked by debugger or user)
```

**Key Design Principles:**
- **Mandatory Chain Enforcement**: Agents hand off to each other programmatically based on results with enforced review cycles (max 2 cycles)
- **Proactive Activation**: Agents trigger automatically based on bilingual (EN/IT) keyword detection
- **150 LOC Enforcement**: Hard limit for component size - automatic decomposition when exceeded
- **Quality Scoring**: 100-point system across Performance, Security, Maintainability, Accessibility
- **Re-review Cycles**: After debugger/refactorer completes, reviewer is re-launched automatically (max 2 cycles)
- **`/chain` Command**: Explicit chain enforcement mode that prohibits direct implementation
- **Project-Aware**: All agents read the target project's CLAUDE.md to adapt to conventions

### Agent Specifications

| Agent | Model | Color | Chain Position | Primary Responsibility |
|-------|-------|-------|----------------|------------------------|
| `frontend-architect` | opus | green | 1st (mandatory) | Component creation with MUI/React best practices |
| `frontend-reviewer` | opus | yellow | 2nd (mandatory) | Quality scoring + automatic chain orchestration |
| `frontend-debugger` | sonnet | red | 3rd (conditional) | Error resolution when score < 80 / CRITICAL/HIGH |
| `frontend-refactorer` | sonnet | purple | 3rd (conditional) | Component decomposition when LOC > 150 / maintainability < 15 |
| `chrome-devtools-analyzer` | sonnet | pink | Independent | Browser-level diagnostics via Chrome DevTools |

**Critical Implementation Details:**
- **Architect** (Chain Position 1st): Acts immediately on UI keywords without asking for confirmation; uses Sequential Thinking MCP server for requirements analysis; always consults MUI MCP and Context7 for API verification; MUST chain to reviewer after completion
- **Reviewer** (Chain Position 2nd): Outputs structured JSON score report; automatically invokes debugger if score < 80 OR refactorer if LOC > 150 / maintainability < 15; uses Memory MCP server to persist review sessions; re-launched after debugger/refactorer (max 2 cycles)
- **Debugger** (Chain Position 3rd, conditional): Requires Chrome DevTools MCP server; uses IDE Diagnostics MCP for TypeScript errors; can invoke chrome-devtools-analyzer for browser diagnostics; MUST trigger re-review after fixes
- **Refactorer** (Chain Position 3rd, conditional): Enforces 150 LOC limit; creates feature-based decomposition; updates imports and exports; MUST trigger re-review after refactoring
- **Chrome DevTools Analyzer** (Independent): Browser-level diagnostics via Chrome DevTools; can be invoked by debugger or directly by user; React-specific analysis (component tree, hydration errors, performance profiling)

### Command: `/chain`

Located in `/commands/chain.md`.

**Purpose**: Activates mandatory agent chain enforcement mode. When active, it is **forbidden** to implement React code directly -- every implementation task MUST go through the agent chain.

**Usage**: `/chain <implementation plan>`

**Chain sequence**:
1. `frontend-architect` -> Creates the component
2. `frontend-reviewer` -> Code review with numeric score (MANDATORY)
3. `frontend-debugger` -> If score < 80 or CRITICAL/HIGH issues (automatic)
4. `frontend-refactorer` -> If LOC > 150 or maintainability < 15/25 (automatic)
5. `frontend-reviewer` -> Re-validation after debugger/refactorer (max 2 cycles)

### Skill: mui-react-development

Located in `/skills/mui-react-development/SKILL.md` with reference documentation in `/references/`.

**Purpose**: Provides MUI v7, React 19, TanStack Query v5, React Hook Form + Zod, React Router v7 patterns.

**Critical Knowledge Areas:**
- **MUI v7 Breaking Changes**: Grid2/Unstable_Grid2 removed, unified Grid API with `size` prop
- **sx Prop Over Inline Styles**: All styling via sx prop for theme integration
- **Path Aliases**: Always use @/ aliases as defined in target project's tsconfig
- **150 LOC Rule**: Components exceeding 150 lines trigger automatic decomposition
- **WCAG 2.1 AA Compliance**: aria-labels, keyboard navigation, focus management required

## Modifying the Plugin

### Adding or Updating Agents

Agent files are located in `/agents/` and use YAML frontmatter + markdown content:

**Frontmatter Structure:**
```yaml
---
name: agent-name
description: "Triggering conditions and examples..."
model: opus | sonnet | haiku
color: green | yellow | red | purple | blue
---
```

**Important Conventions:**
- `description` field MUST include bilingual (EN/IT) triggering examples
- Use `<example>` blocks with `<commentary>` to show activation patterns
- Always specify which MCP tools the agent depends on
- Document auto-chaining logic if the agent calls other agents
- Prefix critical rules with "MANDATORY" or "FUNDAMENTAL RULES"

### Adding or Updating Skills

Skill files are in `/skills/{skill-name}/SKILL.md`:

**Frontmatter Structure:**
```yaml
---
name: skill-name
description: "When to activate this skill..."
---
```

**Content Organization:**
- Use progressive disclosure (## headers for major sections)
- Include "Critical Rules" or "Common Mistakes" sections
- Provide CORRECT/WRONG code examples
- Link to reference documentation in `/references/` subdirectory

### Plugin Metadata

Edit `/.claude-plugin/plugin.json` to update:
- `version` - Semantic versioning (major.minor.patch)
- `description` - Shown in plugin marketplace
- `keywords` - For plugin discovery

## MCP Server Dependencies

This plugin leverages these MCP servers when available in the target environment:

| MCP Server | Tools Used | Purpose |
|------------|------------|---------|
| MUI MCP | `useMuiDocs` | MUI v7 API documentation lookup |
| Context7 | `resolve-library-id`, `query-docs` | Library documentation (React, TanStack Query, Zod) |
| Chrome DevTools | `list_tabs`, `evaluate_expression`, `console_logs` | Live browser inspection for debugging |
| Memory | `create_memory`, `query_memories` | Persist review sessions across conversations |
| Sequential Thinking | `sequentialthinking` | Complex requirements analysis |
| IDE Diagnostics | `getDiagnostics` | TypeScript error checking |

**Note**: Agents gracefully degrade if MCP servers are unavailable, but functionality is limited.

## Testing the Plugin

Since this is a Claude Code plugin (markdown-based), there are no unit tests or build scripts. Testing involves:

1. **Manual Installation**: `/plugin install /path/to/frontend-pipeline`
2. **Activation Testing**: Verify agents trigger on keywords (e.g., "Create a users table")
3. **Chain Testing**: Ensure reviewer auto-invokes debugger/refactorer based on results
4. **MCP Integration**: Confirm agents can access MCP tools when available
5. **Bilingual Testing**: Test both EN and IT triggering keywords

## Distribution

The plugin is part of the `dotCreativeAgency/claude-plugins` marketplace monorepo:
- **Location**: `plugins/frontend-pipeline/` within the marketplace repository
- **Installation**: Via marketplace (`/plugin marketplace add dotCreativeAgency/claude-plugins`)
- **Development**: Clone the marketplace repo and edit in place

Update `LICENSE` and `README.md` before publishing new versions.
