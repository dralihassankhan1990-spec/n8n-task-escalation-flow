# Task Escalation Flow — n8n automation

*by Muhammad Bilal · all data fictional · 100% local, no APIs, no cloud, no credentials*

## The problem
Project managers lose hours every week scanning task lists for what's overdue, what's due in the next 48 hours, and what needs escalating. Missed deadlines surface in status meetings instead of beforehand.

## What it does
A daily n8n workflow that reads `tasks.csv`, classifies every open task and writes two outputs:

- **`escalation-report-YYYY-MM-DD.md`** — severity summary + full flagged-task table
- **`escalation-log.csv`** — append-only log of every escalation (idempotent: re-running the same day adds nothing)

### Severity rules
| Severity | Rule |
|---|---|
| Critical | overdue > 7 days **or** blocks a milestone |
| High | overdue ≤ 7 days |
| Medium | due within 48 hours or within this week |

`Done` and `Cancelled` tasks are excluded.

## Workflow (8 nodes)
```
Daily Schedule Trigger (cron 0 7 * * *)
  → Read Escalation Log (continue on fail — first run has no log yet)
  → Extract Log CSV
  → Read Tasks CSV
  → Extract Tasks CSV
  → Classify & Build Outputs (Code — classification, markdown, idempotent CSV)
  → Write Escalation Report
  → Write Escalation Log
```

## Run it in n8n
1. Start n8n locally: `N8N_PORT=5678 n8n start` (or `npx n8n`).
2. Open http://localhost:5678, create the owner account on first launch.
3. Workflows → `⋯` → **Import from file** → select `workflow.json`.
4. **Before first run:** edit the two Read/Write node paths if you moved this folder — they point at this folder by default (`tasks.csv`, `escalation-log.csv`, outputs land here too).
5. Click **Execute workflow** to run it now, or toggle **Active** to run daily at 07:00.
6. Check `escalation-report-<date>.md` and `escalation-log.csv` in this folder.

## Files
| File | What it is |
|---|---|
| `workflow.json` | Importable n8n workflow (8 nodes, local nodes only) |
| `tasks.csv` | Sample input — 25 fictional PM tasks |
| `escalation-report-*.md` | Generated daily report (after a run) |
| `escalation-log.csv` | Generated idempotent escalation log (after a run) |
| `TEST-REPORT.md` | Real execution evidence |
| `<name>-demo.mp4` | Narrated demo video (real n8n footage) |

## Customise
- Change the cron in **Daily Schedule Trigger** for a different time.
- Edit severity rules in **Classify & Build Outputs** (the `if/else` chain at the top of the Code node).
- Point **Read Tasks CSV** at your real export — it needs columns `task_id,title,owner,status,due_date,blocks_milestone`.
