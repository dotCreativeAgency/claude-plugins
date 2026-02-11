# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a **Claude Code plugin marketplace monorepo** that hosts and distributes Claude Code plugins. It allows users to discover and install multiple plugins from a single marketplace source.

**Repository Structure:**
```
/
├── .claude-plugin/
│   └── marketplace.json          # Marketplace metadata and plugin registry
├── plugins/
│   └── {plugin-name}/            # Individual plugin directories
│       ├── .claude-plugin/
│       │   └── plugin.json       # Plugin metadata
│       ├── agents/               # Agent definitions (*.md)
│       ├── skills/               # Skill definitions (*/SKILL.md)
│       ├── commands/             # Slash commands (*.md)
│       ├── hooks/                # Event hooks (*.md)
│       ├── CLAUDE.md             # Plugin-specific guidance
│       ├── README.md             # Plugin documentation
│       └── LICENSE               # Plugin license
├── README.md                     # Marketplace documentation
└── LICENSE                       # Marketplace license
```

## Marketplace Architecture

### Marketplace Metadata (`.claude-plugin/marketplace.json`)

The marketplace.json file defines:
- **Marketplace identity**: name, owner, description, version
- **Plugin registry**: array of plugins with installation sources

**Plugin Source Types:**
- **Relative path**: `"./plugins/{plugin-name}"` - for monorepo plugins
- **Git URL**: `"https://github.com/user/repo"` - for external plugins
- **Git URL with subdirectory**: For plugins in subdirectories of external repos

**Critical Fields:**
```json
{
  "name": "marketplace-id",
  "plugins": [
    {
      "name": "plugin-name",
      "source": "./plugins/plugin-name",  // Relative path for monorepo
      "description": "...",
      "version": "1.0.0",
      "category": "development",
      "tags": ["tag1", "tag2"]
    }
  ]
}
```

### Plugin Metadata (`.claude-plugin/plugin.json`)

Each plugin directory must contain `.claude-plugin/plugin.json`:
```json
{
  "name": "plugin-name",
  "version": "1.0.0",
  "description": "...",
  "author": { "name": "..." },
  "repository": "...",
  "homepage": "...",
  "license": "MIT",
  "keywords": [...]
}
```

**Version Constraints:**
- Must match version in marketplace.json
- Use semantic versioning (major.minor.patch)
- Update both files when incrementing version

## Development Workflows

### Adding a New Plugin to the Marketplace

1. **Create plugin directory:**
   ```bash
   mkdir -p plugins/{plugin-name}/.claude-plugin
   mkdir -p plugins/{plugin-name}/{agents,skills,commands,hooks}
   ```

2. **Create plugin metadata:**
   - Create `plugins/{plugin-name}/.claude-plugin/plugin.json`
   - Add plugin metadata (name, version, description, author, license, keywords)

3. **Register in marketplace:**
   - Edit `.claude-plugin/marketplace.json`
   - Add plugin entry to `plugins` array:
     ```json
     {
       "name": "plugin-name",
       "source": "./plugins/plugin-name",
       "description": "...",
       "version": "1.0.0",
       "author": { "name": "..." },
       "category": "development",
       "tags": ["tag1", "tag2"]
     }
     ```

4. **Create plugin components:**
   - Add agents in `agents/*.md` with YAML frontmatter
   - Add skills in `skills/{skill-name}/SKILL.md`
   - Add commands in `commands/*.md`
   - Add hooks in `hooks/*.md`

5. **Document the plugin:**
   - Create `plugins/{plugin-name}/README.md` with usage instructions
   - Create `plugins/{plugin-name}/CLAUDE.md` with development guidance
   - Create `plugins/{plugin-name}/LICENSE`

6. **Update marketplace README:**
   - Add plugin to the plugins table in root `README.md`
   - Link to plugin's README

### Modifying Existing Plugins

Plugins are organized in `plugins/` and can be modified directly:

1. **Edit plugin files:**
   - Modify agents, skills, commands, or hooks in place
   - Update `CLAUDE.md` if architecture changes
   - Update `README.md` if usage changes

2. **Update version if needed:**
   - Increment version in `.claude-plugin/plugin.json`
   - Update matching version in `.claude-plugin/marketplace.json`

3. **Commit atomically:**
   - Include both plugin changes and marketplace changes in single commit
   - Use descriptive commit message referencing plugin name

### Testing Plugins Locally

Install the marketplace locally to test plugins:

```bash
# Install from local directory
/plugin marketplace add file:///absolute/path/to/claude-plugins

# Install specific plugin
/plugin install {plugin-name}@{marketplace-id}

# Remove and reinstall after changes
/plugin uninstall {plugin-name}
/plugin install {plugin-name}@{marketplace-id}
```

## Plugin Component Guidelines

### Agents (`agents/*.md`)

Agents are autonomous subprocesses triggered by keywords or manually invoked.

**File Structure:**
```yaml
---
name: agent-name
description: "Triggering conditions with examples..."
model: opus | sonnet | haiku
color: green | yellow | red | purple | blue
---

# System prompt content
```

**Best Practices:**
- Use bilingual (EN/IT) triggering examples in description
- Include `<example>` blocks with `<commentary>` tags
- Document auto-chaining logic if agent calls other agents
- Specify MCP server dependencies
- Use clear section headers for instructions

### Skills (`skills/{skill-name}/SKILL.md`)

Skills provide specialized knowledge loaded on demand.

**File Structure:**
```yaml
---
name: skill-name
description: "When to activate this skill..."
---

# Progressive disclosure content
```

**Organization:**
- Use `## headers` for major sections
- Include reference documentation in `references/` subdirectory
- Provide CORRECT/WRONG code examples
- Document critical rules prominently

### Commands (`commands/*.md`)

Slash commands triggered with `/command-name`.

**File Structure:**
```yaml
---
name: command-name
description: "Command purpose"
args:
  - name: arg-name
    description: "Argument purpose"
    required: true
---

# Command instructions
```

### Hooks (`hooks/*.md`)

Event-driven automations triggered by Claude Code events.

**Supported Events:**
- `PreToolUse` / `PostToolUse`
- `SessionStart` / `SessionEnd`
- `UserPromptSubmit`
- `SubagentStop`
- `Stop`
- `PreCompact`
- `Notification`

## Marketplace Distribution

### Installation

Users add the marketplace with:
```bash
/plugin marketplace add dotCreativeAgency/claude-plugins
```

This reads `.claude-plugin/marketplace.json` and registers all plugins for discovery.

### Publishing Updates

1. **Make changes** to plugins in `plugins/`
2. **Update versions** in both plugin.json and marketplace.json if needed
3. **Commit and push** to master branch
4. **Users refresh** with `/plugin marketplace refresh`

The marketplace uses HTTPS URLs (not SSH) for Git operations to ensure broad compatibility.

## Current Plugins

| Plugin | Version | Description |
|--------|---------|-------------|
| [frontend-pipeline](./plugins/frontend-pipeline) | 1.0.0 | Automated frontend development pipeline with 4 agents (architect, reviewer, debugger, refactorer) for React + MUI + TypeScript projects |

Each plugin has its own CLAUDE.md with plugin-specific development guidance.
