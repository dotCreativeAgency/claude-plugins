---
name: chain-enforcement
description: "Mandatory agent pipeline enforcement for frontend development. Activates when the user asks to create, build, or implement any React/MUI UI component, form, page, table, dashboard, or interface. Triggers on: (EN) 'create a component', 'build a form', 'create a page', 'add a table', 'implement', 'build', 'fix React', '/chain'; (IT) 'crea un componente', 'crea una pagina', 'aggiungi una tabella', 'implementa', 'costruisci'; (Technical) 'MUI', 'Material UI', 'React', 'DataGrid', 'header', 'form', 'dialog', 'dashboard', 'wizard', 'stepper', 'carousel', 'sidebar', 'layout'. Also activates when the user provides a multi-component implementation plan or requests any frontend feature development."
---

# Mandatory Agent Chain Enforcement

## Absolute Rule

When this skill is active, it is **FORBIDDEN** to implement React code directly in the main conversation. EVERY frontend implementation task MUST go through the specialized agent chain using the `Task` tool.

## Input Evaluation

This skill accepts the user's implementation request (via `$ARGUMENTS` if invoked as `/chain-enforcement`, or from the conversation context when auto-triggered).

**Workflow:**
1. Receive the user's request and analyze it
2. Identify discrete components/tasks to implement
3. For each component, execute the full mandatory chain (see below)
4. Report results between phases

If the user provides a multi-component plan (e.g., "Implementa UserTable, UserForm e UserDetail"), decompose it into individual components and process each through the complete chain before moving to the next.

## Mandatory Chain Sequence

For every component/task, execute this sequence:

```
1. frontend-architect   -> Creates the component (design + implementation)
2. frontend-reviewer    -> Code review with numeric score (MANDATORY)
3. frontend-debugger    -> IF score < 80 OR CRITICAL/HIGH issues -> automatic fix
4. frontend-refactorer  -> IF LOC > 150 OR maintainability < 15/25 -> decomposition
5. frontend-reviewer    -> Re-validation after debugger/refactorer (max 2 cycles)
```

## Execution Rules

### Launching Agents

- Use ALWAYS the `Task` tool with `subagent_type` matching the agent name
- Provide a DETAILED prompt with all necessary context: files involved, requirements, output from the previous agent
- DO NOT proceed to the next phase until the current agent has completed
- Each agent has preloaded skills (`mui-react-development`, `page-header-generator`) -- do NOT repeat their content in prompts

### Forbidden Actions

- Reading files and implementing React components directly without agents
- Skipping the reviewer after the architect completes
- Not launching the debugger when the reviewer gives score < 80
- Summarizing what you "would do" instead of actually launching the agent
- Implementing "simple tasks" directly because they seem trivial
- Ignoring LOC > 150 without launching the refactorer
- Proceeding to the next component before the current one completes the full chain

### Automatic Post-Review Actions

- Score < 80 -> launch `frontend-debugger` automatically
- LOC > 150 or maintainability < 15/25 -> launch `frontend-refactorer` automatically
- Score >= 80 but with MEDIUM issues -> ask the user if they want the debugger
- After debugger/refactorer -> re-launch `frontend-reviewer` for verification (max 2 cycles)

### Multi-Component Workflow

- Complete the ENTIRE chain for one component before moving to the next
- Between phases, report to the user: phase completed, score, next phase
- If a phase fails, DO NOT proceed: report the error and await instructions

## Phase Output Format

After each agent completes, report:

```
Phase N - [component name]
   Agent: [agent name] -> Completed
   Result: [brief summary]
   Score: [if reviewer: N/100]
   Next: [next agent name or "Component N+1"]
```

## Agent Skills Reference

The pipeline agents have technical skills preloaded at startup:

| Agent | Preloaded Skills |
|-------|-----------------|
| `frontend-architect` | `mui-react-development`, `page-header-generator` |
| `frontend-reviewer` | `mui-react-development`, `page-header-generator` |
| `frontend-debugger` | `mui-react-development` |
| `frontend-refactorer` | `mui-react-development` |

These skills provide MUI v7 patterns, React best practices, and Rete page standards directly to agents -- you do not need to include this knowledge in agent prompts.

## User Plan

Process the following plan respecting ALL rules above:

$ARGUMENTS
