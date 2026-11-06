# Edits UI fallback workflows

Use these when native MCP is unavailable or a needed action is better verified through the desktop UI. See [native-mcp.md](native-mcp.md) for the preferred tool workflow and [agent-adapters.md](agent-adapters.md) for desktop controls. UI library import is separate from timeline insertion; native `addMedia` combines them.

Observed on Edits for Mac 454.0.0 (1106192771). **Tested** means the operation was performed and its result observed. **Inspected** means the controls were visible but the operation was not completed. **Discovery procedure** means inspect the current UI before deciding the exact action. Do not present a discovery procedure as a tested click sequence.

## Projects and naming — tested

The home screen has **Search projects**, **Select**, **Sort** (observed value: Date Modified), **View mode**, **Settings**, **New project**, and the Projects collection.

To create a project:

1. Click **New project**. Confirm that an editor opens; the tested default name was `New project 1`.
2. Click **Project Name** (`editor.title`) in the toolbar. It changes from text to a text field with the current name selected.
3. Type the intended name and press Return. Verify that the toolbar shows the new name.

To continue an existing project, locate its name in Projects, using **Search projects** if necessary. Open the matching item and verify `editor.title` before changing anything. The existing-project open interaction was not exercised during validation; use the current item action or a normal double-click and verify the editor appears.

The inspected **File** menu exposes **New project**, **Open project**, **Import Media**, **Export project**, **Close**, and **Close All**. Prefer already-visible editor buttons when they perform the requested action. A native MCP `saveProject` tool was later discovered, but UI save/persistence semantics were not established. Do not invent a UI Save control or assume a visible editor is a durable backup. If persistence matters, use an available supported save action, reopen the project, and verify it.

## Import sources into the library — tested for a PNG

1. Open **Media** in the sidebar.
2. Click **Import** (`media.fab.import`). The native **Import Media** file picker says it accepts files or folders into the project library.
3. For a known local path, press Cmd+Shift+G, enter the absolute file path, and press Return. Verify the intended item is selected in the file picker, then click **Import**.
4. Wait for the library to refresh and verify the expected source name/thumbnail. Importing may initially return to an unchanged-looking empty library while processing finishes; do not immediately import again.

The media panel also exposes **Search** (`media.header.search`), a filter menu (`media.header.filter`, initially All), sort (`media.header.sort`, initially Name), settings (`media.header.settings`), and **Close media panel** (`media.header.close`). Check filters/search before concluding a source is absent.

Right-clicking the tested library image exposed **Rename**, **Delete**, and **Reveal in Finder**. These are library-item actions, not timeline editing operations. Do not confuse a library alias with the original source filename.

Video and audio import should be checked by the same observable criteria, but format support was not exhaustively tested. A WAV selected alongside the PNG was not verified as an imported timeline asset; do not infer audio support or successful import from that attempt.

## Add library media to the timeline — tested

1. Open the intended library source. Double-clicking the tested image displayed a source preview and **Add to timeline** (`MCEMediaPanelSourcePreview.add`). The selected thumbnail also exposed `media.cell.plus`.
2. Click **Add to timeline** once.
3. Verify a timeline clip appears, total duration becomes nonzero, and **Play** and **Export** become enabled. The tested still image created a **Photo** clip with a three-second default duration. Treat this as an observed default, not a required duration for all sources/builds.
4. Select the timeline clip to edit it. The source preview and a timeline clip are different selections.

Do not assume a new source is inserted at the desired position. Verify its track and start time. The tested empty timeline moved the playhead to the new clip's end. For multiple sources, add and check placement individually or use screenshot-grounded dragging when the actual UI supports it.

## Exact playhead position and splitting — tested

The timecode (`editor.timecode`) displays `current / total`, for example `00:01.50 / 00:03.00`. Its help reads “Click to type a time, or drag left and right to move the playhead.”

1. Select the intended timeline clip.
2. Enter timecode editing. If normal clicking does nothing and the runtime exposes the control's **Edit timecode** secondary action, invoke it. Verify that the control becomes focused/editable.
3. Select the current time text, type the requested time, and press Return. The tested input `00:01.50` produced a displayed playhead time of 1.50 seconds.
4. Verify the displayed position before cutting. It must fall inside the intended clip, not merely somewhere in the project.
5. Click **Split** (`editor.tool.split`). Confirm the original clip becomes two adjacent clips without changing total duration. The tested three-second Photo clip became two clips after a split at 1.50 seconds.

The timecode uses hundredths in the observed UI; do not interpret the `.50` as 50 frames. For requirements finer than the visible precision, inspect frame-step or other precision controls before claiming frame-accurate work.

