---
name: edits-mac
description: Operate Edits, an Instagram app, on macOS through its native MCP server or desktop UI to create projects, import media, edit timelines, add text and audio, work with captions and effects, and export videos. Use for editing in the installed Edits desktop app, including continuing an existing project. Excludes the mobile app and other video editors.
---

# Edits, an Instagram app — Agent Skill

Complete the user's editing request in Edits for Mac. Prefer the app's **native MCP server**; use desktop automation when MCP is unavailable or a needed operation has no usable tool. These instructions are independent of the agent harness. The technical skill identifier is `edits-mac`.

## Start here

1. Discover the current session's Edits MCP tools. Use the host's tool search or server tool list; tool prefixes differ between clients. Names such as `appState` below refer to the server's tool names, not a hardcoded host namespace.
2. If connected, read [references/native-mcp.md](references/native-mcp.md), call `appState({})` and `getCapabilities({})`, and inspect the current schemas for the operations needed. Confirm the project before any edit.
3. If tools are missing, distinguish an unconfigured client, a disabled/unreachable server, and a feature absent from this app build. Read [references/mcp-setup.md](references/mcp-setup.md) for the matching harness. Follow existing authorization and host permission rules when enabling server access or changing client configuration. Installing this skill does not connect MCP automatically.
4. If MCP cannot serve the task and Mac desktop control is available, read [references/agent-adapters.md](references/agent-adapters.md) and the relevant [UI workflows](references/workflows.md). Target `/Applications/Edits.app` when installed there; bundle ID `com.burbn.basel.mac` may also match a mounted installer copy. Capture current state before acting.
5. Establish source clips, the requested result, the target project, and any output destination from the conversation and available files. Ask only for missing information that changes the result. If neither MCP nor desktop control is available, explain the missing connection; this skill supplies instructions, not access to the Mac.

## Editing workflow

- For a simple edit, keep a compact plan: source order, retained ranges, overlays, audio, aspect ratio, and output requirements. Track source times separately from composition timeline times.
- Create and clearly name a project when requested, or identify the existing project by current name and ID. Preserve unrelated content. Use a separate project for experiments.
- Build the primary sequence first, then add the requested overlays, captions, audio, transitions, and effects. Inspect the resulting tracks, clip times, and duration after each meaningful change.
- **UI library import and timeline insertion are separate operations. Native `addMedia` imports and adds to the composition.** Check the result before adding again; do not accidentally duplicate media when switching control methods.
- Use live IDs and schemas. Rediscover state after splitting, deleting, switching projects, or moving between home and editor. Discover fonts, presets, and effects through the available catalogs instead of guessing names.
- Serialize edits to a project. After a timeout or uncertain result, read state before retrying a mutation. Avoid two agents editing the same open project concurrently.
- Review the preview at the start, end, cuts, and changed overlay boundaries. Use an audio-capable tool when sound needs verification; a waveform does not establish what was heard.

## Finish and verify

For an editable-project deliverable, verify the final state and save through a supported action. For a video deliverable, confirm the requested ratio and export settings, export to the agreed location with a distinct filename, and verify completion and the actual output file. A successful tool call or export click alone is insufficient. Check duration, dimensions, frame rate, and audio streams when media inspection is available.

Keep all work within the requested task and follow the host's permission rules. Do not manipulate private Edits databases, publish media externally, delete unrelated projects, or treat a connected MCP server as authorization for arbitrary edits. Keep the automation token out of chat, logs, screenshots shared publicly, and the skill package.

Report the project, delivered file if any, and material verification limits. The native connection and two read-only tool calls were tested; most advertised MCP editing methods still need outcome verification during use. Read [references/verification.md](references/verification.md) when evaluating coverage or extending tests.

## Install or hand off

Keep the entire `edits-mac/` directory together. [references/installation.md](references/installation.md) covers Codex, Hermes, Claude Code, Cursor, Gemini CLI, OpenCode, and manual loading. Only standard `name` and `description` frontmatter is required; `agents/openai.yaml` is optional Codex display metadata. Other harnesses can ignore it. Remote agents need an authorized connection to the Mac; their own `localhost` is not the Mac's.
