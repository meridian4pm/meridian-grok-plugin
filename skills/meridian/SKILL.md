---
name: meridian
description: Use when the user asks about their Meridian projects, risks, milestones, change requests, status reports, stakeholders, retrospectives, access reviews, team, timesheets or time tracking, or wants to log time, start or stop a timer, save a status report, or report a problem with Meridian itself. Covers which Meridian MCP tool to call and the rules for writing safely.
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
| Is my timer running, and for how long? | `get_active_timer` |
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

## Running the timer

`start_timer`, `update_active_timer`, `stop_timer` and `discard_timer` control the same timer the
user sees in Meridian's header. They need the same write access and "log time" permission as time
entries. Only one timer runs at a time.

1. Call `get_active_timer` first, every time, so you act on the timer that is really running.
2. `start_timer` is refused if a timer is already running: ask the user whether to stop or discard
   it first. Project, task and description are optional at start, but send them if you know them.
   It needs a fresh UUID v4 `idempotencyKey`.
3. `update_active_timer` changes the project, task or description without stopping. It needs the
   `timerId` and `version` (as `expectedVersion`) from `get_active_timer`.
4. `stop_timer` saves the timer as a time entry. It needs `timerId`, `expectedVersion` and a fresh
   UUID v4 `idempotencyKey`, and the entry must end up with a project and a description: ask the
   user for any that are missing. Confirm with the user before calling it with `dryRun: false`.
5. If `runningOver24h` is true the timer was probably forgotten. Ask the user when they really
   stopped and send that as `endedAt`, or discard it and log the time with `create_time_entry`.
6. `discard_timer` throws the elapsed time away and cannot be undone. Prefer `stop_timer` if the
   user did any work, and always confirm before calling it with `dryRun: false`.

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

## Reporting a problem with Meridian

`report_meridian_issue` emails a bug report, feature request or question about Meridian itself
to the Meridian team. Use it only when the user explicitly asks to report a problem with
Meridian, never for issues or risks in their own projects and never on your own initiative.

1. Write the description from what the user told you (what they did, what they expected, what
   happened). Do not invent details.
2. Call with `dryRun: true`, show the user the category and description, and only after they
   agree call again with `dryRun: false` and a fresh UUID v4 `idempotencyKey`.
3. It needs the organization's write access, and is limited to 3 reports an hour and 10 a day
   per person.

There are no delete tools. If the user wants something removed, tell them to do it in Meridian.