## Trim, delete, duplicate, move — controls inspected; outcomes need verification

Visible timeline tools:

| Control | ID hint | Observed purpose |
|---|---|---|
| Select | `editor.tool.toolMode.select` | Select and scrub |
| Slip, Trim, Roll | `editor.tool.toolMode.trim` | Adjust in and out points |
| Blade | `editor.tool.toolMode.blade` | Split where you click |
| Hand | `editor.tool.toolMode.hand` | Drag to pan timeline |
| Trim start | `editor.tool.trimStart` | Trim operation; inspect the resulting boundary |
| Split | `editor.tool.split` | Split the selected clip at the playhead |
| Trim end | `editor.tool.trimEnd` | Trim operation; inspect the resulting boundary |
| Group | `editor.tool.group` | Availability depends on selection |
| Timeline settings | `editor.tool.timelineSettings` | Inspect current settings before relying on snapping/ripple behavior |
| Undo / Redo | `editor.tool.undo`, `editor.tool.redo` | Recover the immediately relevant edit and inspect its result |

For trimming, establish which portion should remain, position the playhead or use visible clip-edge handles, and apply the corresponding trim action. Verify the boundary, resulting duration, neighboring clips, and any gaps. Do not assume ripple behavior or that “Trim start” leaves all other timeline positions unchanged.

For removing a range, split its boundaries, select only that segment, and invoke the app's current deletion action. Check whether the timeline closes the gap; if not, reposition only as required by the task. When working in an existing project, distinguish removal of a timeline instance from deletion of a source or project.

For a move or reorder, return to **Select**, obtain a fresh screenshot, and drag the intended clip to the intended track/time. Verify final order and placement; timeline zoom changes coordinate-to-time mapping. For duplication, inspect the current shortcut or menu action, then check whether the new clip overlaps or follows the original.

### Keyboard presets — inspected

Open **Keyboard shortcuts** (`editor.keyboardShortcuts`). The popover includes **Keyboard Preset**, **Search actions**, and presets **Custom, Edits, CapCut, Final Cut Pro, DaVinci Resolve**. Two toolbar controls were labeled Keyboard shortcuts; choose the one with `editor.keyboardShortcuts`, not the layout control `editor.layout` whose value was Timeline.

The selected **Edits** preset displayed:

| Action | Shortcut |
|---|---|
| Split | Cmd+B |
| Trim start / end | Q / W |
| Select / Slip, Trim, Roll / Blade / Hand | A / T / B / H |
| Duplicate | Cmd+D |
| Delete | Backspace (⌫) |
| Jump to next / previous clip | Down / Up |

Check the active preset before using these. These shortcuts were read from the UI; the validation used the labeled Split button. A shortcut typed into a focused name/text/search field may edit text instead of the timeline.

## Framing, transform, color — inspector inspected

Select a timeline image/video clip first. The tested Photo inspector exposed:

- **Transform**: Keyframe, Scale, X, Y, Rotate, alignment buttons, Crop, Mirror Horizontally, Mirror Vertically.
- **Opacity**: a value/slider and Keyframe.
- **Mask** (`mask.shapePicker`): observed value None.
- **Adjust**: Save, Reset, an enable/disable toggle, Auto, and tabs Adjust, White Balance, HSL, Style, Wheels.
- **Adjust** sliders: Exposure, Clarity, Highlights, Shadows, Contrast, Brightness, Whites, Blacks, Saturation, Vibrance.
- **Utilities**: an expandable section; its contents were not explored.

Use labeled value fields/sliders when exposed and read the value back after changing it. Some parameters appear twice as a “value” and “slider”; they may represent the same setting. Do not change both automatically. For X/Y/Rotate, the observed elements were containers, so do not assume they accept direct `setValue`; inspect their editable interaction.

For crop, mask, or keyframes, open the actual tool and inspect its controls. Verify the preview at the target time, not just the inspector value. For animated settings, compare the start, intermediate, and end states. The availability of a Keyframe button alone does not establish its interpolation or timing behavior.

## Text, captions, audio, effects — discovery procedures

These sidebar entrypoints were observed. Their inner workflows must be inspected in the current app before execution.

### Text

Open **Text**, inspect the available text-creation action/styles, and add the requested overlay. Find the editable content control and enter the user's exact text. Select the resulting timeline overlay to adjust style, position, and duration using the controls actually shown. Verify spelling, line breaks, readability, and appearance/disappearance at the requested times. Do not infer that selecting a style adds an overlay until a timeline item is visible.

### Captions

