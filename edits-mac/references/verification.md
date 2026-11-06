# Validation coverage

Baseline: Edits for Mac **454.0.0 (1106192771)**. UI inspection began on **2026-11-05**; native MCP checks and portable setup documentation were completed on **2026-11-06**. This skill is an independent integration guide; it is not an endorsement or official release from Instagram/Meta.

## Native MCP: live connection verified

With explicit permission to temporarily enable the server, an authenticated local JSON-RPC client completed initialization and tool discovery against the app's displayed loopback endpoint. Observed results:

| Property | Value |
|---|---|
| Endpoint | `http://127.0.0.1:19501/mcp` |
| Authentication | Bearer access token from Edits Automation settings |
| Server | `metapilot-native`, version `1.0.0` |
| Negotiated protocol | `2025-06-18` |
| Capabilities | Resources; tools with list-change notifications |
| Tools discovered in editor | 274 |
| Successful calls | `appState({})`, `getCapabilities({})` |
| Capability schema | `2026.10.19-1` |

`appState` reported the editor. Both tool results reported success. Full input schemas were inspected for the operations summarized in [native-mcp.md](native-mcp.md). Catalog families and coordinate/time conventions came from `getCapabilities`.

The test did **not** execute MCP project creation, import, edits, transcription, caption generation, preview rendering, or export. Advertised tools are not evidence that those operations succeed. The test used a direct protocol client, not a session in each named agent harness.

Afterward, the server was turned off and the loopback listener's removal was verified. Temporary diagnostic processes were stopped and the token-bearing clipboard was cleared. No permanent agent MCP configuration was added. No actual token, raw app-state dump, or diagnostic helper belongs in the distributed skill.

## Desktop UI: operations performed

Through Codex's Mac computer-use interface:

- Selected `/Applications/Edits.app` after bundle-ID lookup also matched a mounted installer.
- Read Projects and About; created and named a separate **Edits Skill Test** project.
- Imported a synthetic PNG through the native file picker, opened its source preview, and used **Add to timeline**.
- Observed a three-second Photo clip, nonzero project duration, and enabled Play/Export controls.
- Inspected the selected clip's Transform, Opacity, Mask, Adjust, and Utilities controls and the Edits keyboard preset.
- Used the exposed **Edit timecode** action, entered `00:01.50`, and verified the displayed position.
- Clicked **Split** and observed two clips with unchanged three-second duration.

The pre-existing Example project was not edited. A File-menu interaction temporarily blocked desktop automation in the first run; editor access was available again in the later session. Do not treat the earlier failure as a permanent app limitation.

Text/audio insertion, export completion, project reopen/persistence, and advanced-panel edits were not completed in the UI test. Their procedures remain discovery-and-verification guidance.

## Harness compatibility

| Harness | Evidence | Still untested |
|---|---|---|
| Codex | Skill discovery on the author's host; actual UI control; current official skill/MCP docs | Native connection through a configured Codex MCP client; full edit/export |
| Hermes | Local skill-directory/profile implementation, computer-use adapter, and MCP config docs inspected | Live Hermes-driven Edits tool calls and edits |
| Claude Code | Current official skill discovery, Streamable HTTP, and token interpolation docs | Live client connection and editing |
| Cursor | Current official skill discovery, HTTP/header configuration, and interpolation docs | Live client connection and editing |
| Gemini CLI | Current official skills, `httpUrl`, and environment expansion docs | Live client connection and editing |
| OpenCode | Official skills and separate V1/V2 MCP configuration docs | Live client connection and editing |

Primary documentation links are alongside the examples in [installation.md](installation.md) and [mcp-setup.md](mcp-setup.md). The same Markdown instructions and native tool names can be reused across clients; no claim of certification or universal plug-and-play operation is made.

## Extend coverage

In the intended client, first perform initialization/discovery and the two read-only calls. To test mutations within an authorized task, use a separate project and non-personal synthetic media: import, add a title, trim/split, set the requested output options, export, inspect the actual file, and reopen to check persistence. Verify previews and audio through suitable media tools. Record the client/app version and only the operations actually completed. Keep fixtures, raw responses, credentials, and generated outputs outside the reusable package.
