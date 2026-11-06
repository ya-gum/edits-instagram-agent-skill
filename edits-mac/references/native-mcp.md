# Native Edits MCP workflows

Read this when Edits MCP is connected. Setup is in [mcp-setup.md](mcp-setup.md). These are schema-guided procedures, not claims that every edit was executed during validation. See [verification.md](verification.md).

## Discover the live interface

The tested Mac build exposed a Streamable HTTP endpoint at `http://127.0.0.1:19501/mcp`, authenticated with a bearer token. It identified itself as `metapilot-native` version `1.0.0`, negotiated MCP `2025-06-18`, and listed 274 tools in the editor. Treat the endpoint, tool count, and schema version as observations; read the installed app's current values.

Use the harness's MCP client rather than hand-building a second client. It handles initialization, sessions, and transport. Discover tools by server and name; do not assume a particular namespace such as `mcp__edits__` is universal. Refresh discovery after a tool-list notification or screen/project change when necessary; the server advertised `tools.listChanged`.

Start with:

| Tool | Arguments | Purpose |
|---|---|---|
| `appState` | `{}` | Identify home/editor/loading state and current app context. |
| `getCapabilities` | `{}` | Read conventions, catalogs, and available recipes. |
| `projects` | `{}` | Locate a project by current name and ID. |
| `tracks` | `{}` | Inspect tracks in the editor. |
| `elements` | `{}` or supported filters | Inspect composition elements. |
| `clips` | `{}` or supported filters | Inspect timeline placement and source in/out ranges. |

Only `appState` and `getCapabilities` were invoked during native validation. The other entries were discovered in `tools/list`. For `elements` and `clips`, follow returned pagination (`nextOffset`) when present; the observed default limit was 200. Use supported field/track/range filters to limit large responses.

Consider tool descriptions, input schemas, and capabilities together. If an example conflicts with the live schema, use the live schema. Never invent a missing argument or a successful result. Read-only annotations are useful hints, not a substitute for understanding effects: the observed `transcribeClip` is a mutation and can temporarily add library media to a track.

## Units and identity

The observed capability schema `2026.10.19-1` defined:

- Times are composition timeline seconds unless a field explicitly specifies source seconds or milliseconds.
- Canvas coordinates are normalized from 0 to 1 with origin at the top left; positive Y points down. Element X/Y specify its centre.
- Rotation is degrees clockwise; colors are `#RRGGBB` or `#RRGGBBAA`.

Read these conventions again for a new build. Use actual project, element, track, and source IDs from fresh results. A source/library ID and a timeline element ID are not interchangeable. Refresh IDs after a split or replacement.

## Projects and media

1. Use `appState` and `projects` to identify the intended project. Open it with the currently required `openProject` fields, or use `newProject` for an authorized new project. Read back the resulting state before proceeding.
2. The observed `openProject` description says “by ID or name,” but its schema requires **both** `id` and `name`. Supply the matching pair from `projects`; do not guess IDs or silently discard required fields. `renameProject` likewise requires `id` and `name`.
3. Import the agreed files using paths on the **Mac running Edits**. Native `addMedia` both imports and adds media to the composition. Its observed schema requires `path` and `audioKind`. The latter had no enum or explained default: obtain its valid meaning from live documentation/capabilities or use the verified UI import workflow; do not copy an incomplete one-argument example.
4. Inspect `clips`, `tracks`, and `elements` after adding. Confirm source, order, start, duration, and track before adding more. For existing library items, inspect the live `addLibraryMedia` schema rather than importing duplicates.
5. Use the discovered `saveProject` action when completing requested project work, and verify state. Reopen to check persistence if that is part of the task; persistence was not tested in the baseline.

## Timeline edits

Read current timing first, then use the applicable tool:

| Tool | Observed required arguments | Non-obvious behavior |
|---|---|---|
| `seek` | `seconds` | The argument is `seconds`, not `time`. |
| `splitAt` | `id`, `times` | `times` is an array. Default `projectFrame` snapping uses timeline seconds. |
| `trim` | `id`, `side`, `deltaSeconds` | `side` is `left` or `right`; positive delta shortens, negative extends. |
| `move` | `id`, `toSeconds` | Use the actual timeline element and inspect neighbors afterward. |

For example, these are **argument objects** to pass to the named tools after substituting a current clip ID:

`seek`:

```json
{"seconds": 1.5}
```

`splitAt` at 1.5 seconds of the composition:

```json
{"id": "CURRENT_CLIP_ID", "times": [1.5], "snap": "projectFrame"}
```