Open **Captions** and inspect whether the current build offers generation, language selection, import, or manual editing. Establish which speech-bearing media is targeted before generating. Review resulting caption text, timing, and coverage; specifically check names and words that may be misrecognized. Do not promise a language, transcription model, offline processing, or successful generation from the mere presence of a Captions button.

### Audio, voiceover, sound effects

Use **Audio** or **Sound FX** for the relevant library and inspect its import/add controls. Verify the item appears on an audio track at the intended start time. Select the audio clip and use its actual inspector for level, fades, or other processing; those controls were not established by the original test.

The toolbar has **Main volume** (`mainLevelButton`), but its effect on monitoring versus rendered output was not established. Prefer a verified clip-level setting for a requested mix change and check the exported result. Do not report listening to audio if the tools only exposed a waveform or screenshot.

For **Voiceover**, inspect recording controls only when recording is part of the user's task. Follow the host's microphone permission rules. Do not record ambient audio simply to explore the panel.

### Effects, transitions, filters

Open the requested panel and inspect the available items and their application interaction. Establish the target clip or edit boundary before applying an item. Verify that the result actually appears in the timeline/inspector and preview. Check duration, intensity, and neighboring footage where controls exist. Do not assume a named mobile-app effect exists in the Mac build, or apply an arbitrary alternative without explaining the difference.

## Aspect ratio, quality, and export — entrypoints inspected

Toolbar control hints:

| Control | ID hint | Observed value |
|---|---|---|
| Aspect ratio | `editor.aspectRatio` | 9:16 |
| Quality | `editor.resolution` | 2K/60 |
| Canvas zoom level | `editor.canvasZoom` | Fit |
| Timeline zoom | accessibility label Timeline zoom | Changes with timeline state |
| Export | `editor.export` | Disabled while timeline empty; enabled after a clip was added |

1. Open the aspect-ratio control and select the requested available option. Inspect composition afterward; a ratio change can require reframing.
2. Open **Quality** and inspect the actual resolution/frame-rate choices. Select the requested available values; do not assume this guide's observed default or a fixed list of supported formats.
3. Review the timeline and preview. Resolve unintended gaps, incorrect clip order, overlay timing, and framing before export.
4. Click **Export** or the inspected **File → Export project** entrypoint. Read the resulting dialog before acting; the original validation did not establish its detailed controls, save format, or destination behavior.
5. Set any requested options offered by the actual dialog and select a clear output filename/path. Use Cmd+Shift+G in a native save dialog when appropriate. Avoid overwriting an unrelated existing file.
6. Complete export and wait for a completion result. If the app reports failure, retain its exact error and diagnose that condition rather than repeatedly starting exports.
7. Verify the resulting file exists and has nonzero size. When available, use a working media-inspection tool to check duration, dimensions, frame rate, and audio streams. Preview representative frames and check sound using an audio-capable tool when needed. If a metadata/preview tool is unavailable, state what was verified through the app and what remains unverified.

Do not call an editable project, an opened save dialog, or an export button click a delivered video. If the user only requested an editable project, stop at that requested outcome after verifying its state; exporting is not mandatory for every task.

## Recovery

| Symptom | Next step |
|---|---|
| Bundle identifier resolves to two Edits apps | Target `/Applications/Edits.app`; an installer volume may expose the duplicate. |
| Imported item is absent immediately after import | Allow processing to settle, inspect search/filter and the correct library, then verify the actual filename before retrying. |
| Export disabled | Confirm at least one source is on the timeline. A source sitting in the library is insufficient. |
| Inspector says Select a clip | Select a timeline clip, not only a media-library source. |
| Wrong thing changes after a shortcut | Check field focus and keyboard preset; inspect Undo before applying more edits. |
| Timecode click does nothing | Use exposed Edit timecode accessibility action, or a documented equivalent; confirm an editable state before typing. |
| Multiple clips share a generic Photo/Video label | Use current screenshot, track position, and thumbnail to disambiguate. |
| Element ID invalid or stale | Re-capture current state and use its new indices. |
| Library or timeline add action has uncertain result | Inspect whether the item is already present before retrying; do not create duplicates blindly. |
| A menu remains the only returned state and screenshots are unavailable | Try Escape and a fresh state, then rebind the app once. If it persists, ask the user to dismiss the menu/click the editor and resume after a fresh capture. This occurred during validation; repeated cancel/click attempts did not recover it. Do not force-quit or discard project state to continue exploration. |
| A feature/panel differs from this guide | Inspect current labels and available actions; update the plan based on evidence. Report a concrete unsupported requirement if no suitable control exists. |
