# Install or hand off

Public title: **Edits, an Instagram app — Agent Skill**. The folder and invocation identifier remain `edits-mac`.

Copy the whole folder, including its references. It uses ordinary `SKILL.md` instructions with `name` and `description` YAML frontmatter. Optional `agents/openai.yaml` provides Codex display metadata; other harnesses can ignore it. The package has no token, bundled MCP server, or executable dependencies.

## Skill discovery locations

Paths below are destinations for the complete `edits-mac` folder, not for a single Markdown file. Choose the scope intended for the user or project and avoid duplicate installations of the same skill.

| Harness | User installation | Project installation | Source |
|---|---|---|---|
| Codex | `~/.agents/skills/edits-mac/` in current documentation; existing hosts may use `$CODEX_HOME/skills/edits-mac/` | `.agents/skills/edits-mac/` | [Codex skills](https://learn.chatgpt.com/docs/build-skills) |
| Hermes | `<active Hermes home>/skills/edits-mac/`, default `~/.hermes/skills/edits-mac/` | Use the intended agent/profile's active home | [Hermes skills](https://hermes-agent.nousresearch.com/docs/user-guide/features/skills) |
| Claude Code | `~/.claude/skills/edits-mac/` | `.claude/skills/edits-mac/` | [Claude Code skills](https://code.claude.com/docs/en/skills) |
| Cursor | `~/.cursor/skills/edits-mac/` | `.cursor/skills/edits-mac/` | [Cursor skills](https://cursor.com/docs/skills) |
| Gemini CLI | `~/.gemini/skills/edits-mac/` | `.gemini/skills/edits-mac/` | [Gemini skills](https://geminicli.com/docs/cli/skills/) |
| OpenCode | `~/.config/opencode/skills/edits-mac/` | `.opencode/skills/edits-mac/` | [OpenCode skills](https://opencode.ai/docs/skills/) |

The author's Codex host discovered this skill at `~/.codex/skills/edits-mac/`. Keep an existing working installation there; do not create a second copy just to match a newer documentation path. For another machine, consult its current skill discovery settings. Hermes profiles have separate homes; copying into the default home does not necessarily install it for every Hermes agent.

Reload skill discovery or start a new session, then confirm `edits-mac` appears. Where supported, Gemini provides `/skills reload` and `/skills list`; Claude Code exposes `/edits-mac`; Codex can invoke `$edits-mac`. Other clients can load it by name or description. If discovery fails, first check folder nesting: the destination must contain `SKILL.md` directly.

## Connect the tools separately

Follow the matching section of [mcp-setup.md](mcp-setup.md). Installing instructions does not enable Edits automation or give the harness its access token. The agent must reach the Mac's native MCP endpoint, or have a supported Mac computer-use tool for the fallback workflow.

MCP provides the primary portable interface; desktop APIs are optional, host-specific adapters. Compatibility here means shared instructions plus documented connection settings, not that all six clients have completed an editing test. See [verification.md](verification.md) for measured coverage.

## Example requests

Codex:

> Use $edits-mac to arrange these clips in this order, add the supplied title, and export to the specified folder.

Hermes or another skill-aware agent:

> Load the edits-mac skill. In Edits on my Mac, open Product Demo, remove the first two seconds, and export a new copy to my specified output folder.

Without installing:

> Read edits-mac/SKILL.md and its relevant references, then follow them for this Edits task: …

The target must be able to read supporting files. A manual handoff works even without automatic skill discovery, provided the agent has the required MCP or desktop tools. Cloud/browser chat alone does not imply access to the Mac.
