# Script Sequencer data contract v1.0

**This is a technical spec.** "Data contract" is the software term for a file format that several programs promise to read and write the same way. This page describes the files and asset tags Script Sequencer writes, so your own tools and pipeline scripts can read them. It grants no rights and adds no terms; your Fab license covers that.

*Version 1.0, frozen. First shipped with Script Sequencer 0.2.0 (Early Access).* `suite-contract-v1.0.schema.json` (JSON Schema 2020-12, next to this file) is the machine-readable reference for every format below; this page explains it.

Where things are located:
| Item(s) | Location | Written by |
|---|---|---|
| Sync record | `<Project>/ScriptSequencer/<Script>.sync.json` | Script Sequencer only |
| Change journal | `<Project>/ScriptSequencer/<Script>.changes.jsonl` | Script Sequencer only |
| Asset tags | `suite.*` metadata on generated assets | Script Sequencer |
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
| `suite.contract` | Every asset Script Sequencer generates | Contract version, e.g. `1.0` |
| `suite.generator` | Same | `ScriptSequencer` |
| `suite.script` | Same | Script key |
| `suite.unit` | Level Sequences for a unit | Unit code |
| `suite.scene` | Level Sequences for a unit | Scene ID |
| `suite.line` | Scratch Sound Waves | Line ID |
| `suite.character` | Scratch Sound Waves | Character name as written in the script |
| `suite.slot` | Scratch Sound Waves | `scratch` |

Metadata tags are editor-only and not available at runtime; Script Sequencer is an editor plug-in. Every value is a string. The schema reserves further values (other `suite.slot` values and a `suite.take` key) that this version doesn't write. To check an asset's tags, validate them against the schema's `unitSequenceTags` or `lineAudioTags`.

## 3. Names and labels inside a sequence
| Element | Format |
|---|---|
| Shot section display name | The unit code |
| Line marker label | `SCR:<line ID> <CHARACTER>: <text>` (64 characters max) |
| Scene heading marker label | `SCR:<scene ID> <heading>` |
| Dialogue audio track | `Dialogue_<CHARACTER>` |

Readers match on the `SCR:<ID>` prefix only; the rest of the label is for clarity.

## 4. The sync record
The plug-in's record of what it generated from one script and where: every line's ID, the sequence and frames it landed on, what a person has taken over (`owned`), and which editor syncs the script. Comparing this record with what's in Sequencer is how Script Sequencer tells its own work from yours.

`<Project>/ScriptSequencer/<Script>.sync.json`. **Only Script Sequencer writes it**, automatically (write a temp file, then rename). Tools that read it must ignore fields they don't know.

```json
{
  "contract_version": "1.0",
  "script": { "key": "Scripts/Ep01.fountain", "format": "fountain", "hash": "3f2c…" },
  "master": "/Game/Script/Ep01/Ep01_Master",
  "frame_rate": { "numerator": 24, "denominator": 1 },
  "sync_owner": "anna@ws-anim-04",
  "journal_seq": 42,
  "units": [
    { "code": "sc010", "scene": "S-1a2b3c", "shot_note": null,
      "asset": "/Game/Script/Ep01/Sequences/sc010",
      "state": "active", "range": [0, 480], "owned": false }
  ],
  "lines": [
    { "id": "L-4d5e6f", "scene": "S-1a2b3c", "unit": "sc010", "kind": "dialogue",
      "character": "ANNA", "text": "It's been boiling for an hour.", "text_hash": "9c1e…",
      "state": "active", "range": [240, 312],
      "audio": { "slot": "scratch", "asset": "/Game/Script/Ep01/Scratch/SW_Scratch_L_4d5e6f",
                 "track": "Dialogue_ANNA", "claimed_by": null },
      "owned": false }
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
| `units[]` | `code`, `scene`, `shot_note` (set for one unit per shot note), `asset`, `state` (`active`, `script_cut`, `edit_cut`), `range` on the master (null unless `active`), `owned` |
| `lines[]` | `id`, `scene`, `unit`, `kind` (`dialogue` or `action`), `character` (empty for action), `text`, `text_hash`, `state` (`active`, or `cut` when an owned line left the script), `range` inside the unit sequence, `audio`, `owned` |
| `lines[].audio` | `slot`, `asset`, `track`, `claimed_by` (reserved; null in this version). Null when scratch audio is off; always null for action lines. |
| `internal` | Script Sequencer's private state; don't read it, as it can change in any release |

Ranges are `[start, end)` frames at `frame_rate`. Hashes are lowercase SHA-256; `text_hash` is taken after collapsing runs of whitespace and trimming the ends.

## 5. The change journal
`<Project>/ScriptSequencer/<Script>.changes.jsonl` — one JSON object per sync, appended after the sync record is saved. A reader remembers the last `seq` it handled and reads only newer lines.

```json
{"seq": 42, "time": "2026-10-12T14:03:11Z", "contract_version": "1.0", "source": "script",
 "lines": {"added": ["L-91aa02"], "text_changed": ["L-4d5e6f"], "removed": [], "moved": [], "retimed": ["L-5e6f70"]},
 "units": {"added": [], "removed": [], "retimed": ["sc020"]}}
