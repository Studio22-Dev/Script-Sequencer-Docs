# Script Sequencer

Script-to-animatic scaffolding for real-time CG animation in Unreal Engine. Script Sequencer
builds Level Sequences from a Fountain or Final Draft (`.fdx`) screenplay and keeps them in
sync as the script changes, without overwriting anyone's work.

**Version** 0.2.1 (Early Access) · **Unreal Engine** 5.6, 5.7, 5.8 (editor only) · **Platforms** Windows, Mac  
**Download on Fab** [https://fab.com/s/d64f0cb6cf3e](https://fab.com/s/d64f0cb6cf3e) · **Support** [studio22.development@gmail.com](mailto:studio22.development@gmail.com)

## Contents
- [Quick start](#quick-start)
- [Menu](#menu)
- [Automation presets](#automation-presets)
- [Shots and cameras](#shots-and-cameras)
- [Final Draft (.fdx)](#final-draft-fdx)
- [Naming and folders](#naming-and-folders)
- [What the plug-in will and won't change](#what-the-plug-in-will-and-wont-change)
- [Working with the edit (EDL and OTIO)](#working-with-the-edit-edl-and-otio)
- [Working in a team](#working-in-a-team)
- [Tips](#tips)
- [Data contract, tags and your own tools](#data-contract-tags-and-your-own-tools)
- [Files](#files)
- [Known limitations (Early Access)](#known-limitations-early-access)
- [Support](#support)

## Quick Start
Requires Unreal Engine 5.8, 5.7 or 5.6 (editor only, Windows or Mac). On older engine
versions, the plug-in still loads but logs a warning, and some features may not work.

Installed from Fab, the plug-in lives in your engine's
`Engine/Plugins/Marketplace/ScriptSequencer/` folder. This guide, the data contract
and three sample scripts (`Sample.fountain`, `Sample_Shots.fountain`, `Sample_FDX.fdx`) are in
its `Docs` folder. Keep scripts inside your project, for example in `<Project>/Scripts/`.
It is best to make a backup of your script before importing. The plug-in records each script
by its path in the project, so the same path works on every machine.

1. Enable **Script Sequencer** in Edit > Plugins. It also enables Python Editor Script Plugin,
   Editor Scripting Utilities and Sequencer Scripting. Restart the editor.
2. Choose how much you want automated: **Tools > Script Sequencer > Automation Settings...**
   (or Project Settings > Plugins > Script Sequencer). The default preset is *Blocking*, which
   also adds empty cine cameras. Choose *Script timing* if you don't want cameras to be created.
3. Choose **Tools > Script Sequencer > Import Script (Fountain or FDX)...** and pick a
   `.fountain` or `.fdx` file (try `Sample.fountain`).
4. For a Fountain script, decide whether ID notes such as `[[id:L-4d5e6f]]` may be
   added. Yes keeps rewritten lines linked more reliably (a `.bak` copy is saved first).
   Final Draft files are never changed: Final Draft 13 keeps its own IDs (see *Final Draft* below).
5. Open `/Game/Script/<Episode>/<ScriptName>_Master` (`<Episode>` is the title page's
   `Episode:` value, or the script name if there isn't one). Its **Shots** track holds one
   section per scene (`sc010`, `sc020`...), each its own Level Sequence with an empty
   camera (`CAM_sc010`) and its camera cut. Takes, EDL export/import, and Movie Render Queue
   work like any Unreal master sequence.
6. Keep writing. Depending on the sync mode, saves update Sequencer straight away, show
   you the changes first, or wait until you use **Sync All Scripts Now**.

## Menu
- **Import Script (Fountain or FDX)...** — first import, or re-link a script.
- **Sync All Scripts Now** — apply every script's changes, including ones left pending.
- **Take Sync Ownership** — make your editor the one that syncs every script on save.
- **Automation Settings...** — project-wide defaults.
- **Stop Watching Scripts** — pause live updates, including after a restart.
- **Resume Watching Scripts** — turn live updates back on.
- **About Script Sequencer...** — the plug-in, data contract and engine versions.

## Automation Presets
Teams: commit your project's Config/DefaultEditor.ini so everyone shares those settings.

| Preset | Result | Timing |
|---|---|---|
| Structure only | Shot sections and scene heading markers are created per scene | Existing timing is held |
| Script timing | A marker per line and silent scratch audio per dialogue line is also created | Estimated from the script, ripples |
| Blocking *(default)* | Empty cine cameras and camera cuts are also created | Estimated, ripples |
| Custom | Any combination of the settings | Your choice |

**Sync** decides when changes reach the Sequencer: *Apply on every save* (default) sends changes to the
Sequencer automatically every time your script is saved, *Show changes and ask before applying* shows
a window in Unreal after every script save, and *Only when I choose Sync* requires you to use
**Tools > Sync All Scripts Now**.

By using Python from the Unreal Output Log console, you can change the same settings (replacing `<ScriptName>`):
For example: `import script_sequencer.editor_entry as ss, ss.configure("<ScriptName>", preset="blocking")`
or `ss.configure("<ScriptName>", timing="hold", sync="review")`.

**When an editor takes over scene timing**, switch to `Timing: Hold existing timing` in the plug-in Automation
settings, or use `timing="hold"` in the Python console. The script then only updates labels and adds new material, never retiming what exists.

Placeholder timing lengths come from word count: 2.5 words a second for dialogue (about 150 words
a minute), 3.5 for action, with gaps, lead-ins, and minimum lengths, all of which you can change under
**Placeholder Timing** on the settings page.

## Shots and Cameras
- A shot note on its own line, such as `[[SHOT: MCU Anna, slow push]]`, starts a new shot. In Final Draft,
  a **Shot** paragraph does the same.
- With *Structure: one shot section per [[SHOT:]] note* (chosen at import), each note becomes its own shot
  sequence (`sc010_sh0010`...).
- With cameras on (the default *Blocking* preset), the plug-in adds an **empty** Cine Camera per shot note,
  with camera cuts, a sequence with no notes gets one camera for its whole length. *Script timing* and
  *Structure only* add no cameras, and turning cameras off later removes the generated cuts but keeps the
  cameras. A shot size at the start of the note (EWS, WS, MS, MCU, CU, ECU...) sets a starting focal length.
  It never places, animates, moves or deletes a camera.

## Final Draft (.fdx)
- Scene numbers become codes: scene `12` is `sc012`, an inserted `12A` becomes `sc012A`.
- Dialogue, action, character extensions such as `(V.O.)`, dual dialogue and **Shot** paragraphs are read.
  Parentheticals, transitions, cast lists and act breaks are skipped. Revision marks are ignored.
- The file is never changed. Final Draft 13 gives every paragraph an ID of its own and keeps it when you
  edit the text. Script Sequencer follows each line by that ID, so a line rewritten beyond recognition
  keeps its scratch audio and marker, as in a Fountain script with ID notes. A pasted copy of a line is a
  new line.
- Files without these IDs (saved by an older Final Draft version or another app) are matched by scene
  number and by how similar the text is, so a line rewritten beyond recognition can count as a new line.
  Opening and saving the script in Final Draft 13 adds the IDs. Anything you've taken ownership of is
  still kept.
- **If an .fdx file won't import**, open it in Final Draft and save it again, then retry. If it still fails,
  export it as Fountain from a writing app that reads .fdx files (Fade In, Highland and WriterDuet), and
  import the `.fountain` file instead.

## Naming and Folders
Names are chosen on the settings page under **Structure and Naming** and are **fixed per
script when it's imported**, so a later change to project settings never renames existing work.

| Naming preset | Sequences | Shots | Cameras |
|---|---|---|---|
| Scene / shot *(default)* | `sc010` | `sc010_sh0020` | `CAM_sc010`, `CAM_sc010_sh0020` |
| Sequence / shot | `sq010` | `sq010_sh0010` | `CAM_sq010`, `CAM_sq010_sh0010` |
| Custom | Your own prefixes and templates, below | | |

With *Custom*, names come from templates with these tokens: `{episode}` (the title page's
`Episode:` value, else the script name), `{script}` (the file name), `{scene_num}` and
`{shot_num}` (zero-padded numbers such as `012A`, `0020`), `{scene}` and `{shot}` (prefix plus
number, such as `sc012A`, `sh0020`) and, for cameras, `{unit}` (the sequence or shot the camera
belongs to). The **Root Template** (default `/Game/Script/{episode}`) is the folder for each
script, so each episode gets its own folder. Templates with unknown tokens, or that could give
two things the same name, are refused when you save the settings.

To import with a different scheme without touching project settings:
`ss.import_script("<path>/Ep02.fountain", naming_preset="sequence_shot")`. Two scripts
can't share a folder: give each its own `Episode:` value, or put `{script}` in the Root
Template.

Each script in a project also needs its own file name, ignoring the extension: `Ep01.fountain`
and `Ep01.fdx` can't both be imported, because the plug-in tells scripts apart by name.
Importing the second one is refused with a message, rename one of them first.

To change the structure or naming of a script that's already imported, import it again into
a new folder, existing sequences are never renamed.

## The report window

It's on by default: Project Settings > Plugins > Script Sequencer > Report >
Report window, or for one script ss.configure("Episode01", report_window=False)
to turn it off. After a script save, Script Sequencer lists what changed, grouped by scene. A change
that needs a decision has a dropdown:

| Change | Choices |
|---|---|
| A scene's dialogue no longer fits its shot (timing on hold) | Ripple later shots · Extend the shot · Ignore |
| A scene's dialogue now needs less time than its shot (timing on hold) | Shorten and Ripple · Keep |
| A line you edited by hand was cut from the script | Remove anyway · Keep |
| The script changed under audio or a marker you edited | Reset to script · Keep |

- **Ask first** (sync="review"): the window opens before anything changes.
  *Apply* uses your choices, *Not now* changes nothing.
- **Apply on every save**: the changes apply, and the window opens only when
  something new needs your decision. Choices take effect straight away.
- Every change shows its dropdown each time. To skip changes you've already
  answered with Ignore, Keep or Dismiss, turn on Report > Remember choices:
  you're then asked again only if the scene or line changes.
- Ripple and Extend only move what Script Sequencer generated. A shot you
  moved or trimmed, or one placed by an EDL or OTIO import, never moves.

## What The Plug-in Edits
- It changes only markers, scratch audio sections, shot sections and camera cuts it created.
- If you move, edit, replace or delete one of those, it becomes **owned** by you and is never changed
  again. Editing any camera cut in a sequence hands that sequence's cuts to you. The report, found in
  Unreal's Output Log, tells you when the script and owned content disagree.
- Names (`sc010`, `CAM_sc010_sh0020`) are assigned once and never change. Script scene numbers
  (`#12#`, `#12A#`) are used when present, inserted scenes get in-between numbers when possible (`sc015`).
- To hand an owned item back, use `import script_sequencer.editor_entry as ss,`
  `ss.reset_items("<ScriptName>", ["L-xxxxxx"])`, replacing ScriptName and using the item's ID,
  including `L-`.
- Cutting a scene removes its shot section from the master but keeps its Level Sequence.
- Undo doesn't reverse a sync, because the assets, the sync record, and the change journal are
  saved together. To go back, revert the script and save it, or revert the assets and the
  sync record together in source control.

## Working With The Edit (EDL and OTIO)
Three things shape the master sequence, and each has one job:

- **The script decides what exists**: scenes, lines and shot notes.
- **The edit decides when and how long**: where each shot section sits on the Shots track.
- **Ownership keeps them apart**: anything someone edits is never changed again by the plug-in
  (unless that item is handed back).

In practice:
- Import an EDL onto the Shots track, or edit the sections by hand. Retimed and moved
  sections are kept exactly where the edit places them. Later script changes flow around them.
- A section the edit removed is cut in edit. Its Level Sequence and everything inside it is kept.
  Place it back on the Shots track to bring it back.
- If a Level Sequence already exists where the plug-in would create one (for example
  `/Game/Script/<Episode>/Sequences/sc020`, made by an OpenTimelineIO import), it is
  **adopted**. It keeps the editor's section, and only line markers and scratch audio are added
  inside it. Point your OTIO import hooks' `unreal.sub_sequence` paths at those names.
- Every report has an **Edit vs script** list: scenes with no section on the Shots track, and sections
  with no scene in the script (edit-only shots). Neither is changed.

## Working In A Team
Several people can open the same project through source control (Perforce, Git LFS 2, or any provider
set up under **Revision Control** in the editor's toolbar). A single Multi-User session can also be joined.
One editor per script turns its saves into Sequencer changes:

- **The sync owner.** The editor that imports a script owns it, recorded as `user@machine`
  in the script's sync record. Only the owner's editor watches the script and syncs on save, so
  in a Multi-User session each change is made once and reaches everyone.
- **Everyone else** sees "synced automatically by …" in the Output Log. **Sync All Scripts Now**
  still works for them, but asks first. **Take Sync Ownership** moves a script to your
  editor, ask the current owner to submit their sync record first, so you get the latest version.
- **Check-out before changes.** With source control on, each sync first checks out the
  sync record, its change journal and every asset it's about to change, in one call. New
  sequences and scratch sounds are marked for add after they're saved.
- **Held files wait.** If someone else has the sync record checked out, or it's out of date, or
  source control can't be reached, nothing changes and the sync tells you why. If someone
  holds a generated sequence or sound, only the changes to it wait, the report says who holds
  it, and they're applied at a later sync. Anything that person edited meanwhile is kept as
  theirs.
- **Git.** With the Git LFS 2 plug-in (UEGitPlugin) and LFS locking, a lock someone else holds
  counts as *checked out by* them, and a file with newer commits on the remote is *out of
  date*: its changes wait until you pull. The sync record and journal are text files Git doesn't
  lock, so the plug-in asks Git directly whether they're behind the remote (as of your last
  fetch or Refresh). Unreal's built-in Git provider has no check-outs, so there only the sync
  owner rule applies.
- `ss.status()` in the Python console lists each script's owner, sync mode and whether it's
  watched in your editor.

## Tips
- Your writing app may not show the `[[id:…]]` notes the plug-in adds, or a new ID it gives a
  pasted line, until you close and reopen the script. TextEdit on Mac, for one, keeps showing
  the version it opened. The notes are in the file, reopening shows them. Close the script
  before saying Yes to ID notes, too, so the app doesn't save its old copy over them.

- To see what the plug-in has recorded for a script — every sequence and line with its
  range, what's owned by you, what's cut, and the last few syncs — run
  `import script_sequencer.editor_entry as ss, ss.inspect("<ScriptName>")` in the Python console.
  It only reads data, so nothing changes.

- Live updates are faster if the editor keeps working in the background: turn off
  **Edit > Editor Preferences > General > Performance > Use Less CPU when in Background**.

- Scripts edited while the editor was closed are synced about 5 seconds after startup.

## Data Contract, Tags and Your Own Tools
Script Sequencer writes what it generates in a documented file format, the **data contract**
(`Docs/DataContract.md`, a technical spec), so your own pipeline scripts and tools can read it.

- Every generated sequence and scratch Sound Wave carries `suite.*` metadata tags: the line or
  scene ID, the unit code, the character, and the slot. To filter the Content Browser by them,
  see *Content Browser filters* below.
- The sync record and change journal (see *Files*) say what each sync changed, line by line.
- Replacing scratch audio with your own sound (a recorded take, say) changes the line to become
  owned and its audio is never touched by the plug-in.

### Content Browser Filters
The plug-in registers the `suite.*` keys with the Asset Registry when the editor starts, so you can type,
for example, `suite.line=L-4d5e6f` in the Content Browser's search box. If a search finds nothing, add the keys yourself: open **Edit > Project Settings > Game > Asset Manager**, find **Asset Registry > Metadata Tags For Asset Registry**, add `suite.contract`, `suite.generator`, `suite.script`, `suite.unit`, `suite.scene`, `suite.line`, `suite.character`, `suite.slot` and `suite.take`, and restart the editor.

## Files
- Sync record: `<Project>/ScriptSequencer/<ScriptName>.sync.json`, the plug-in's record of what it generated
  from the script and where. Comparing it with Sequencer is how the plug-in knows what you've changed. Check
  it into source control with the generated assets. Only Script Sequencer writes it. Don't edit it by hand.
- Change journal: `<Project>/ScriptSequencer/<ScriptName>.changes.jsonl`, one line per sync (the last 1,000).
  Check it in with the sync record.
- Scratch audio: one silent Sound Wave of about 3 KB per dialogue line, in `/Game/Script/<Episode>/Scratch/`.
- `Docs/DataContract.md` and `Docs/suite_contract_v1.0.schema.json`: the data contract, the technical spec of
  these files.

## Known Limitations (Early Access)
- Built for virtual/CG productions in Unreal. Live-action workflows have limited support.
- Final Draft files are never written to. Lines are followed by Final Draft's own paragraph IDs (Final Draft
  13). Files without them are matched by text. Revision reports are planned for a future update.
- Scratch audio is silence: placeholder timing, not a voice. Timing is estimated from word count unless held.
- Take Recorder takes are protected as owned content. Script-driven take slates are planned for a future update.
- One editor per script syncs automatically, others sync on request. Batch import across episodes is planned.
- Structure and naming can't be changed after import without importing again into a new folder.

## Support
Email studio22.development@gmail.com with your engine version, the Output Log from the sync, and if you can, the script and its `.sync.json`.
