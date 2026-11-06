# Connect a harness to Edits

Use only the section for the current client. These examples configure the existing Edits server; they do not install another server or enable automation themselves. Client syntax was checked against documentation on 2026-11-06. End-to-end connections through each named client have not all been tested; see [verification.md](verification.md).

## Common prerequisites

1. Run a Mac build of Edits that exposes **Settings → Automation**. Record its current enabled state. The inspected app offered **Enable Automation Server**, Port, Endpoint, Access token, and client instructions for Claude Code, Codex, and Muse.
2. Enable automation when authorized by the task and the host's permission rules. If the user authorized a temporary test, turn it off again when that test finishes. Do not disable a server that was already enabled without a reason within the task.
3. Use the endpoint displayed in the app. The observed value was `http://127.0.0.1:19501/mcp`. The transport is **Streamable HTTP** with `Authorization: Bearer <token>` authentication, not a stdio command or legacy SSE endpoint.
4. Supply the app's token through the client's supported private secret/environment mechanism. The examples call it `EDITS_TOKEN`; this is our example name, not a required Edits variable. Make it available to the actual client process before connecting. A variable exported in one terminal may not reach an already-running desktop app.
5. Merge the example into the existing configuration; preserve other servers and settings. Select the intended user/project/profile scope. Keep actual token values out of repository files, chat, logs, shell history, and screenshots shared with others.
6. Reload/restart the client's MCP connection, inspect its discovered tools, then call `appState({})` and `getCapabilities({})` through that client. Confirm successful results. Tool listing alone is not proof that tools can execute.

The default endpoint is reachable only from the Mac itself. A remote agent's `127.0.0.1` points to that remote machine. A container may also have its own loopback. Run the MCP client on the Mac or use an explicitly authorized private connection that reaches the Mac and preserves bearer authentication. Copying this skill to a cloud agent does not create that connection; do not expose the token or port publicly to make it work.

## Codex

Merge into the active host's `~/.codex/config.toml` (or trusted project `.codex/config.toml`):

```toml
[mcp_servers.edits]
url = "http://127.0.0.1:19501/mcp"
bearer_token_env_var = "EDITS_TOKEN"
```

Make `EDITS_TOKEN` available to Codex at startup. The app's **Connect an AI client → Codex** instructions are an alternative; in the inspected build they required running both copied lines before launching Codex. Treat any copied token-bearing commands as private. Verify the server in MCP settings or `codex mcp list`, then perform the common read-only calls. Desktop, CLI, and IDE clients use the configuration for the host where they run. [Official Codex MCP documentation](https://learn.chatgpt.com/docs/extend/mcp?surface=cli).

## Hermes

Merge into the **target agent's active profile** `config.yaml`, normally `~/.hermes/config.yaml`:

```yaml
mcp_servers:
  edits:
    url: "http://127.0.0.1:19501/mcp"
    headers:
      Authorization: "Bearer ${EDITS_TOKEN}"
    supports_parallel_tool_calls: false
```

Hermes expands `${EDITS_TOKEN}` from its active profile's secret scope, falling back to the process environment. Configure the secret in that profile; do not assume another agent/profile inherits it. Leave `transport: sse` unset for this Streamable HTTP server. Use `hermes mcp test edits` in the same profile to check connection/discovery, then reload MCP (`/reload-mcp` where supported) and perform the common tool calls. [Hermes MCP reference](https://hermes-agent.nousresearch.com/docs/reference/mcp-config-reference).

## Claude Code

Merge into project `.mcp.json`:

```json
{
  "mcpServers": {
    "edits": {
      "type": "http",
      "url": "http://127.0.0.1:19501/mcp",
      "headers": {"Authorization": "Bearer ${EDITS_TOKEN}"}
    }
  }
}
```

Set the environment variable before launching Claude Code. Use its normal project-server trust flow and `/mcp` or `claude mcp list`, then perform the common tool calls. The app's Claude Code connection instructions provide another setup route. This section concerns local Claude Code; cloud/Cowork sessions do not automatically inherit the Mac's endpoint or files. [Claude Code MCP documentation](https://code.claude.com/docs/en/mcp).

## Cursor

Merge into user `~/.cursor/mcp.json` or project `.cursor/mcp.json`:

```json
{
  "mcpServers": {
    "edits": {
      "url": "http://127.0.0.1:19501/mcp",
      "headers": {"Authorization": "Bearer ${env:EDITS_TOKEN}"}
    }
  }
}
```

Cursor uses `${env:NAME}` interpolation. Provide the variable to the running app; `envFile` is not supported for HTTP/SSE servers. Check the server in MCP settings and perform the common tool calls. [Cursor MCP documentation](https://cursor.com/docs/mcp).

## Gemini CLI

Merge into `~/.gemini/settings.json` or project `.gemini/settings.json`:

```json
{
  "mcpServers": {
    "edits": {
      "httpUrl": "http://127.0.0.1:19501/mcp",
      "headers": {"Authorization": "Bearer ${EDITS_TOKEN}"}
    }
  }
}
```

Use **`httpUrl`**, which selects Streamable HTTP; `url` selects SSE in the documented configuration. Set the variable in the CLI environment or its supported private environment-file mechanism. Inspect `/mcp`, then perform the common tool calls. Keep existing tool confirmation settings. [Gemini MCP documentation](https://geminicli.com/docs/tools/mcp-server/), [environment expansion and settings locations](https://geminicli.com/docs/reference/configuration/).

## OpenCode

Check the installed major version before editing `opencode.json`/`opencode.jsonc`. **V2** nests server entries under `mcp.servers`:

```json
{
  "mcp": {
    "servers": {
      "edits": {
        "type": "remote",
        "url": "http://127.0.0.1:19501/mcp",
        "oauth": false,
        "headers": {"Authorization": "Bearer {env:EDITS_TOKEN}"}
      }
    }
  }
}
```

For **V1**, place the identical `edits` object directly under `mcp`, removing the `servers` level. OpenCode calls HTTP connections `remote` even when the URL is loopback. The native Edits token uses a header, so disable OAuth for this entry. Leave protocol negotiation at its compatible default; do not force a protocol newer than the Edits server supports. Verify with `opencode mcp list`, then perform the common tool calls. [OpenCode V2 MCP documentation](https://opencode.ai/v2/docs/mcp-servers/), [V1 MCP documentation](https://opencode.ai/docs/mcp-servers/).

## Other harnesses and missing native support

Any harness capable of reading this skill and connecting to authenticated Streamable HTTP MCP can use the same native workflow. Configure the observed endpoint and bearer header using that client's documented syntax. It must support the server's negotiated protocol, tool discovery and calls; media display/file inspection improve output verification. A stdio-only client needs a separately configured, trusted transport bridge; this skill does not bundle one.

If **Settings → Automation** is absent, first establish the installed app/build and whether a supported update exposes it. A disconnected client is not evidence that Edits has no MCP server. Use the [desktop adapters](agent-adapters.md) when native automation is unavailable and the host provides computer use.

For a build genuinely without MCP, creating a custom server would require a Mac-side process using an MCP SDK, an authorized control layer (for example macOS Accessibility/screenshots when no documented app API exists), tools for project/media/timeline operations, state and error reporting, serialized edits, and authentication if served over HTTP. It would also need tests of real edits and exports against that build. Such a wrapper would inherit UI fragility and permissions; do not present it as a native Edits API. The verified build already supplies a native server, so a replacement server is unnecessary for it.