```

- `source` is `script` (a script save), `edit` (EDL or OTIO changes found on the Shots track) or `mixed` (both in one sync).
- `lines.moved` means a line changed scene, unit or order; `lines.retimed` means same text, new range — for example rippled by an earlier line. Anything you place per line can be re-timed from `retimed`.
- `lines.claimed` and `lines.released` are optional lists the schema reserves; this version doesn't write them.
- `time` is UTC. The journal keeps its last 1,000 entries; a reader that falls further behind re-reads the sync record.

## 6. Ownership
1. **Hand edits win.** Anything someone changes is owned and never changed automatically.
2. **Script Sequencer edits only what it generated**, if nobody has touched it since.
3. **Edits from an EDL or OTIO import** are hand edits. The shot section no longer matches, so it's owned and logged in the journal with `source: "edit"` (or `"mixed"` when the same sync also applied script changes).

## 7. Discovery
A Python script or plug-in can check whether Script Sequencer is installed, and which contract versions it writes, by importing its public module:

```python
try:
    import suite_script_sequencer as ss
except ImportError:
    ss = None                      # Not installed
if ss and "1.0" in ss.contract_versions():
    ...
```

`contract_versions()` returns `["1.0"]`; `capabilities()` returns `["sync_record", "journal", "slots", "edit_adoption"]`; `plugin_version()` returns the plug-in's own version. This is the only Script Sequencer module meant to be imported; everything else is internal and can change in any release.

## 8. Versioning
- Script Sequencer migrates older sync records forward on load, keeping a `.bak`.
- This page and the schema ship in the plug-in's `Docs` folder, with the changelog below.

## 9. Validating
Any JSON Schema 2020-12 validator works. The schema's root accepts a sync record or a journal entry; tag maps and discovery results are checked against their `$defs`. In Python, with `jsonschema` 4.18 or later:

```python
import json
from jsonschema import Draft202012Validator
from referencing import Registry, Resource

schema = json.load(open("Docs/suite-contract-v1.0.schema.json"))
registry = Registry().with_resource(schema["$id"], Resource.from_contents(schema))

def validator(defname=None):
    target = schema if defname is None else {"$ref": f'{schema["$id"]}#/$defs/{defname}'}
    return Draft202012Validator(target, registry=registry)

validator().validate(json.load(open("Ep01.sync.json")))   # a sync record or a journal entry
validator("lineAudioTags").validate(tags)                     # one Sound Wave's tags
```

## Changelog
| Version | Date | Changes |
|---|---|---|
| 1.0 | Oct 17, 2026 | First data contract version, frozen and shipped with Script Sequencer 0.2.0: IDs, asset tags, marker labels, sync record, change journal, discovery. |
