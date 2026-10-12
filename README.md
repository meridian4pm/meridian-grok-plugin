# Meridian plugin for Grok Build

Official plugin for [Meridian](https://www.meridian4pm.com), the PMBOK-aligned project
management office platform. Connect Grok to your Meridian workspace and ask about your
projects, risks, milestones, change requests, status reports and timesheets, or log and correct
your own time entries without leaving your editor.

Maintained by Meridian (WireApps Ltd).

## What's included

| Component | Path | Purpose |
| --- | --- | --- |
| MCP server | `.mcp.json` | Connects to the hosted Meridian MCP server over HTTP |
| Skill | `skills/meridian/SKILL.md` | Tells Grok which tool fits which question, and how to write time entries safely |
| Manifests | `.grok-plugin/plugin.json`, `.claude-plugin/plugin.json` | Plugin metadata |

The plugin has no hooks, no commands, no local scripts and no dependencies.

## Requirements

- A Meridian organization on a paid plan.
- An organization admin has turned on **MCP connections** in Settings, Integrations.
- To log or edit time, the admin has also turned on **write access** for MCP connections.
- Your role is not Guest.

## Connecting

Install the plugin, then start a session. On the first call Grok opens your browser to sign in
to Meridian and shows an Allow page listing what the connection can do. Read access is always
included. If your organization allows it, the Allow page also offers "Also let it log and edit
your time entries"; untick it for a read-only connection.

You can see and disconnect every connected app in Meridian under Settings, Integrations,
Connected apps. Changing your Meridian password disconnects them all.

## Tools

Read (scope `read:core`): `list_projects`, `get_project`, `get_risk_register`,
`get_status_report`, `list_milestones`, `list_change_requests`, `list_resources`,
`list_time_entries`, `get_timesheet_summary`, `get_team_status`, `list_time_projects`,
`list_time_tags`, `get_my_access`.

Write (scope `write:time-tracking`): `create_time_entry`, `update_time_entry`. Both support a
`dryRun` preview, and the skill asks Grok to confirm with you before writing. There are no
delete tools.

Every tool is limited to your own organization and your role's permissions in Meridian. Calls
are rate limited and recorded in your organization's audit log.

## Network endpoints

All traffic goes to one host, `www.meridian4pm.com`, over HTTPS:

| Endpoint | Used for |
| --- | --- |
| `https://www.meridian4pm.com/api/mcp` | MCP requests (JSON-RPC over HTTP) |
| `https://www.meridian4pm.com/.well-known/oauth-protected-resource` | OAuth discovery |
| `https://www.meridian4pm.com/.well-known/oauth-authorization-server` | OAuth discovery |
| `https://www.meridian4pm.com/oauth/authorize` | Sign-in and Allow page (in your browser) |
| `https://www.meridian4pm.com/api/oauth/register` | Client registration (RFC 7591) |
| `https://www.meridian4pm.com/api/oauth/token` | Token exchange and refresh |
| `https://www.meridian4pm.com/api/oauth/revoke` | Token revocation |

## Credentials

The plugin ships no secrets and asks for no API keys or environment variables. Sign-in uses
OAuth 2.1 with PKCE. Grok stores the access token (valid for 1 hour) and refresh token itself.
Tokens are bound to `https://www.meridian4pm.com/api/mcp` and are revoked when you disconnect
the app in Meridian or change your password.

## Support

Email [info@meridian4pm.com](mailto:info@meridian4pm.com) or open an issue in this repository.

## License

[MIT](LICENSE)
