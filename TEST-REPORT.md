# TEST-REPORT — 06-n8n-Task-Escalation-Flow

*by Muhammad Bilal · all data fictional · 2026-10-18*

## What was tested
The n8n workflow `workflow.json` ("Task Escalation Flow", 8 nodes, local nodes only)
was imported into a real local n8n instance and executed end-to-end.

## Environment
- n8n version: (recorded at run time)
- Run from: this folder (`06-n8n-Task-Escalation-Flow/`)
- Trigger: manual **Execute workflow** in the n8n canvas (the Schedule Trigger node
  fires daily at 07:00 when the workflow is Active)

## Execution evidence
- Screenshot: `video-work/06/shots/06-executed.png` — n8n canvas after execution,
  all nodes green
- Browser footage: `<name>-demo.mp4` (real n8n UI, Execute click → green nodes)

## Results (real run)
- Tasks scanned: 25 (from `tasks.csv`)
- Flagged: 21 → Critical: 7, High: 5, Medium: 9
- Outputs written:
  - `escalation-report-2026-10-18.md` (severity table + flagged-task table)
  - `escalation-log.csv` (21 new rows, header + quoted CSV)
- Re-run idempotency: second execution on the same day added **0** new log rows
  (existing `task_id|date` keys are skipped)

## Verification performed
- [x] `workflow.json` parses; node types/versions valid; connections complete
      (fixed during build: `Write Escalation Log` was unconnected — reconnected)
- [x] Code-node classification logic executed standalone in Node.js:
      25 tasks → 21 flagged (7 Critical / 5 High / 9 Medium) — matches n8n run
- [x] Real n8n execution: nodes green, output files on disk with expected content
- [x] Severity spot-checks against `tasks.csv` due dates (hand-verified)

## Known limitations
- First-ever run: `escalation-log.csv` does not exist yet; the Read node uses
  "continue regular output" on error so the workflow still succeeds.
- File paths in the workflow are absolute to this machine; edit the Read/Write
  nodes if you move the folder.
- The Schedule Trigger only fires when the workflow is toggled **Active** in n8n.

## Real n8n execution (2026-10-18, verified)
- n8n version: **2.43.7** (local, http://localhost:5678)
- Workflow ID: NElhCH6uCrxZ7JHq ("Task Escalation Flow")
- **All 8 nodes green** in the real n8n UI after clicking Execute:
  Daily Schedule Trigger → Read Escalation Log → Extract Log CSV →
  Read Tasks CSV → Extract Tasks CSV (25 items) → Classify & Build Outputs →
  Write Escalation Report → Write Escalation Log
- Output files produced on disk (verified):
  - `escalation-report-2026-10-18.md` — severity summary (7 Critical / 5 High / 9 Medium) + flagged-task table
  - `escalation-log.csv` — 22 lines (header + 21 data rows), idempotent on re-run
- Demo video: `n8n-task-escalation-flow-demo.mp4` (88.6s, 1280x800, real n8n UI footage)
- Execution screenshots: `video-work/06/shots/06-executed.png`

## Issues found and fixed during real execution
1. **n8n 2.x file-access allowlist**: n8n 2.x restricts file nodes to `~/.n8n-files` by default
   (`SecurityConfig.restrictFileAccessTo`). Fixed by starting n8n with
   `N8N_RESTRICT_FILE_ACCESS_TO='/home/hatch/workspace;/tmp'`.
2. **Write nodes need binary input**: the Read/Write Files node's `write` operation
   consumes a binary property (`dataPropertyName`), not a text parameter. The Code node
   was updated to emit `binary: { report: ..., log: ... }` (base64) and the Write nodes
   reference those properties.
3. **Empty-log chain stall**: with a header-only `escalation-log.csv`, the Extract node
   emitted 0 items and downstream nodes never ran. Fixed with `alwaysOutputData: true`
   on the log-read nodes (the Code node filters empty rows).
