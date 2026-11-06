# Edits, an Instagram app — Agent Skill

A reusable agent skill for **Edits on macOS**, using the app's native MCP server with desktop UI automation as a fallback.

Use it with **Hermes, Codex, Claude Code, Cursor, Gemini CLI, or OpenCode**. The editing instructions are shared; each harness has its own connection settings. This is an independent community project, unaffiliated with Instagram or Meta.

## What it covers

- Creating and opening projects, importing media, and building timelines.
- Splitting, trimming, arranging clips, adding text, and working with captions, audio, and effects.
- Discovering the installed app's current tools, presets, and supported values.
- Reviewing previews, exporting videos, and checking actual output files.
- Recovering from stale UI state, connection failures, and uncertain tool results.

The public title is **Edits, an Instagram app — Agent Skill**. Its portable folder name and skill identifier are **`edits-mac`**.

## Get started

1. Clone this repository or download the source ZIP from GitHub:

   ```sh
   git clone https://github.com/ya-gum/edits-instagram-agent-skill.git
   ```

2. Copy the complete [`edits-mac/`](edits-mac/) folder into your harness's skill directory. See the [installation guide](edits-mac/references/installation.md) for user, project, and Hermes profile locations. The entrypoint is [`SKILL.md`](edits-mac/SKILL.md).
3. On the Mac running Edits, open **Settings → Automation** if your build provides it. Enable the server when you want an agent to connect, and use the endpoint and access token displayed by the app.
4. Follow the matching [MCP setup instructions](edits-mac/references/mcp-setup.md) for your harness. Store the token privately; the repository's configuration examples contain environment references, not credentials.
5. Reload the skill and MCP connection, then ask the agent to call `appState` and `getCapabilities` before beginning an edit.

Example request:

> Load the edits-mac skill. Open my Product Demo project in Edits, remove the first two seconds, add the supplied title, and export a new copy to the output folder I specify.

In Codex, you can invoke it with `$edits-mac`; in Claude Code, with `/edits-mac`. An agent without automatic skill discovery can read `edits-mac/SKILL.md` and its linked references directly.

## Requirements and compatibility

You need Edits installed on a Mac and either a working native MCP connection or a Mac computer-use tool. The skill contains instructions; installing it does not install Edits, enable automation, configure a client, or grant desktop access.

The verified native endpoint was `http://127.0.0.1:19501/mcp` with bearer authentication over Streamable HTTP. Use your app's displayed endpoint if it differs. A cloud agent's `localhost` is not your Mac; remote agents need an authorized private connection. Keep the port and token private.

| Harness | Included guidance |
|---|---|
| Hermes | Active-profile skill installation, MCP YAML, token expansion, optional desktop fallback |
| Codex | Skill discovery, MCP TOML, optional desktop fallback |
| Claude Code | Skill discovery and HTTP MCP configuration |
| Cursor | Skill discovery and environment-based HTTP headers |
| Gemini CLI | Skill discovery and `httpUrl` configuration |
| OpenCode | Skill discovery and separate V1/V2 MCP configuration |

Other agents can use the same instructions if they can read the files and connect to authenticated Streamable HTTP MCP. Host-specific desktop tools are optional. `agents/openai.yaml` supplies Codex display metadata and can be ignored by other clients.

## What has been tested

On Edits for Mac **454.0.0 (1106192771)**:

- A live authenticated MCP connection, initialization, discovery of **274 tools** in the editor, and successful `appState` and `getCapabilities` calls.
- Through Codex desktop control: creation and naming of a separate test project, synthetic PNG import, timeline insertion, timecode entry, and splitting a clip.
- Configuration syntax and tool argument examples checked against the captured schemas; installation and client setup checked against primary documentation.

**Full editing and export through every harness have not been tested.** Most native editing tools were discovered and documented rather than executed. The guide requires live schema checks and result verification. See the [validation coverage](edits-mac/references/verification.md) for the exact limits.

## Guide contents

| File | Purpose |
|---|---|
| [SKILL.md](edits-mac/SKILL.md) | Agent entrypoint and editing workflow |
| [Native MCP workflows](edits-mac/references/native-mcp.md) | Discovery, timing conventions, tools, previews, and export |
| [MCP setup](edits-mac/references/mcp-setup.md) | Connection examples for each harness |
| [Installation](edits-mac/references/installation.md) | Skill discovery locations and handoff examples |
| [Desktop adapters](edits-mac/references/agent-adapters.md) | Codex, Hermes, and generic UI fallback |
| [UI workflows](edits-mac/references/workflows.md) | Observed controls and desktop procedures |
| [Validation coverage](edits-mac/references/verification.md) | Tested behavior and remaining gaps |

## Contributing

Report the Edits build, harness/version, control method, intended operation, and observed result. Remove access tokens, personal media, and private project details from reports. For workflow changes, distinguish executed tests from tool discovery or documentation review, and update the relevant reference and validation notes.

## License

[MIT](LICENSE). A copy is included inside the skill folder so it stays with the skill when redistributed. Instagram, Meta, and Edits names identify the application this independent skill operates; no endorsement is implied.
