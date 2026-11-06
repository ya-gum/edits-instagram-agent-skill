# Desktop fallback adapters

Read this when native MCP cannot serve the task and Mac desktop control is available. MCP setup and the preferred workflow are in [mcp-setup.md](mcp-setup.md) and [native-mcp.md](native-mcp.md). Read only the section matching the current host. The procedures in [workflows.md](workflows.md) use application labels; always follow the actual tool schema available in the session.

Use **observe → act → observe → verify**. Prefer fresh accessibility elements over coordinates. Numeric element indices are valid only for their current capture; control IDs in the workflows are search hints. Use a fresh screenshot for timeline geometry, crop handles, and canvas placement, and recalculate coordinates after layout, zoom, or window-size changes. Disambiguate generically named clips by track, position, and thumbnail. Verify the active keyboard preset and leave text fields before issuing timeline shortcuts; do not change the user's preset to match examples.

## Codex desktop: `mcp__cua_repl.js`

Initialize using the documented entrypoint:

```javascript
var edits = await cua.getApp("/Applications/Edits.app");
```

Read the returned tool documentation and state before issuing further actions. The installed path avoids the ambiguity observed when `/Volumes/Edits Installer/Edits.app` also shared the bundle ID.

Use the current numeric index from the returned state:

```javascript
// Replace each ...Index with an index from the latest applicable capture.
await edits.click(mediaButtonIndex);
await edits.getAXState();
```

Useful supported operations:

```javascript
await edits.getAXStateAndScreenshot();
await edits.getAXState({ disableDiffing: true }); // Full tree when needed.
await edits.click(index, { clickCount: 2 });
await edits.click(index, { mouseButton: "right" });
await edits.setValue(index, "value");
await edits.typeText("text");
await edits.pressKey("super+shift+g"); // Cmd+Shift+G in a native file dialog.
await edits.pressKey("Return");
await edits.scroll(index, "down", 1);
await edits.drag([startX, startY], [endX, endY]);
```

`super` is Command in this adapter. This adapter's native-app typing/key calls do not take an element index; focus the intended field first. Coordinates must come from a current screenshot. After any actions, capture the resulting state before deciding the next action.

For the timecode, use the secondary action only when it is exposed:

```javascript
await edits.performSecondaryAction(timecodeIndex, "Edit timecode");
await edits.getAXState(); // Confirm the timecode is focused/editing.
// In the next action block:
await edits.pressKey("super+a");
await edits.typeText("00:01.50");
await edits.pressKey("Return");
await edits.getAXState();
```

If the tool says Computer Use is inactive for Edits, refresh `edits.getAXState()` before acting. If the REPL session was reset, bind `edits` again with `cua.getApp`. A successful tool call alone does not confirm that an edit happened.

## Hermes: `computer_use`

These examples match the Hermes computer-use skill installed on the author's Mac at validation time. The Hermes adapter was inspected, but an Edits session was not driven through Hermes during this validation. Inspect your runtime's available schema before using the examples.

```text
computer_use(action="capture", mode="som", app="Edits")
computer_use(action="click", element=N, capture_after=True)
computer_use(action="double_click", element=N, capture_after=True)
computer_use(action="type", text="My project", capture_after=True)
computer_use(action="key", keys="return", capture_after=True)
computer_use(action="set_value", element=N, value="value", capture_after=True)
```

`N` means the current element index. `capture` with `app="Edits"` selects the target for subsequent input. Use `mode="ax"` for a text-only capture or `mode="som"` for screenshot plus indexed elements. Re-capture after UI changes; stale handles are invalid.

Hermes uses `cmd` for Command, for example `keys="cmd+shift+g"`. It uses `coordinate=[x,y]` for a screenshot-grounded click and `from_coordinate=[x,y]`, `to_coordinate=[x,y]` for a drag. Follow the runtime's existing escalation guidance if background input has no effect; verify before retrying so an add/split operation is not applied twice.

Do not translate Codex's `performSecondaryAction` into a guessed Hermes action. The inspected Hermes interface did not advertise it. For exact timecode entry, try the visible timecode's editable interaction and verify that an editor opens. If it does not, use a documented equivalent exposed by the actual runtime or report the precision limitation. Do not claim exact placement after approximate dragging.

If `computer_use` is unavailable, use native MCP when connected. For the desktop fallback, the inspected Hermes computer-use documentation provides setup/diagnostic commands `hermes computer-use install` and `hermes computer-use doctor`; use the installed host documentation and required permissions. A remote Hermes agent needs a working connection to the Mac's MCP endpoint or desktop, not just a copy of this folder.

## Other agent frameworks

Prefer native MCP; it avoids depending on a particular desktop adapter. If a UI fallback is needed, map the workflows to app selection, current screenshot/accessibility capture, element click, text input, keypress, drag, and scroll using the framework's documented APIs. Native MCP was verified; a separate public REST API, CLI, or AppleScript dictionary was not established. Do not fabricate one or manipulate private storage to bypass missing access.
