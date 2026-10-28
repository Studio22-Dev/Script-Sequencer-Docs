   # Roadmap

   Script Sequencer turns a Fountain or Final Draft script/screenplay into a Sequencer structure in Unreal Engine 5.6–5.8, and keeps it in sync as the script changes, without overwriting anyone's work.
   
   ## Current Release: 0.3.0, on Fab
   
   ### What's new in 0.3.0
   - A report window: each sync's changes are shown in one list, with a choice per script edit (ripple later shots, extend the shot, ignore, keep, or remove)
   - Final Draft lines followed by Final Draft's own paragraph IDs, so heavy rewrites keep their ID links.
   - Revision-aware reports for Final Draft scripts: changes grouped by revision set, each changed line tagged with its set
   - Markers now follow dragged lines and shots, with shortened labels and IDs that can be hidden
   - Merged scenes keep the sequence of the scene that brought the most lines
   - Data contract 1.1 for pipeline tools, with every 1.0 file still valid

   When working on a team, upgrade the editor that owns each script's sync first.
   
   ### Features Since 0.2.0
   - A master sequence with a shot per scene, or per shot note
   - Markers and silent scratch audio for every line
   - Optional empty cameras ready for blocking
   - Updates on each save, or only when you choose Sync
   - EDL and OTIO edits respected
   - One machine syncs each imported script, tested with Perforce and Git LFS
   
   ## Next: 0.4.0, In Development
   - A casting table: each character gets an empty binding of its Blueprint or MetaHuman in every scene sequence
   - Speaker cameras: a placeholder camera per character plus a wide camera, cutting on each change of speaker dialogue
   - Take Recorder slates set from scene codes, with each take recorded against the script lines it covers, virtual-camera recordings included
   - Capture lists per scene for the stage, as CSV or Markdown, plus a slate export for Mocap Manager
   - Flags on lines rewritten after they were captured
   - Batch import for series: a folder of episode scripts, one grouped report
   - Naming tokens for render file names that follow script codes
   
   New features may be off by default, so check the Automation Settings in the Tools window to make sure
   
   ## Follow Along   
   - [Forum thread](https://forums.unrealengine.com/t/studio22-dev-script-sequencer-early-access/2843536): progress posts and short clips as each part works. Questions and requests go here too.
   - [YouTube](https://www.youtube.com/@Studio-fd5cc): a short overview, with a longer tutorial video in the works
   - [Get it on Fab](https://fab.com/s/d64f0cb6cf3e)