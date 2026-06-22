---
name: ado-connection
description: >
  Azure DevOps connection and authentication bootstrap for the QuorumSoftware organization.
  Use this skill EVERY time the user asks anything related to Azure DevOps, ADO, work items,
  boards, sprints, iterations, pull requests, repos, wiki, pipelines, backlogs, queries,
  or any task that requires accessing the QuorumSoftware Azure DevOps org. This includes
  creating, reading, updating, or commenting on work items; searching code or wiki;
  listing projects, repos, or branches; and any other ADO operation. Even if the user's
  request seems simple (e.g., "check my work items"), always follow this skill first to
  ensure the connection is live before proceeding.
---

# Azure DevOps Connection & Authentication

## Purpose

This skill ensures Claude always establishes and verifies an authenticated connection to
the **QuorumSoftware** Azure DevOps organization before attempting any ADO operations.
Skipping this step can lead to failed tool calls, confusing errors, or stale/missing data.

## When to Use

**Always** — before the first ADO tool call in any conversation. If you haven't yet
confirmed the connection in the current conversation, do it now.

## Authentication Steps

### Step 1: Load the ADO Tools

Before any ADO operation, call `tool_search` to load the relevant Azure DevOps MCP tools.
Use a query that matches the task at hand. Common queries:

| Task                        | Suggested `tool_search` query          |
|-----------------------------|----------------------------------------|
| Work items (CRUD, query)    | `azure devops work items`              |
| Pull requests               | `azure devops pull requests`           |
| Repos / branches / code     | `azure devops repos code`              |
| Wiki                        | `azure devops wiki`                    |
| Projects / teams            | `azure devops projects teams`          |
| Search (code, wiki, items)  | `azure devops search`                  |
| Iterations / sprints        | `azure devops iterations`              |

If you need tools from multiple categories, call `tool_search` once per category.

### Step 2: Verify the Connection

After loading tools, immediately run a lightweight verification call to confirm
authentication is working:

```
azure-devops:wit_my_work_items
  project: "Quorum"
  top: 1
  type: "assignedtome"
```

This is the cheapest call that proves end-to-end auth is live. If it succeeds
(returns a result with at least an `id`), you're good — proceed with the user's request.

### Step 3: Handle Failures

If the verification call fails or returns an error:

1. **Tell the user immediately** — don't silently retry in a loop.
2. Share the error message so the user can diagnose (e.g., expired token, MCP server
   not connected, network issue).
3. Suggest the user check:
   - That the Azure DevOps MCP integration is enabled in their Claude settings / tools menu.
   - That their Azure DevOps session hasn't expired (re-authenticate if needed).
4. Do **not** attempt the user's original ADO request until auth is confirmed.

## Defaults

When calling any ADO tool, use these defaults unless the user specifies otherwise:

- **Organization**: `QuorumSoftware`
- **Project**: `Quorum` ← always use this; NEVER use `QuorumSoftware` for work items or stories
- **Expand**: `all` (when using `wit_get_work_item`)
- **Comments**: Always run `wit_list_work_item_comments` separately after retrieving a
  work item if the user needs the full discussion thread — comments are not included
  in the standard work item response even with `expand: all`.

## Important Notes

- This authentication check only needs to happen **once per conversation**. After the
  first successful verification, you can make subsequent ADO calls without re-verifying.
- If a later ADO call fails unexpectedly mid-conversation, re-run the verification step
  before retrying.
- The ADO **organization** is `QuorumSoftware`, but the **project** for all EAM work items
  and stories is always `Quorum`. These are different things — never use `QuorumSoftware`
  as the project parameter when creating or querying work items.
