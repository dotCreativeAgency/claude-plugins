# Frontend Pipeline Plugin

Automated frontend development pipeline for React + MUI + TypeScript projects. Provides 4 specialized agents that work together in an automated chain, plus a comprehensive MUI/React best practices skill.

## Pipeline Flow

```
frontend-architect ──> frontend-reviewer ──> frontend-debugger (score < 80)
                                         ──> frontend-refactorer (LOC > 150)
                                         ──> DONE (score >= 80, LOC <= 150)
```

## Components

### Agents

| Agent | Model | Color | Purpose |
|-------|-------|-------|---------|
| **frontend-architect** | opus | green | Proactive React/MUI component creation |
| **frontend-reviewer** | opus | yellow | Code quality scoring (0-100) with auto-chaining |
| **frontend-debugger** | sonnet | red | Chrome DevTools-integrated debugging |
| **frontend-refactorer** | sonnet | purple | Component decomposition (150 LOC limit) |

### Skills

| Skill | Purpose |
|-------|---------|
| **mui-react-development** | MUI v7, React 19, TanStack Query v5, React Hook Form + Zod, React Router v7 patterns |

## Features

- **Proactive activation**: Agents trigger automatically based on context (bilingual EN/IT)
- **Automatic chaining**: Agents hand off to each other based on results
- **150 LOC enforcement**: Components exceeding 150 lines are automatically flagged for decomposition
- **Quality scoring**: 100-point system across Performance, Security, Maintainability, Accessibility
- **Chrome DevTools integration**: Real-time browser inspection for debugging
- **WCAG 2.1 AA compliance**: Accessibility checks built into every agent
- **Project-aware**: Agents read CLAUDE.md to adapt to project conventions

## Prerequisites

This plugin works best with the following MCP servers configured:

| MCP Server | Purpose | Required? |
|------------|---------|-----------|
| **MUI MCP** (`mcp__mui-mcp__useMuiDocs`) | MUI v7 API documentation | Recommended |
| **Context7** (`mcp__plugin_context7_context7`) | Library documentation lookup | Recommended |
| **Chrome DevTools** (`mcp__chrome-devtools__*`) | Browser inspection for debugging | For debugger agent |
| **Memory** (`mcp__memory__*`) | Review session persistence | For reviewer agent |
| **Sequential Thinking** (`mcp__sequential-thinking__*`) | Complex analysis | Recommended |
| **IDE Diagnostics** (`mcp__ide__getDiagnostics`) | TypeScript error checking | Recommended |

## Installation

### From marketplace (recommended)

```bash
# 1. Add the marketplace
/plugin marketplace add dotCreativeAgency/claude-plugins

# 2. Install the plugin
/plugin install frontend-pipeline@dotcreativeagency-plugins
```

You can choose the installation scope:
- **User** (default): available in all your projects
- **Project**: shared with the team via `.claude/settings.json`
- **Local**: personal, gitignored via `.claude/settings.local.json`

### Development

This plugin is part of the [dotCreativeAgency marketplace monorepo](https://github.com/dotCreativeAgency/claude-plugins).

To develop locally:

```bash
# Clone the marketplace repository
git clone https://github.com/dotCreativeAgency/claude-plugins.git

# Edit the plugin in place
cd claude-plugins/plugins/frontend-pipeline

# Test locally
/plugin install file:///path/to/claude-plugins/plugins/frontend-pipeline
```

## Usage

### Architect (Component Creation)

The architect activates automatically when you mention creating UI elements:

```
"Create a users table with DataGrid"
"Ho bisogno di un form per i tenant"
"Implement a confirmation dialog"
"Aggiungi una dashboard con statistiche"
```

### Reviewer (Code Review)

Triggers automatically after architect completes, or manually:

```
"Review the UserProfile component"
"Controlla il componente che ho creato"
"Do a code audit before deploy"
```

### Debugger (Bug Fixing)

Activates on errors or low review scores:

```
"TypeError: Cannot read property 'map' of undefined"
"Il login non funziona"
"@debug check the authentication flow"
```

### Refactorer (Decomposition)

Triggers when files exceed 150 LOC:

```
"This component is too long, simplify it"
"Decomponi il componente InvoiceList"
```

## Project Compatibility

This plugin is designed for projects using:
- React 18+ with TypeScript
- MUI (Material UI) v6 or v7
- MUI X (DataGrid, DatePickers)
- TanStack Query v5
- React Hook Form + Zod
- React Router v6 or v7

For best results, ensure your project has a CLAUDE.md file documenting:
- Tech stack versions
- Path aliases
- Theme configuration
- Feature module structure
- Code conventions

## License

MIT
