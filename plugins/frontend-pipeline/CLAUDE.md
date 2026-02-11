# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin that provides an automated frontend development pipeline for React + MUI + TypeScript projects. The plugin consists of 4 specialized agents that work in an automated chain, plus a comprehensive MUI/React best practices skill.

**Repository Structure:**
- `/agents/` - 4 specialized agent definitions (architect, reviewer, debugger, refactorer)
- `/skills/mui-react-development/` - MUI v7 + React patterns skill with reference documentation
- `/.claude-plugin/plugin.json` - Plugin metadata and configuration

## Plugin Architecture

### Agent Pipeline Flow

The agents form an automated chain that processes frontend components from creation through quality assurance:

```
frontend-architect (creates component)
    ↓
frontend-reviewer (scores code quality 0-100)
    ↓
    ├─> DONE (score ≥ 80, LOC ≤ 150)
    ├─> frontend-debugger (score < 80)
    └─> frontend-refactorer (LOC > 150)
```

**Key Design Principles:**
- **Automatic Chaining**: Agents hand off to each other programmatically based on results (no manual invocation required)
- **Proactive Activation**: Agents trigger automatically based on bilingual (EN/IT) keyword detection
- **150 LOC Enforcement**: Hard limit for component size - automatic decomposition when exceeded
- **Quality Scoring**: 100-point system across Performance, Security, Maintainability, Accessibility
- **Project-Aware**: All agents read the target project's CLAUDE.md to adapt to conventions

### Agent Specifications

| Agent | Model | Color | Primary Responsibility |
|-------|-------|-------|------------------------|
| `frontend-architect` | opus | green | Component creation with MUI/React best practices |
| `frontend-reviewer` | opus | yellow | Quality scoring + automatic chain orchestration |
| `frontend-debugger` | sonnet | red | Error resolution with Chrome DevTools integration |
| `frontend-refactorer` | sonnet | purple | Component decomposition when LOC > 150 |

**Critical Implementation Details:**
- **Architect**: Acts immediately on UI keywords without asking for confirmation; uses Sequential Thinking MCP server for requirements analysis; always consults MUI MCP and Context7 for API verification
- **Reviewer**: Outputs structured JSON score report; automatically invokes debugger if score < 80 OR refactorer if LOC > 150; uses Memory MCP server to persist review sessions
- **Debugger**: Requires Chrome DevTools MCP server; uses IDE Diagnostics MCP for TypeScript errors; can launch local dev server for live inspection
- **Refactorer**: Enforces 150 LOC limit; creates feature-based decomposition; updates imports and exports; re-triggers reviewer after refactoring

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
