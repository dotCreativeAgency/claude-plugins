# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Claude Code plugin that enhances code explanations with visual diagrams, everyday analogies, step-by-step walkthroughs, and common gotchas.

**Repository Structure:**
- `/skills/explain-code/` - Code explanation skill with visual diagrams and analogies
- `/.claude-plugin/plugin.json` - Plugin metadata and configuration

## Plugin Architecture

### Skill: explain-code

Located in `/skills/explain-code/SKILL.md`.

**Purpose**: Activates when explaining how code works, teaching about a codebase, or answering "how does this work?" questions. Enforces a structured explanation format: analogy, ASCII diagram, step-by-step walkthrough, and gotcha highlight.

## Modifying the Plugin

### Adding or Updating Skills

Skill files are in `/skills/{skill-name}/SKILL.md`:

**Frontmatter Structure:**
```yaml
---
name: skill-name
description: "When to activate this skill..."
---
```

### Plugin Metadata

Edit `/.claude-plugin/plugin.json` to update:
- `version` - Semantic versioning (major.minor.patch)
- `description` - Shown in plugin marketplace
- `keywords` - For plugin discovery

## Distribution

The plugin is part of the `dotCreativeAgency/claude-plugins` marketplace monorepo:
- **Location**: `plugins/code-explainer/` within the marketplace repository
- **Installation**: Via marketplace (`/plugin marketplace add dotCreativeAgency/claude-plugins`)