`trim` seven seconds from the right of a ten-second clip:

```json
{"id": "CURRENT_CLIP_ID", "side": "right", "deltaSeconds": 7}
```

These illustrate inspected schemas; they were not executed as native editing tests. Ensure the requested boundary lies within the clip and verify the resulting timing. Do not assume trim/delete ripple behavior. For `splitAt`, `sourceFrameCeil` changes the interpretation to source seconds; use it only with source-based times such as shot detection. The observed tool leaves the playhead unchanged.

## Text, captions, audio, and styling

- `addText` requires `text` and `duration`; optional `atSeconds` gives placement, otherwise it uses the playhead. Read the returned element ID and inspect timing before styling.
- `addOverlay` requires `path`, `startSeconds`, `x`, `y`, and `scale`. Follow current canvas conventions and inspect the preview.
- Discover catalogs with `listCatalogKinds`, then `listCatalog`, and inspect an item with `getCatalogItem` where needed. In the observed schema, `listCatalog` requires **kind, query, category, cursor, limit**. Do not copy shorthand examples that omit required keys. Obtain valid filter/cursor values from live descriptions/results; use UI discovery if the client cannot form a valid request.
- Catalog kinds included `font`, `textPreset`, `textAnimation`, `captionStyle`, `transition`, `videoEffect`, `filter`, `lut`, `speedCurvePreset`, `emphasisStyle`, and `maskShape`. These are discovery hints, not a fixed whitelist. For `setTextFont`, the catalog's PostScript item ID is accepted in `family`; do not guess a family variant.
- `generateCaptions` requires `source`, `language`, and `curseWords` in the observed schema. Resolve their actual values and source scope before calling; inspect names, spelling, timing, and coverage afterward. Do not infer available languages or offline processing.
- `transcribeClip` requires `id`, `language`, and `mergeSplitWords` and is **not read-only**. Use it only within an authorized transcription/editing task.
- For audio placement, levels, fades, transitions, and effect intensity, discover the specific tool and its schema. Verify resulting properties and preview/listen where the client supports it. Capability presence does not prove output quality.

## Preview and export

`previewVideo` is asynchronous in the inspected build:

1. Start a bounded preview using current arguments, for example the object below. The observed maximum duration is 10 seconds and maximum frame count is 12.
2. Retain the returned `resultId`. Poll `previewVideoStatus` with lightweight arguments `{}` until completion or an explicit error; avoid busy polling and stop after a reasonable task deadline.
3. Match the completed result to the requested ID. Fetch `previewVideoFrames` with `{"resultId":"RETURNED_RESULT_ID"}` once, then render/view the returned frames using the host's media tools. Polling status does not itself show the agent the frames.

```json
{"startSeconds": 0, "durationSeconds": 3, "frameCount": 6, "includeClip": false}
```

Finished results can be evicted. If the requested result is gone, rerender once when still needed. A local video endpoint or base64 payload may require host-specific display support; do not claim to have visually checked bytes that were never displayed.

For export, inspect available settings and current state before calling `setExportSettings`. Its observed optional fields include `aspectRatio`, `resolution`, `resolutionType`, `frameRate`, `dynamicRange`, `codec`, `quality`, and `bitrate`; accepted string values were not enumerated in the manifest. Resolve choices from live capabilities/UI and verify applied values instead of assuming a preset string works.

Call `export` with the requested absolute `path`, then inspect `exportStatus` until completion or failure. On timeout, inspect status before starting another export. Check that the file exists, is nonempty, and matches the requested format/duration; preview representative frames and check audio where available. A successful export response alone does not establish these properties.

## Recovery

| Symptom | Response |
|---|---|
| Connection refused | Check app running, Automation enabled, live port, and whether client and Edits share the same host. See setup. |
| Unauthorized | Check token availability in the client process and whether it was regenerated. Keep credentials private. |
| Missing tool | Refresh discovery and inspect current screen; also check client tool filters. Availability may depend on editor state/build. |
| Invalid arguments | Compare required fields and types to the live schema; do not repeat a guessed request. |
| Mutation timeout or connection drop | Read state and determine whether it applied before retrying. |
| Unexpected result | Inspect state; use supported Undo only when it corresponds to your operation and no intervening user edit would be lost. |
| UI fallback used | Re-read MCP state before continuing; UI and MCP control the same project. |

Never modify private project files as a workaround for a missing API. If neither exposed MCP tools nor the UI can perform the requested operation, report the concrete limitation.
