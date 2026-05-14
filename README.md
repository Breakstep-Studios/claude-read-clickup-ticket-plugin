# Read ClickUp Ticket Plugin for Claude Code

A Claude Code plugin that pulls the ClickUp ticket associated with the current git branch into context — description, checklists, comments, and activity.

## Installation

```
/plugin marketplace add Breakstep-Studios/claude-plugins
/plugin install read-clickup-ticket@breakstep-studios
```

## Prerequisites

You need a ClickUp MCP server connected in Claude Code. Either:

- The `clickup` connector from the product-management plugin (`/plugin install anthropics/product-management`), or
- A custom ClickUp MCP server with `clickup_get_task` and `clickup_get_task_comments` tools.

## Branch Naming Convention

The skill expects branches that include the ClickUp custom task ID, e.g.:

```
#CU-868javd3p-feat-Implement-a-game-state-FSM
CU-868javd3p-fix-null-ref-on-load
```

It will also fall back to extracting an unprefixed segment matching the ClickUp ID shape (`[0-9][a-z0-9]{6,}`) if `CU-` is not present.

## Usage

Invoke the skill in Claude Code:

```
/read-clickup-ticket
```

That's it. The skill will:

1. Read the current branch name via `git branch --show-current`
2. Extract the ClickUp ticket ID
3. Fetch the task (description, checklists, custom fields) and its comments
4. Print the full ticket contents into the conversation

If the branch name is ambiguous or the ID can't be parsed, the skill will ask you for the ID.

## What Gets Loaded

- Title and status
- Assignees and due date
- Description (full markdown)
- All checklists with completion state
- All comments / activity, most recent first
