# Script Sequencer data contract 1.1 (draft)

> **Draft: may change until Script Sequencer 0.3.0 ships.** Script Sequencer 0.2.x writes
> [contract 1.0](DataContract.md), which stays valid. Build against 1.0; this page
> shows what 0.3.0 will add, so your tools are ready for it.

This is a technical spec: the file formats Script Sequencer writes, so your own pipeline
tools can read them. It grants no rights and adds no terms; your Fab license covers that.

## What 1.1 changes

Contract 1.1 only **adds**. Every new field is optional, nothing is renamed or removed, and
every 1.0 file is valid 1.1. A tool written for 1.0 keeps working, as long as it follows the
reading rules below.

| Where | New in 1.1 | What it holds |
| --- | --- | --- |
| Sync record | `episode` | The `{episode}` value the script was imported with |
| Sync record | `revision` | The script's current revision set: `name`, plus `color` and `index` from Final Draft; or null |
| Sync record | `cast[]` | The casting table: `character` and `class` (an actor class path) |
| `units[]` | `cast[]` | The character bindings Script Sequencer created in that unit's sequence: `character`, `binding` |
| `lines[]` | `captures[]` | Every take that covered the line: `take`, `slate`, `number`, `recorder` (`take_recorder` or `mocap_manager`), `time` (UTC), `text_hash` |
| `lines[]` | `changed_after_capture` | True when the line's text changed after its newest take |
| `lines[]` | `revision` | The revision set in which the line last changed, or null |
| Journal entry | `source: "capture"` | A take was recorded; no sync ran |
| Journal entry | `episode`, `revision` | The record's values at the time of the entry |
| Journal entry | `lines.captured`, `lines.changed_after_capture`, `lines.capture_cleared` | Lines a take covered, captured lines whose text changed, and flags that cleared |
| Asset tags | `suite.episode` | The episode, on every generated asset |
| Asset tags | `suite.lines`, `suite.take` | On a recorded take's Level Sequence: the line IDs it covers (comma-separated) and its take number |

A field only appears once the feature that writes it is in use. A script that uses none of
them looks exactly like a 1.0 record, apart from `"contract_version": "1.1"`.

## Downloads

| File | What it is |
| --- | --- |
| [suite_contract_v1.1.schema.json](suite_contract_v1.1.schema.json) | The JSON Schema (2020-12) for 1.1. `$id`: `urn:script-sequencer-suite:contract:1.1` |
| [Ep01.sync.json](examples/contract-1.1/Ep01.sync.json) | A sync record using every 1.1 field |
| [Ep01.changes.jsonl](examples/contract-1.1/Ep01.changes.jsonl) | Two journal entries, the second from a recorded take |
| [Ep01.lipsync.json](examples/contract-1.1/Ep01.lipsync.json) | A companion plug-in file header at 1.1 |

The [1.0 schema](suite_contract_v1.0.schema.json) stays available, unchanged.

## Reading files from a newer minor version

- Accept any `1.x` `contract_version`. Refuse `2.0` and later with a message.
- Ignore fields you don't know. If you write a file back, keep them.
- Skip journal entries whose `source` you don't know, rather than stopping. 1.1 adds
  `capture`, which the 1.0 schema doesn't list, so a strict validator using the 1.0 schema
  rejects those entries. Validate against the newest schema you ship, or check `source` first.
- `revision` is an object with at least a `name`, or null. A Fountain script's title page
  line `Revision: Blue` gives the name only.
- To check whether Script Sequencer is installed, test the major version:
  `any(v.split(".")[0] == "1" for v in ss.contract_versions())`, not `"1.0" in ...`.
  From 0.3.0, `contract_versions()` returns `["1.0", "1.1"]`.

## Upgrading a team

From 0.3.0, Script Sequencer keeps fields it doesn't recognise when it saves a sync record.
0.2.x doesn't: a 0.2.x editor that syncs a script rewrites its record as 1.0 and drops the
1.1 fields. So on a team, upgrade the editor that syncs each script (its `sync_owner`) first.

## Changelog

| Version | Status | Changes |
| --- | --- | --- |
| 1.1 | Draft, ships with 0.3.0 | Everything in *What 1.1 changes* above |
| 1.0 | Frozen, shipped with 0.2.0 (Oct 17, 2026) | [First version](DataContract.md) |
