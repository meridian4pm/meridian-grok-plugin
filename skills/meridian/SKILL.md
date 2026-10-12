---
name: meridian
description: Use when the user asks about their Meridian projects, risks, milestones, change requests, status reports, team, timesheets or time tracking, or wants to log or correct time in Meridian. Covers which Meridian MCP tool to call and the rules for writing time entries safely.
---

# Meridian

Meridian is a PMBOK-aligned project management office platform. The `meridian` MCP server
gives you the signed-in user's view of their organization. Everything is scoped to that user's
organization and role: you never see another company's data, and a tool the user's role cannot
use is simply not listed.

## Read tools

| Ask | Tool |
| --- | --- |
| "What projects do we have?", filter by status or phase | `list_projects` |
| Detail on one project | `get_project` |
| Risks for a project (status, category) | `get_risk_register` |
| Latest or historical status report, RAG health | `get_status_report` |
| Milestones and dates | `list_milestones` |
| Change requests and their decisions | `list_change_requests` |
| Who is on the team, by role | `list_resources` |
| Time logged in a date range | `list_time_entries` |
| Totals by project, tag or person | `get_timesheet_summary` |
| Who has or hasn't logged time for a week (admins) | `get_team_status` |
| Time projects, tasks and tags to log against | `list_time_projects`, `list_time_tags` |
| What this connection is allowed to do | `get_my_access` |

Start with a list tool to find ids, then call the detail tool. Dates are `YYYY-MM-DD`.
`get_team_status` takes a Monday as `weekStart`. Time tools default to the user's own entries;
`scope: "ORG"` works for admins only.

If a call fails with a permission or plan error, call `get_my_access` and explain what it says
rather than retrying.

## Writing time entries

`create_time_entry` and `update_time_entry` write real data to the user's timesheet. They are
available only when the organization has turned on write access and the user ticked "Also let
it log and edit your time entries" when connecting.

1. Look up the ids first with `list_time_projects` and `list_time_tags`. Never guess an id.
2. Call the tool with `dryRun: true` and show the user the preview.
3. Only after the user confirms, call it again with `dryRun: false`.
4. `create_time_entry` needs a fresh UUID v4 `idempotencyKey`. Reuse the same key only when
   retrying the exact same request, so a retry never creates a duplicate.
5. `update_time_entry` needs `expectedUpdatedAt` from your latest read. If it is rejected
   because the entry changed, read it again and ask the user before retrying.
6. When changing timing on an update, send `date`, `startedAt`, `endedAt` and
   `durationSeconds` together. Timestamps need an explicit UTC offset.
7. An entry is at most 24 hours, and its end time cannot be in the future.

There are no delete tools. If the user wants an entry removed, tell them to do it in Meridian.
