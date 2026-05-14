---
name: read-clickup-ticket
description: Pull the ClickUp ticket associated with the current git branch into context. Parses the ticket ID from the branch name and fetches the task description, checklists, comments, and activity via the ClickUp MCP tools.
user_invocable: true
---

# Read ClickUp Ticket Skill

You are helping the user pull the full ClickUp ticket associated with the current git branch into context, so the conversation can proceed with that ticket's information available.

## Workflow

### 1. Get the current branch name

Run:
```bash
git branch --show-current
```

If the result is empty (detached HEAD), report that and stop.

### 2. Extract the ClickUp ticket ID

ClickUp custom task IDs in this org follow the pattern `CU-<id>`, where `<id>` is lowercase alphanumeric (typically starts with a digit, ~8–10 chars). Example branch:

```
#CU-868javd3p-feat-Implement-a-game-state-FSM
```

The ID is `868javd3p`.

Apply these rules in order:

1. **Primary**: regex `CU-([A-Za-z0-9]+)` (case-insensitive). Take the first capture group.
2. **Fallback**: if no `CU-` prefix is present, scan the branch's dash-separated segments for one matching `^[0-9][a-z0-9]{6,}$` (starts with a digit, mixes digits and lowercase letters, ≥7 chars). If exactly one segment matches, use it.
3. **Ambiguous / not found**: if zero or multiple candidates, ask the user to provide the ticket ID directly. Do not guess.

### 3. Fetch the ticket

Use the ClickUp MCP tools available in the environment. The relevant calls are:

- `clickup_get_task` with the extracted `taskId` and `custom_task_ids: true` — returns title, status, assignees, description, checklists, custom fields, tags.
- `clickup_get_task_comments` with the same task — returns the activity/comments thread.

Pass `custom_task_ids: true` (and the workspace/team id if the tool requires it) because the IDs parsed from branch names are ClickUp **custom** task IDs, not internal task IDs.

If the user has multiple ClickUp MCP servers connected, prefer the one whose tools are namespaced under the active workspace.

### 4. Report back

After fetching, present a concise summary in this order:

1. **Title** and **status** (e.g. `[in progress]`)
2. **Assignees** and **due date** (if any)
3. **Description** — render markdown as-is, do not summarize
4. **Checklists** — each checklist with its items, marking `[x]` for completed and `[ ]` for incomplete
5. **Comments / activity** — most recent first, with author and timestamp, full text

Do not truncate. The whole point is to pull the ticket *into context* — completeness matters more than brevity here.

End with one line: `Ticket loaded. Ready for your next instruction.`

## Error handling

- **No git repo**: report and stop.
- **Detached HEAD / empty branch**: report and stop.
- **No ID matched**: ask the user for the ID instead of guessing.
- **ClickUp tool not available**: tell the user the ClickUp MCP isn't connected and link them to the connector setup; do not attempt curl fallbacks.
- **Task not found**: report the ID that was tried and ask the user to confirm it.

## Notes

- Never echo API tokens or auth headers.
- Do not modify the ticket. This skill is read-only — no status changes, no comments added, unless the user asks in a follow-up.
