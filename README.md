# CheckFlow MCP Server

The official MCP server for [CheckFlow](https://checkflow.io), the checklist and process software. Connect an AI assistant to your CheckFlow workspace and ask in plain language: start a checklist, tell me what is overdue, fill in this task, turn this procedure into a template.

The server is hosted by CheckFlow. There is nothing to install or run: you point your MCP client at the address below and send your CheckFlow API key with each request. This repository holds the documentation and the [`server.json`](server.json) published to the [MCP Registry](https://registry.modelcontextprotocol.io); the server's code is not open source.

| | |
|---|---|
| Server address (United States) | `https://api.checkflow.io/mcp` |
| Server address (Europe) | `https://api-eu.checkflow.io/mcp` |
| Transport | Streamable HTTP, stateless (JSON responses, no sessions, no event stream) |
| Authentication | `X-API-KEY` request header. There is no OAuth. |
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

## Before you start

1. **Get an API key.** An Administrator creates keys on the API & Integrations tab of the Team page in CheckFlow. For a personal assistant, create a key that acts as you, so "my tasks" means your tasks and changes are recorded in your name. See [Creating an API key](https://docs.checkflow.io/docs/api-v3/authentication).
2. **Use your region's address.** Workspaces in the United States use `https://api.checkflow.io/mcp`; workspaces in Europe use `https://api-eu.checkflow.io/mcp`. A key sent to the other region is refused with `WRONG_REGION`, and the message names the right address.
3. **Check your client.** Any MCP client works if it can connect to a remote server over Streamable HTTP and send a custom request header. A client that can only sign in with OAuth can connect through a bridge such as [`mcp-remote`](https://www.npmjs.com/package/mcp-remote), as in the Claude Desktop example below.

In the examples, replace `your-api-key-here` with your key, and use the Europe address if your workspace is in Europe. Configuration files are often synced, backed up or shared, so where a client supports environment variables or a secret prompt, use them rather than pasting the key into the file.

## Connect a client

### Claude Code

```bash
claude mcp add --transport http checkflow https://api.checkflow.io/mcp --header "X-API-KEY: your-api-key-here"
```

Add `--scope user` to make it available in every project, rather than only the current one. Run `claude mcp list` to check that it connected, then ask Claude something like "List my CheckFlow templates."

### Claude Desktop

Claude Desktop connects to a remote server with a custom header through the `mcp-remote` bridge, which needs Node.js 18 or later. Open Settings, then Developer, and click Edit Config. Add a `checkflow` entry under `mcpServers`:

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

Save the file and restart Claude Desktop. The CheckFlow tools appear in the tools menu of a new conversation. Write the header as `X-API-KEY:${CHECKFLOW_API_KEY}` with no space after the colon: some platforms split arguments on spaces, which would break the header.

### Cursor

Add the server to `~/.cursor/mcp.json` for every project, or to `.cursor/mcp.json` in one project:

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

Add the server to `.vscode/mcp.json` in your workspace. The `inputs` block makes VS Code prompt for the key once and store it securely, rather than keeping it in the file:

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

### MCP Inspector

[MCP Inspector](https://github.com/modelcontextprotocol/inspector) lets you explore the tools and call them by hand. It is the quickest way to check a key.

1. Run `npx @modelcontextprotocol/inspector`.
2. Choose the Streamable HTTP transport and enter `https://api.checkflow.io/mcp`.
3. Add a custom header named `X-API-KEY` with your key.
4. Click Connect, then List Tools.

### Other clients

Step-by-step guides for more assistants are on checkflow.io: [Claude](https://checkflow.io/ai/claude) and [Copilot](https://checkflow.io/ai/copilot) (GitHub Copilot in VS Code and the CLI, and Copilot Studio). Any other client needs only the address and the header.

## Check the connection

Ask your assistant: "Which CheckFlow workspace are you connected to?" It answers from `get_workspace_info`, which returns the workspace's name, region, members, groups, tags and templates.

| What you see | Cause | Fix |
|---|---|---|
| An authorisation or 401 error | The key is missing, mistyped, revoked or expired, or its member can no longer be acted as. | Check the header name is exactly `X-API-KEY` and copy the key again from the Team page. |
| `WRONG_REGION` | The workspace is in the other region. | Use the address the message gives. |
| `API_KEY_ACTS_AS_WORKSPACE` | The key acts as the workspace and the tool is about a person. | Use a key that acts as a member. |
| `RATE_LIMIT_EXCEEDED` | The workspace has used its hourly budget. | Wait for the number of seconds the message gives. |
| `MCP_DISABLED` | An Administrator has turned off **Allow AI assistants to connect**. | Ask an Administrator to turn it back on, on the AI tab of Team Management. |

## Tools

147 tools in eight families. Each tool has a description written for the model, a JSON Schema for its arguments and annotations that tell your client whether it only reads (57 tools), writes in a way you can undo in the app (64) or is destructive (26). Most clients ask before they run a destructive tool. Full reference: [docs.checkflow.io/docs/mcp](https://docs.checkflow.io/docs/mcp/overview).

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

Every API key acts as one member of the workspace, or as the workspace itself. The assistant works with that member's identity and access: it sees what they can see, its changes are recorded in their name, and it is refused wherever they would be. A key that acts as the workspace can't use the tools about a person, such as the Tasks grid and snoozing.

## Keeping it safe

An assistant with an API key can make real changes to your workspace, and a model can misunderstand a request.

- Use a key that acts as the least powerful member who can do the job. Avoid giving an assistant an Administrator's key unless it needs one.
- Give each assistant its own key, named for it, so you can see when it was last used and revoke it on its own.
- Keep your client's confirmation prompts on. Deleting a checklist, a standalone task, a Data Set or its records through the MCP server is as permanent as doing it in the app.
- Treat the key as a secret, and revoke it on the API & Integrations tab if you stop using the assistant or think it has been exposed.
- An Administrator can stop all assistants connecting with **Allow AI assistants to connect** on the AI tab of Team Management. The REST API is not affected.

## Limits

MCP requests share the workspace's rate limits with the REST API: 1,000 standard requests an hour and 10,000 cheap ones, shared by all of the workspace's keys and MCP connections. Listing tools, reading the schema and authoring guide and validating templates count against the cheap budget. One request to the server counts once, however many messages it batches. See [Rate limits](https://docs.checkflow.io/docs/api-v3/rate-limits).

## For developers

The server speaks JSON-RPC 2.0 over one endpoint: every message is a `POST` to `/mcp`, and every answer is JSON. `initialize` is answered for clients that need it, but `tools/list` and `tools/call` work without it. A tool returns the same data as the matching [REST API v3](https://checkflow.io/developers) request, as `structuredContent` and as a text copy. A refusal (a template that does not exist, a value that fails validation) comes back as a tool result with `isError: true` and an error code, so the model can correct itself. Writes that are not safe to repeat take an optional `idempotencyKey`. Details: [How the MCP server works](https://docs.checkflow.io/docs/mcp/how-it-works).

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
