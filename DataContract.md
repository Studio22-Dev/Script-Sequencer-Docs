# Script Sequencer data contract v1.1

**This is a technical spec.** "Data contract" is the software term for a file format that several programs promise to read and write the same way. This page describes the files and asset tags Script Sequencer writes, so your own tools and pipeline scripts can read them. It grants no rights and adds no terms; your Fab license covers that.

*Version 1.1, shipped with Script Sequencer 0.3.0. 1.1 only adds optional fields, one journal `source` value and two tag keys to 1.0, so every 1.0 file is valid 1.1 and tools written for 1.0 keep working (see [Reading files from a newer minor version](#reading-files-from-a-newer-minor-version)).* `suite_contract_v1.1.schema.json` (JSON Schema 2020-12, next to this file) is the machine-readable reference for every format below; this page explains it. `suite_contract_v1.0.schema.json` ships alongside it, unchanged. Fields new in 1.1 are marked *added in 1.1*.

Where things are located:
| Item(s) | Location | Written by |
|---|---|---|
| Sync record | `<Project>/ScriptSequencer/<Script>.sync.json` | Script Sequencer only |
| Change journal | `<Project>/ScriptSequencer/<Script>.changes.jsonl` | Script Sequencer only |
| Asset tags | `suite.*` metadata on generated assets and recorded takes | Script Sequencer |
| Discovery | Python module `suite_script_sequencer` | Script Sequencer |

All of these files are checked into source control with the project. Script Sequencer checks out the sync record and journal before every sync.

## 1. Identifiers
| ID | Format | Example | Rule |
|---|---|---|---|
| Scene | `S-` + 6 lowercase hex | `S-1a2b3c` | Stable for the life of the production; never reused |
| Line | `L-` + 6 lowercase hex | `L-4d5e6f` | Stable; recorded and captured takes can be matched to lines through it |
| Shot note | `H-` + 6 lowercase hex | `H-7a8b9c` | Stable |
| Unit code | From the naming template | `sc010`, `sc010_sh0020`, `sq010_sh0010` | Assigned once, never changed |
| Script key | Project-relative script path | `Scripts/Ep01.fountain` | Identifies the sync record and journal |

## 2. Asset metadata tags
Set with `unreal.EditorAssetLibrary.set_metadata_tag` and saved with the asset ([Epic docs](https://dev.epicgames.com/documentation/en-us/unreal-engine/asset-metadata-in-unreal-engine)). Every key starts with `suite.`. Script Sequencer registers the keys with the Asset Registry, so you can filter the Content Browser by them.

| Key | Location | Value |
|---|---|---|
| `suite.contract` | Every asset Script Sequencer generates | Contract version, e.g. `1.1` |
| `suite.generator` | Same | `ScriptSequencer` |
| `suite.script` | Same | Script key |
| `suite.episode` | Same | The script's `{episode}` value. *Added in 1.1* |
| `suite.unit` | Level Sequences for a unit, and recorded takes | Unit code (for a take: the unit it was recorded into) |
| `suite.scene` | Level Sequences for a unit | Scene ID |
| `suite.line` | Scratch Sound Waves | Line ID |
| `suite.character` | Scratch Sound Waves | Character name as written in the script |
| `suite.slot` | Scratch Sound Waves | `scratch` |
| `suite.lines` | A recorded take's Level Sequence | Comma-separated line IDs the take covers, no spaces, e.g. `L-4d5e6f,L-91aa02`. *Added in 1.1* |
| `suite.take` | A recorded take's Level Sequence | Take number, e.g. `3`. *Reserved in 1.0, written from 1.1* |

A scratch Sound Wave keeps its tags after its line is cut or its slot is claimed: syncs never delete assets, because deleting an asset in Unreal closes every open asset editor. It stays until someone uses *Tools > Delete Unused Scratch Audio*. To find the sound a line plays, read `lines[].audio` in the sync record instead of searching tags.

Metadata tags are editor-only and not available at runtime; Script Sequencer is an editor plug-in. Every value is a string. Assets generated before an upgrade keep the `suite.contract` value they were tagged with until a sync regenerates them; any `1.x` value is valid. To check an asset's tags, validate them against the schema's `unitSequenceTags`, `lineAudioTags` or `takeSequenceTags`.

## 3. Names and labels inside a sequence
| Element | Format |
|---|---|
| Shot section display name | The unit code |
| Line marker label | `SCR:<line ID> <CHARACTER>: <text>` (64 characters max) |
| Scene heading marker label | `SCR:<scene ID> <heading>` |
| Dialogue audio track | `Dialogue_<CHARACTER>` |

Readers match on the `SCR:<ID>` prefix only; the rest of the label is for clarity.

*Since 0.3.0 (contract 1.1).* With **Show IDs in markers** off, the label holds only the text after `SCR:<ID> `, and `SCR:<ID>` alone is the marker's comment. A reader looks for the ID in the label first and in the comment second. Engines whose markers have no comment field always keep the ID in the label. **Marker labels** may also shorten the text, ending it in `…`, so readers shouldn't rely on the text.

## 4. The sync record
The plug-in's record of what it generated from one script and where: every line's ID, the sequence and frames it landed on, what a person has taken over (`owned`), and which editor syncs the script. Comparing this record with what's in Sequencer is how Script Sequencer tells its own work from yours.

`<Project>/ScriptSequencer/<Script>.sync.json`. **Only Script Sequencer writes it**, automatically (write a temp file, then rename). Tools that read it must ignore fields they don't know.

```json
{
  "contract_version": "1.1",
  "script": { "key": "Scripts/Ep01.fountain", "format": "fountain", "hash": "3f2c…" },
  "master": "/Game/Script/Ep01/Ep01_Master",
  "frame_rate": { "numerator": 24, "denominator": 1 },
  "sync_owner": "anna@ws-anim-04",
  "journal_seq": 42,
  "episode": "Ep01",
  "revision": { "name": "Blue", "color": "#9FC5E8", "index": 2 },
  "cast": [ { "character": "ANNA", "class": "/Game/Characters/BP_Anna.BP_Anna_C" } ],
  "units": [
    { "code": "sc010", "scene": "S-1a2b3c", "shot_note": null,
      "asset": "/Game/Script/Ep01/Sequences/sc010",
      "state": "active", "range": [0, 480], "owned": false,
      "cast": [ { "character": "ANNA", "binding": "ANNA" } ] }
  ],
  "lines": [
    { "id": "L-4d5e6f", "scene": "S-1a2b3c", "unit": "sc010", "kind": "dialogue",
      "character": "ANNA", "text": "It's been boiling for an hour.", "text_hash": "9c1e…",
      "state": "active", "range": [240, 312],
      "audio": { "slot": "scratch", "asset": "/Game/Script/Ep01/Scratch/SW_Scratch_L_4d5e6f",
                 "track": "Dialogue_ANNA", "claimed_by": null },
      "owned": false,
      "revision": "Blue",
      "captures": [ { "take": "/Game/Cinematics/Takes/2026-10-24/sc010_3", "slate": "sc010",
                      "number": 3, "recorder": "take_recorder",
                      "time": "2026-10-24T10:12:40Z", "text_hash": "9c1e…" } ],
      "changed_after_capture": false }
  ],
  "internal": { "…": "Script Sequencer's own state; not part of the contract" }
}
```

| Field | Meaning |
|---|---|
| `contract_version` | `1.x`. A reader that sees `2.0` or later should stop with a clear message. |
| `script` | `key` (script key), `format` (`fountain` or `fdx`), `hash` (SHA-256 of the file at the last sync) |
| `master` | The master Level Sequence whose Shots track holds the units |
| `frame_rate` | Display rate shared by every generated sequence, as a fraction (`24/1`, `30000/1001`) |
| `sync_owner` | `user@machine` of the one editor allowed to auto-sync the script, or null before the first import finishes |
| `journal_seq` | `seq` of the last journal entry; 0 when the journal is empty |
| `episode` | The `{episode}` value the script was imported with. *Added in 1.1* |
| `revision` | The script's current revision set, or null. `name` is always set; `color` (`#9FC5E8`) and `index` (from 1) come from Final Draft. A Fountain title page line `Revision: Blue` gives the name only. Null when the script names no revision. *Added in 1.1* |
| `cast[]` | The casting table in use: `character` (as written in the script) and `class` (a class path such as `/Game/Characters/BP_Anna.BP_Anna_C`). *Added in 1.1* |
| `units[]` | `code`, `scene`, `shot_note` (set for one unit per shot note), `asset`, `state` (`active`, `script_cut`, `edit_cut`), `range` on the master (null unless `active`), `owned` |
| `units[].cast[]` | `character` and `binding` (the binding's display name) for each character Script Sequencer bound in that unit's sequence. *Added in 1.1* |
| `lines[]` | `id`, `scene`, `unit`, `kind` (`dialogue` or `action`), `character` (empty for action), `text`, `text_hash`, `state` (`active`, or `cut` when an owned line left the script), `range` inside the unit sequence, `audio`, `owned` |
| `lines[].audio` | `slot`, `asset`, `track`, `claimed_by` (reserved; null in this version). Null when scratch audio is off; always null for action lines. |
| `lines[].revision` | Name of the revision set in which the line last changed (Final Draft), or null. *Added in 1.1* |
| `lines[].captures[]` | One entry per take that covered the line, oldest first: `take` (the take's Level Sequence), `slate`, `number`, `recorder` (`take_recorder` or `mocap_manager`), `time` (UTC), `text_hash` (the line's `text_hash` when it was recorded). A record of coverage only: the audio slot is unchanged. *Added in 1.1* |
| `lines[].changed_after_capture` | True when the line's `text_hash` differs from its newest capture's. A line with this set to true always has at least one capture. *Added in 1.1* |
| `internal` | Script Sequencer's private state; don't read it, as it can change in any release |

Ranges are `[start, end)` frames at `frame_rate`. Hashes are lowercase SHA-256; `text_hash` is taken after collapsing runs of whitespace and trimming the ends.

Every 1.1 field is optional. A field appears once the feature that writes it is in use (casting, captures, revision sets, episodes), and a script that uses none of them has the same fields as in 1.0. 0.3.0 writes the revision fields (`revision` and `lines[].revision`). The others are defined now and written from 0.4.0.

## 5. The change journal
`<Project>/ScriptSequencer/<Script>.changes.jsonl` — one JSON object per sync, appended after the sync record is saved. A reader remembers the last `seq` it handled and reads only newer lines.

```json
{"seq": 42, "time": "2026-10-12T14:03:11Z", "contract_version": "1.1", "source": "script", "episode": "Ep01",
 "revision": {"name": "Blue", "color": "#9FC5E8"},
 "lines": {"added": ["L-91aa02"], "text_changed": ["L-4d5e6f"], "removed": [], "moved": [], "retimed": ["L-5e6f70"],
           "changed_after_capture": []},
 "units": {"added": [], "removed": [], "retimed": ["sc020"]}}
{"seq": 43, "time": "2026-10-24T10:12:41Z", "contract_version": "1.1", "source": "capture", "episode": "Ep01",
 "lines": {"added": [], "text_changed": [], "removed": [], "moved": [], "retimed": [], "captured": ["L-4d5e6f"]},
 "units": {"added": [], "removed": [], "retimed": []}}
```

- `source` is `script` (a script save), `edit` (EDL or OTIO changes found on the Shots track), `mixed` (both in one sync) or `capture` (a take was recorded; no sync ran). *`capture` added in 1.1.*
- `lines.moved` means a line changed scene, unit or order; `lines.retimed` means same text, new range — for example rippled by an earlier line. Anything you place per line can be re-timed from `retimed`.
- Optional lists *added in 1.1*: `lines.captured` (lines a take covered), `lines.changed_after_capture` (captured lines whose text changed) and `lines.capture_cleared` (lines whose flag cleared, because of a new take or a dismissal). A tool that tracks recapture work only needs `lines.changed_after_capture`.
- `episode` and `revision` (`name`, optionally `color`) repeat the sync record's values at the time of the entry. *Added in 1.1.*
- `lines.claimed` and `lines.released` are optional lists the schema reserves; this version doesn't write them.
- `time` is UTC. The journal keeps its last 1,000 entries; a reader that falls further behind re-reads the sync record.

## 6. Ownership
1. **Hand edits win.** Anything someone changes is owned and never changed automatically.
2. **Script Sequencer edits only what it generated**, if nobody has touched it since.
3. **Edits from an EDL or OTIO import** are hand edits. The shot section no longer matches, so it's owned and logged in the journal with `source: "edit"` (or `"mixed"` when the same sync also applied script changes).
4. **Recorded takes are their own.** A take's tracks and audio are owned like any hand edit; `captures` and the take tags only record which lines it covers.

## 7. Discovery
A Python script or plug-in can check whether Script Sequencer is installed, and which contract versions it writes, by importing its public module:

```python
try:
    import suite_script_sequencer as ss
except ImportError:
    ss = None                      # Not installed
if ss and any(v.split(".")[0] == "1" for v in ss.contract_versions()):
    ...                            # Any 1.x: files this tool can read
```

`contract_versions()` returns `["1.0", "1.1"]`; `capabilities()` returns `["sync_record", "journal", "slots", "edit_adoption"]` plus a name for each 1.1 feature this version ships (`captures`, `casting`, `revisions`, `batch`); `plugin_version()` returns the plug-in's own version. Check for a version's major number, not for one exact version: a tool that tests for `"1.0"` alone would miss a later Script Sequencer that writes `1.2`. This is the only Script Sequencer module meant to be imported; everything else is internal and can change in any release.

## 8. Versioning
- Versions are `MAJOR.MINOR`. A minor version only adds optional fields, enum values and tag keys; a new major version (`2.0`) may change anything.
- Script Sequencer reads any `1.x` file and writes the newest version it knows. It keeps fields it doesn't recognise, at the top level of the sync record and in `units[]` and `lines[]`, so a newer version's data survives a save by an older one. Releases before 0.3.0 don't do this, so on a team, upgrade the editor that syncs each script first.
- A sync record from before contract 1.0 (the starter format) is migrated on load, keeping a `.bak`. A 1.0 record is read as it is and saved as 1.1 at its next sync.
- This page and both schemas ship in the plug-in's `Docs` folder, with the changelog below.

### Reading files from a newer minor version
- Accept any `1.x` `contract_version`. Refuse `2.0` and later with a message.
- Ignore fields you don't know. If you write the file back, keep them.
- Skip journal entries whose `source` you don't know, rather than stopping. 1.1 adds `capture`, which the 1.0 schema's list doesn't include, so a strict validator using the 1.0 schema rejects those entries. Validate against the newest schema you ship, or check `source` first.
- `revision` is an object with at least a `name`, or null.

## 9. Validating
Any JSON Schema 2020-12 validator works. The schema's root accepts a sync record or a journal entry; tag maps and discovery results are checked against their `$defs`. In Python, with `jsonschema` 4.18 or later:

```python
import json
from jsonschema import Draft202012Validator
from referencing import Registry, Resource

schema = json.load(open("Docs/suite_contract_v1.1.schema.json"))
registry = Registry().with_resource(schema["$id"], Resource.from_contents(schema))

def validator(defname=None):
    target = schema if defname is None else {"$ref": f'{schema["$id"]}#/$defs/{defname}'}
    return Draft202012Validator(target, registry=registry)

validator().validate(json.load(open("Ep01.sync.json")))   # a sync record or a journal entry
validator("lineAudioTags").validate(tags)                     # one Sound Wave's tags
validator("takeSequenceTags").validate(take_tags)             # one recorded take's tags (1.1)
```

Every 1.0 file also validates against the 1.1 schema.

## Changelog
| Version | Date | Changes |
|---|---|---|
| 1.1 | 0.3.0 | Optional fields: `episode`, `revision` and `cast[]` on the sync record; `units[].cast[]`; `lines[].revision`, `lines[].captures[]` and `lines[].changed_after_capture`. Journal: `source` value `capture`; optional `episode`, `revision` and the `lines.captured`, `lines.changed_after_capture` and `lines.capture_cleared` lists. Tags: `suite.episode`, `suite.lines`, and `suite.take` written on recorded takes. New `takeSequenceTags` definition. Script Sequencer keeps unknown fields when it saves a sync record. Every 1.0 file is valid 1.1. |
| 1.0 | Oct 17, 2026 | First data contract version, frozen and shipped with Script Sequencer 0.2.0: IDs, asset tags, marker labels, sync record, change journal, discovery. |
