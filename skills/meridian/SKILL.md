---
name: meridian
description: Use when the user asks about their Meridian projects, risks, milestones, change requests, status reports, stakeholders, retrospectives, access reviews, team, timesheets or time tracking, or wants to log time or save a status report in Meridian. Covers which Meridian MCP tool to call and the rules for writing safely.
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
| Stakeholders and their linked projects | `list_stakeholders`, `get_stakeholder` |
| Sprint retrospectives and their action items | `list_retrospectives`, `get_retrospective`, `list_retro_action_items` |
| Who has access to which software, review due dates, an entry's history | `get_access_matrix`, `get_access_review_status`, `get_access_entry_history` |
| Facts for drafting a status report (Admins, Scrum Masters) | `get_status_report_context` |
| What this connection is allowed to do | `get_my_access` |

Start with a list tool to find ids, then call the detail tool. Dates are `YYYY-MM-DD`.
`get_team_status` takes a Monday as `weekStart`. Time tools default to the user's own entries;
`scope: "ORG"` works for admins only.

If a call fails with a permission or plan error, call `get_my_access` and explain what it says
rather than retrying.

## Writing time entries

`create_time_entry` and `update_time_entry` write real data to the user's timesheet. They are
available only when the organization has turned on write access and the user kept the
"log time" permission ticked when connecting.

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

## Saving a status report

`create_status_report` publishes a report under the user's name straight away and cannot be
edited or deleted through MCP. Admins and Scrum Masters only, with the organization's write access
on.

1. Call `get_status_report_context` for the project first. Write the six sections
   (`overallSummary`, `design`, `devFE`, `devBE`, `qa`, `clarifications`) in markdown following its
   `sectionGuide`, using only facts from that result. A section with nothing to say is
   "No update this period." with `include: false`.
2. Choose `health` at or above its `minimumHealth` (GREEN < YELLOW < RED). A lower colour is refused.
3. Set `communicationType`: `INTERNAL` for the team; `EXTERNAL` is also visible to client guests,
   so use it only when the user asks for a client-facing report, and say which you chose.
4. Call with `dryRun: true`, show the user the preview, and only after they agree call again with
   `dryRun: false` and a fresh UUID v4 `idempotencyKey`.
5. `reportDate` defaults to today; it cannot be in the future, earlier than the project's latest
   report, or more than 14 days back.

There are no delete tools. If the user wants something removed, tell them to do it in Meridian.
