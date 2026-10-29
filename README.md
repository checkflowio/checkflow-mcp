# CheckFlow MCP Server

The official MCP server for [CheckFlow](https://checkflow.io), the checklist and process software. Connect an AI assistant to your CheckFlow workspace and ask in plain language: start a checklist, tell me what is overdue, fill in this task, turn this procedure into a template.

The server is hosted by CheckFlow. There is nothing to install or run: you add the address below to your MCP client and sign in with your CheckFlow account, or send an API key. This repository holds the documentation and the [`server.json`](server.json) published to the [MCP Registry](https://registry.modelcontextprotocol.io); the server's code is not open source.

| | |
|---|---|
| Server address (United States) | `https://api.checkflow.io/mcp` |
| Server address (Europe) | `https://api-eu.checkflow.io/mcp` |
| Transport | Streamable HTTP, stateless (JSON responses, no sessions, no event stream) |
| Authentication | Sign in with your CheckFlow account (OAuth 2.1, the MCP authorization specification), or an API key in the `X-API-KEY` header |
| Tools | 147, in eight families |
| Resources | 1, the template authoring guide |
| Plans | Every CheckFlow plan, and the 14-day free trial |
| Full documentation | [docs.checkflow.io/docs/mcp/overview](https://docs.checkflow.io/docs/mcp/overview) |

## What you can ask for

| You ask | The assistant |
|---|---|
| "What's on my plate this week?" | Reads your Tasks grid, filtered to tasks due this week. |
| "Start an Invoice Review for INV-2041, due Friday, and assign the approval to Sarah." | Creates the checklist from the Invoice Review template, sets the due date and assigns the task. |
| "Mark the bank details check as done and note that the IBAN was verified by phone." | Fills in the task's fields, adds a comment and completes the task. |
| "Which supplier onboardings have been stuck for more than two weeks?" | Searches checklists and reads their activity. |
| "Turn this procedure document into a template." | Reads the authoring guide, writes a template, validates it, fixes what the validator reports and publishes it. |
| "Add these twelve suppliers to the Suppliers Data Set." | Creates the records in one bulk call. |
| "Set up a checklist every Monday at 9 am for the weekly stock check." | Creates a schedule. |

The assistant runs on its own provider's AI, not CheckFlow's, so it uses none of your workspace's AI credits.

## Two ways to connect

- **Sign in with your CheckFlow account.** The client opens CheckFlow in your browser, you sign in and choose Allow, and the assistant then works as you. There is no key to copy or store. This is the only way to connect claude.ai, Claude Desktop's Connectors screen, the Claude mobile apps and ChatGPT.
- **An API key**, sent in the `X-API-KEY` header. This is for clients that cannot sign in, for an assistant that should act as somebody other than you or as the workspace, and for scripts.

## Before you start

1. **Use your region's address.** Workspaces in the United States use `https://api.checkflow.io/mcp`; workspaces in Europe use `https://api-eu.checkflow.io/mcp`. A request to the other region is refused with `WRONG_REGION`, and the message names the right address.
2. **Check you're allowed to connect.** An Administrator can turn off **Allow AI assistants to connect** on the AI tab of Team Management; it is on unless someone has turned it off. Members need the **AI.Assistant** permission, which new Members are given and an Administrator can grant or remove on the Users tab. Administrators don't need it. Guests can't connect an assistant.
3. **For an API key only:** an Administrator creates keys on the API & Integrations tab of Team Management. For a personal assistant, create a key that acts as you. See [Creating an API key](https://docs.checkflow.io/docs/api-v3/authentication). You don't need a key to sign in.

The examples use the United States address; use the Europe address if your workspace is in Europe.

## Signing in

1. Your browser opens a CheckFlow page that names the assistant asking to connect.
2. Sign in the way you normally do: with your email and password, with Google or Microsoft, or with your company's single sign-on. For single sign-on, enter your workspace's address (for example `acme` for acme.checkflow.io) and CheckFlow sends you to your company's sign-in page.
3. CheckFlow shows what the assistant will be able to do, which workspace it is for and the address it will send you back to. Check them, then click **Allow**.
4. Your browser returns to the assistant, which is now connected.

The assistant works as you, in the workspace you were signed in to. If you belong to more than one workspace, switch to the right one in CheckFlow before you connect; to connect another workspace too, add the server a second time and connect it while that workspace is selected. The connection lasts until you or an Administrator disconnects it, or until it goes unused for 30 days. It also stops working if you leave the workspace or are made a Guest.

## Connect a client

### Claude (web, desktop and mobile)

1. In Claude, open Settings, then Connectors, and choose **Add custom connector**.
2. Name it CheckFlow, enter your server address, and add it.
3. Click **Connect** and sign in.

A connector you add on the web is also available in the Claude Desktop and mobile apps. On a Team or Enterprise plan, an Owner may need to add the connector for your organisation before members can connect it.

### ChatGPT

ChatGPT connects to a custom MCP server in developer mode. Follow OpenAI's instructions for connecting a remote MCP server, give it your server address, and choose OAuth as the authentication. ChatGPT finds CheckFlow's sign-in page on its own, so there is nothing else to configure. Then sign in.

### Claude Code

```bash
claude mcp add --transport http checkflow https://api.checkflow.io/mcp
```

Then, in Claude Code, run `/mcp`, choose `checkflow` and **Authenticate**. Your browser opens so that you can sign in.

To connect with an API key instead, add the header:

```bash
claude mcp add --transport http checkflow https://api.checkflow.io/mcp --header "X-API-KEY: your-api-key-here"
```

Add `--scope user` to either command to make the server available in every project, rather than only the current one. Run `claude mcp list` to check that it connected, then ask Claude something like "List my CheckFlow templates."

### Cursor

Add the server to `~/.cursor/mcp.json` for every project, or to `.cursor/mcp.json` in one project. With only the address, Cursor offers to sign you in when you enable the server:

```json
{
  "mcpServers": {
    "checkflow": {
      "url": "https://api.checkflow.io/mcp"
    }
  }
}
```

To use an API key instead, add a `headers` entry:

```json
{
  "mcpServers": {
    "checkflow": {
      "url": "https://api.checkflow.io/mcp",
      "headers": {
        "X-API-KEY": "your-api-key-here"
      }
    }
  }
}
```

Open Cursor's MCP settings to check that the server shows as connected and its tools are listed.

### Visual Studio Code

Add the server to `.vscode/mcp.json` in your workspace. With only the address, VS Code asks you to sign in when the server starts:

```json
{
  "servers": {
    "checkflow": {
      "type": "http",
      "url": "https://api.checkflow.io/mcp"
    }
  }
}
```

To use an API key instead, add an `inputs` block, which makes VS Code prompt for the key once and store it securely rather than keeping it in the file:

```json
{
  "inputs": [
    {
      "type": "promptString",
      "id": "checkflow-api-key",
      "description": "CheckFlow API key",
      "password": true
    }
  ],
  "servers": {
    "checkflow": {
      "type": "http",
      "url": "https://api.checkflow.io/mcp",
      "headers": {
        "X-API-KEY": "${input:checkflow-api-key}"
      }
    }
  }
}
```

Start the server from the file or the **MCP: List Servers** command, then use the tools from Copilot Chat in agent mode.

### Claude Desktop with an API key

Claude Desktop's Connectors screen signs in, as described above. To connect Claude Desktop with an API key instead, use the [`mcp-remote`](https://www.npmjs.com/package/mcp-remote) bridge, which needs Node.js 18 or later. Open Settings, then Developer, and click Edit Config. Add a `checkflow` entry under `mcpServers`:

```json
{
  "mcpServers": {
    "checkflow": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote",
        "https://api.checkflow.io/mcp",
        "--header",
        "X-API-KEY:${CHECKFLOW_API_KEY}"
      ],
      "env": {
        "CHECKFLOW_API_KEY": "your-api-key-here"
      }
    }
  }
}
```

Save the file and restart Claude Desktop. Write the header as `X-API-KEY:${CHECKFLOW_API_KEY}` with no space after the colon: some platforms split arguments on spaces, which would break the header.

Configuration files are often synced, backed up or shared. Where a client supports environment variables or a secret prompt, use them rather than pasting an API key into the file. Signing in avoids storing a key at all.

### MCP Inspector

[MCP Inspector](https://github.com/modelcontextprotocol/inspector) lets you explore the tools and call them by hand. It is the quickest way to check a key.

1. Run `npx @modelcontextprotocol/inspector`.
2. Choose the Streamable HTTP transport and enter `https://api.checkflow.io/mcp`.
3. Add a custom header named `X-API-KEY` with your key.
4. Click Connect, then List Tools.

### Other clients

Any client works if it can connect to a remote MCP server over Streamable HTTP, and either:

- signs in using the MCP authorization specification (OAuth 2.1), or
- sends an API key with every request, in the `X-API-KEY` header or as `Authorization: Bearer your-api-key-here`.

The `Authorization: Bearer` form is for clients that can send a bearer token but not a custom header, such as the MCP connector in the Claude API and Microsoft 365 agent connectors. If a request carries both headers, `X-API-KEY` is used.

CheckFlow only sends a sign-in back to the addresses of the assistants it supports: Claude, Claude Code, ChatGPT, VS Code, Cursor, Microsoft 365 Copilot, and other desktop applications that receive the sign-in on your own computer. If a client's sign-in page says CheckFlow does not send sign-ins to its address, connect it with an API key, or ask [support](mailto:support@checkflow.io) to add the client.

More step-by-step guides are on checkflow.io: [Claude](https://checkflow.io/ai/claude) and [Copilot](https://checkflow.io/ai/copilot).

## Disconnecting an assistant

- **One you connected by signing in:** open My Profile & Settings from the menu under your name, go to the AI Assistants tab and click Disconnect next to it. Removing the connector in the assistant also ends the connection.
- **Any member's assistant, as an Administrator:** on the API & Integrations tab of Team Management, find it under Connected AI assistants and revoke it.
- **One that uses an API key:** an Administrator revokes the key on the API & Integrations tab.
- **Every assistant at once:** an Administrator turns off **Allow AI assistants to connect** on the AI tab. The REST API is not affected.

## Check the connection

Ask your assistant: "Which CheckFlow workspace are you connected to?" It answers from `get_workspace_info`, which returns the workspace's name, region, members, groups, tags and templates.

| What you see | Cause | Fix |
|---|---|---|
| The sign-in page says **This application cannot connect**. | CheckFlow doesn't recognise the client, or the address it asked to send you back to. | Remove the server from the client and add it again. If it keeps happening, connect with an API key. |
| The sign-in page says an administrator has turned off connections from AI assistants. | **Allow AI assistants to connect** is off. | Ask an Administrator to turn it back on, on the AI tab. |
| The sign-in page says Guests cannot connect AI assistants. | You are a Guest in this workspace. | Ask an Administrator to make you a Member. |
| The sign-in page says you don't have the AI.Assistant permission. | An Administrator hasn't given you AI.Assistant. | Ask an Administrator to grant it on the Users tab. |
| The assistant is connected to the wrong workspace. | You were signed in to another workspace when you connected. | Disconnect it, switch workspace in CheckFlow, and connect again. |
| The assistant asks you to connect or sign in again. | The connection was disconnected or revoked, went unused for 30 days, or you have left the workspace or been made a Guest. | Connect again. |
| An authorisation or 401 error with an API key | The key is missing, mistyped, revoked or expired, or its member can no longer be acted as. | Check the header name is exactly `X-API-KEY` and copy the key again from the API & Integrations tab. |
| `WRONG_REGION` | The workspace is in the other region. | Use the address the message gives. If you signed in, remove the connection and connect again with that address. |
| `API_KEY_ACTS_AS_WORKSPACE` | The key acts as the workspace and the tool is about a person. | Use a key that acts as a member, or sign in. |
| `RATE_LIMIT_EXCEEDED` | The workspace has used its hourly budget. | Wait for the number of seconds the message gives. |
| `AI_ASSISTANT_NOT_PERMITTED` | The member the assistant acts as doesn't have AI.Assistant. | Ask an Administrator to grant it on the Users tab. It works again within a minute, without reconnecting. |
| `MCP_DISABLED` | An Administrator has turned off **Allow AI assistants to connect**. | Ask an Administrator to turn it back on, on the AI tab. |

## Tools

147 tools in eight families. Each tool has a title, a description written for the model, a JSON Schema for its arguments and annotations that tell your client whether it only reads (57 tools), writes in a way you can undo in the app (64) or is destructive (26). Most clients ask before they run a destructive tool. Full reference: [docs.checkflow.io/docs/mcp](https://docs.checkflow.io/docs/mcp/overview).

<details>
<summary><strong>Checklists</strong> (15): list, search, create, complete, archive, share and tag checklists</summary>

`list_checklists`, `search_checklists`, `get_checklist`, `get_checklist_activity`, `list_attached_tasks`, `create_checklist`, `update_checklist`, `delete_checklist`, `set_checklist_archived`, `set_checklist_complete`, `add_checklist_tags`, `remove_checklist_tag`, `get_checklist_share`, `set_checklist_share`, `stop_sharing_checklist`

[Reference](https://docs.checkflow.io/docs/mcp/tools-checklists)
</details>

<details>
<summary><strong>Checklist tasks</strong> (27): complete tasks, fill in fields and tables, files, assignees, due dates, comments, snoozes, tags</summary>

`list_checklist_tasks`, `get_task`, `get_task_activity`, `complete_task`, `uncomplete_task`, `set_task_not_applicable`, `clear_task_not_applicable`, `set_task_assignees`, `clear_task_assignees`, `set_task_due_date`, `clear_task_due_date`, `add_task_tags`, `remove_task_tag`, `get_task_fields`, `set_task_fields`, `get_task_files`, `add_task_file`, `remove_task_file`, `get_task_table_rows`, `add_task_table_row`, `remove_task_table_row`, `set_task_table_cell`, `add_task_comment`, `remove_task_comment`, `get_task_snooze`, `set_task_snooze`, `end_task_snooze`

[Reference](https://docs.checkflow.io/docs/mcp/tools-tasks)
</details>

<details>
<summary><strong>Standalone tasks</strong> (26): raise and manage tasks outside checklists, with sub-tasks, comments and files</summary>

`raise_task`, `get_standalone_task`, `update_standalone_task`, `delete_standalone_task`, `complete_standalone_task`, `uncomplete_standalone_task`, `set_standalone_task_not_applicable`, `clear_standalone_task_not_applicable`, `get_standalone_task_activity`, `add_standalone_task_tags`, `remove_standalone_task_tag`, `get_standalone_task_comments`, `add_standalone_task_comment`, `remove_standalone_task_comment`, `get_standalone_task_files`, `add_standalone_task_file`, `remove_standalone_task_file`, `get_standalone_task_sub_tasks`, `add_standalone_task_sub_tasks`, `update_standalone_task_sub_task`, `remove_standalone_task_sub_task`, `complete_standalone_task_sub_task`, `uncomplete_standalone_task_sub_task`, `get_standalone_task_snooze`, `set_standalone_task_snooze`, `end_standalone_task_snooze`

[Reference](https://docs.checkflow.io/docs/mcp/tools-standalone-tasks)
</details>

<details>
<summary><strong>Workspace, people and Tasks grid</strong> (12): members, groups, tags, the Tasks grid and its saved views</summary>

`list_tags`, `list_members`, `get_member`, `list_groups`, `get_group`, `list_my_work`, `snooze_tasks`, `end_snooze_tasks`, `list_my_work_views`, `create_my_work_view`, `update_my_work_view`, `delete_my_work_view`

[Reference](https://docs.checkflow.io/docs/mcp/tools-workspace)
</details>

<details>
<summary><strong>Template authoring</strong> (30): the workspace summary, the schema and authoring guide, templates, versions, permissions, upgrades and drafts</summary>

`get_authoring_guide`, `get_template_schema`, `list_content_types`, `describe_content_type`, `list_condition_operators`, `list_due_date_scenarios`, `get_workspace_info`, `list_templates`, `get_template`, `list_template_versions`, `export_template`, `set_template_archived`, `list_template_permissions`, `set_template_permissions`, `validate_template`, `copy_template`, `create_template`, `create_template_version`, `list_checklist_upgrades`, `upgrade_checklists`, `create_template_draft`, `list_template_drafts`, `get_template_draft`, `replace_template_draft`, `update_template_draft`, `apply_draft_operations`, `validate_template_draft`, `commit_template`, `get_commit_status`, `delete_template_draft`

[Reference](https://docs.checkflow.io/docs/mcp/tools-templates)
</details>

<details>
<summary><strong>Data Sets</strong> (21): Data Sets, fields, records, views, and CSV import and export</summary>

`list_data_sets`, `get_data_set`, `create_data_set`, `update_data_set`, `delete_data_set`, `list_data_set_connections`, `create_data_set_field`, `update_data_set_field`, `delete_data_set_field`, `list_data_set_records`, `get_data_set_record`, `create_data_set_record`, `update_data_set_record`, `create_data_set_records`, `delete_data_set_records`, `create_data_set_view`, `update_data_set_view`, `delete_data_set_view`, `create_data_set_from_csv`, `replace_data_set_records_from_csv`, `export_data_set_csv`

[Reference](https://docs.checkflow.io/docs/mcp/tools-data-sets)
</details>

<details>
<summary><strong>Schedules</strong> (9): recurring schedules and their runs</summary>

`list_schedules`, `get_schedule`, `create_schedule`, `update_schedule`, `delete_schedule`, `list_schedule_runs`, `get_schedule_run`, `delete_schedule_run`, `create_schedule_run_checklist`

[Reference](https://docs.checkflow.io/docs/mcp/tools-schedules)
</details>

<details>
<summary><strong>Webhooks</strong> (7): webhook subscriptions, deliveries and replays</summary>

`list_webhooks`, `get_webhook`, `list_webhook_deliveries`, `replay_webhook_delivery`, `create_webhook`, `update_webhook`, `delete_webhook`

[Reference](https://docs.checkflow.io/docs/mcp/tools-webhooks)
</details>

The server also publishes one resource, `checkflow://guides/template-authoring`: a guide to writing a CheckFlow template, which clients that support resources let you attach to a conversation. It has no prompts.

## Who the assistant acts as

- **Signed in:** the assistant works as you, in the workspace you were signed in to when you connected. A connection made by signing in always acts as the person who signed in, never as anybody else or as the workspace.
- **With an API key:** every key acts as one member of the workspace, or as the workspace itself, chosen when the key is created. A key that acts as the workspace can't use the tools about a person, such as the Tasks grid and snoozing.

Either way, the assistant sees what that member can see, its changes are recorded in their name, and it is refused wherever they would be.

## Keeping it safe

A connected assistant can make real changes to your workspace, and a model can misunderstand a request.

- Sign in where your client allows it. There is then no key to store or leak, and the connection can do no more than you can.
- If you use a key, use one that acts as the least powerful member who can do the job, and avoid giving an assistant an Administrator's key unless it needs one. Give each assistant its own key, named for it, so you can see when it was last used and revoke it on its own. Treat the key as a secret.
- Keep your client's confirmation prompts on. Deleting a checklist, a standalone task, a Data Set or its records through the MCP server is as permanent as doing it in the app.
- Disconnect what you no longer use (see [Disconnecting an assistant](#disconnecting-an-assistant)).

## Limits

MCP requests share the workspace's rate limits with the REST API: 1,000 standard requests an hour and 10,000 cheap ones, shared by all of the workspace's keys and MCP connections. Listing tools, reading the schema and authoring guide and validating templates count against the cheap budget. One request to the server counts once, however many messages it batches. See [Rate limits](https://docs.checkflow.io/docs/api-v3/rate-limits).

## For developers

The server speaks JSON-RPC 2.0 over one endpoint: every message is a `POST` to `/mcp`, and every answer is JSON. `initialize` is answered for clients that need it, but `tools/list` and `tools/call` work without it. A tool returns the same data as the matching [REST API v3](https://checkflow.io/developers) request, as `structuredContent` and as a text copy. A refusal (a template that does not exist, a value that fails validation) comes back as a tool result with `isError: true` and an error code, so the model can correct itself. Writes that are not safe to repeat take an optional `idempotencyKey`.

Sign-in follows the MCP authorization specification: OAuth 2.1 with the authorisation code flow and PKCE (S256), public clients only, identified by a Client ID Metadata Document or by Dynamic Client Registration. A request with no credential gets `401` with a `WWW-Authenticate` challenge that points to the protected resource metadata (RFC 9728) at `/.well-known/oauth-protected-resource/mcp`. Access tokens last an hour, work on `/mcp` only and act as the person who signed in; refresh tokens rotate on every use. Details: [How the MCP server works](https://docs.checkflow.io/docs/mcp/how-it-works).

```http
POST https://api.checkflow.io/mcp
X-API-KEY: your-api-key-here
Content-Type: application/json

{ "jsonrpc": "2.0", "id": 1, "method": "tools/list" }
```

## Links

- [MCP server documentation](https://docs.checkflow.io/docs/mcp/overview): [connecting a client](https://docs.checkflow.io/docs/mcp/connecting), [how it works](https://docs.checkflow.io/docs/mcp/how-it-works) and the tool reference
- [CheckFlow for developers](https://checkflow.io/developers): the REST API v3, webhooks and the MCP server
- [AI in CheckFlow](https://checkflow.io/ai)
- [Pricing](https://checkflow.io/pricing): the MCP server is included on every plan and in the free trial
- [Start a free trial](https://app.checkflow.io/Signup)

## Support

Email [support@checkflow.io](mailto:support@checkflow.io). Report a security issue to [security@checkflow.io](mailto:security@checkflow.io).

Connectors named "CheckFlow" in other catalogues are built by third parties, not by CheckFlow, and are not covered by this documentation.
