---
name: cron-visualizer
description: Visualise all Hermes cron schedules on a timeline, detect overlaps and gaps, generate human-readable schedules, enable one-click edits, and export as markdown or calendar format.
version: 1.0.0
author: Hermes Agent
license: MIT
metadata:
  tags:
    - devops
    - cron
    - scheduling
    - visualization
    - automation
    - cli
  related_skills:
    - session-summary
commands:
  - name: show
    description: Display all cron schedules on a visual timeline
    args:
      - name: view
        type: string
        default: timeline
        description: View type (timeline, list, grid, detail)
      - name: day
        type: string
        default: today
        description: Day to visualise (today, tomorrow, or ISO date)
      - name: timezone
        type: string
        default: Europe/London
        description: Timezone for display
  - name: check
    description: Detect overlaps and gaps in schedule coverage
    args:
      - name: severity
        type: string
        default: warning
        description: Minimum severity to report (info, warning, critical)
      - name: range
        type: string
        default: 24h
        description: Time range to check (24h, 7d, 30d)
  - name: export
    description: Export schedules as markdown table or calendar format
    args:
      - name: format
        type: string
        required: true
        description: Output format (markdown, ical, json)
      - name: output
        type: string
        default: ""
        description: Output file path (defaults to stdout)
      - name: week
        type: integer
        default: 0
        description: Weeks ahead to export (0 = current week)
  - name: edit
    description: Open a cron schedule for one-click editing via hermes CLI
    args:
      - name: job
        type: string
        required: true
        description: Job name or ID to edit
      - name: schedule
        type: string
        default: ""
        description: New cron expression (e.g., '0 8 * * *')
      - name: enable
        type: boolean
        default: true
        description: Enable or disable the job
  - name: list
    description: List all registered cron jobs with metadata
    args:
      - name: filter
        type: string
        default: all
        description: Filter by status (all, active, disabled)
      - name: skill
        type: string
        default: ""
        description: Filter by owning skill name
---

# Cron Visualizer

Gain clarity on all your Hermes Agent scheduled tasks. See every cron job on a timeline, spot overlaps that waste resources, identify gaps in coverage, and export clean schedules — all with one-click editing from the CLI.

## How It Works

1. **Discovery**: Scans all Hermes skill cron configurations from `~/.hermes/skills/*/SKILL.md` and `~/.hermes/config.yaml`
2. **Timeline Rendering**: Maps each job onto a 24-hour timeline (or multi-day view) with visual markers
3. **Analysis Engine**: Detects overlapping executions, idle gaps, and scheduling conflicts
4. **Export Pipeline**: Converts schedule data into markdown tables, iCal files, or JSON for programmatic use
5. **Edit Bridge**: Invokes `hermes cron` commands directly to modify schedules without leaving the tool

## Commands Reference

### `show`

Display your cron schedules as a visual timeline in the terminal.

```bash
# Default: today's timeline view
hermes cron-visualizer show

# List view for compact display
hermes cron-visualizer show --view list

# Grid view showing full week
hermes cron-visualizer show --view grid

# Specific date with detail view
hermes cron-visualizer show --view detail --day 2026-05-05

# Tomorrow's schedule
hermes cron-visualizer show --day tomorrow
```

**Timeline View Output**:

```
06:00 ──┬── weather-fetch ───────── 2min
07:00 ──┼── habit-reminder ──────── 1min
         ├── morning-brief ──────── 3min
         │   ⚠ OVERLAP: habit-reminder
08:00 ──┼── fitness-log-sync ────── 5min
12:00 ──┼── midday-energy-prompt ── 1min
18:00 ──┼── evening-review ──────── 4min
21:00 ──┼── daily-note-cleanup ──── 2min
22:00 ──┴── sleep-prep-reminder ── 1min

Legend: ── scheduled  ⚠ overlap detected  ✓ gap-free
```

**List View Output**:

| Time | Job | Frequency | Est. Duration | Skill |
|------|-----|-----------|---------------|-------|
| 06:00 | weather-fetch | daily | 2min | weather |
| 07:00 | habit-reminder | daily | 1min | habits |
| 07:00 | morning-brief | daily | 3min | routine-optimizer |
| ... | ... | ... | ... | ... |

### `check`

Analyze your schedule for problems.

```bash
# Check for overlaps in next 24 hours
hermes cron-visualizer check

# Check the full week
hermes cron-visualizer check --range 7d

# Only show critical issues
hermes cron-visualizer check --severity critical

# Check next 30 days for rare job collisions
hermes cron-visualizer check --range 30d
```

**Detected Issues**:

- **Overlap** (warning): `habit-reminder` and `morning-brief` both run at 07:00 — consider staggering by 5min
- **Gap** (info): No scheduled tasks between 13:00-17:00 — consider adding afternoon check-in
- **Conflict** (critical): `weekly-report-gen` and `health-export` both run Sunday 00:00 and require >500MB RAM

### `export`

Export your schedule for sharing, importing, or backup.

```bash
# Markdown table to stdout
hermes cron-visualizer export --format markdown

# Export this week as iCal file
hermes cron-visualizer export --format ical --output ~/obsidian-vault/3-Resources/schedule.ics

# Export next week's schedule as JSON
hermes cron-visualizer export --format json --week 1 --output schedule-next-week.json

# Markdown table saved to Obsidian
hermes cron-visualizer export --format markdown --output ~/obsidian-vault/3-Resources/cron-schedule.md
```

**Markdown Export Example**:

```markdown
# Hermes Cron Schedule — Week 18, 2026

- **06:00 daily** — `weather-fetch` (weather skill) ~2min
- **07:00 daily** — `habit-reminder` (habits skill) ~1min
- **07:05 daily** — `morning-brief` (routine-optimizer skill) ~3min
- **08:00 daily** — `fitness-log-sync` (fitness-nutrition skill) ~5min
- **12:00 daily** — `midday-energy-prompt` (routine-optimizer skill) ~1min
- **18:00 daily** — `evening-review` (session-summary skill) ~4min
- **21:00 daily** — `daily-note-cleanup` (obsidian skill) ~2min
- **22:00 daily** — `sleep-prep-reminder` (routine-optimizer skill) ~1min
- **Sun 00:00 weekly** — `weekly-health-dashboard` (apple-health-sync) ~10min
- **Mon 08:00 weekly** — `weekly-report-gen` (routine-optimizer) ~5min
```

**iCal Export**: Produces standard `.ics` files importable by Google Calendar, Apple Calendar, Outlook, etc.

### `edit`

Modify cron schedules directly from the CLI.

```bash
# Change schedule for a specific job
hermes cron-visualizer edit --job morning-brief --schedule "0 7 * * 1-5"

# Stagger overlapping jobs
hermes cron-visualizer edit --job morning-brief --schedule "5 7 * * *"

# Disable a job temporarily
hermes cron-visualizer edit --job midday-energy-prompt --enable false

# Re-enable it
hermes cron-visualizer edit --job midday-energy-prompt --enable true
```

The edit command runs the equivalent `hermes cron update <job> --schedule "..."` or `hermes cron disable <job>` under the hood, providing a smoother interface.

### `list`

Show all registered cron jobs with full metadata.

```bash
# List all jobs
hermes cron-visualizer list

# Only active jobs
hermes cron-visualizer list --filter active

# Jobs from a specific skill
hermes cron-visualizer list --skill routine-optimizer
```

## Configuration

Add to your Hermes config (`~/.hermes/config.yaml`):

```yaml
cron-visualizer:
  # Default display timezone
  timezone: Europe/London
  # Estimation for job durations (minutes) — used for overlap detection
  # Override per-job if actual durations differ
  default_duration_min: 2
  # Custom duration estimates for known slow jobs
  duration_overrides:
    weekly-health-dashboard: 10
    weekly-report-gen: 5
    fitness-log-sync: 5
  # Wording preferences for exports
  export:
    time_format: "24h"
    date_format: "YYYY-MM-DD"
  # Where to save exported schedules
  obsidian_vault: ~/obsidian-vault
  resources_path: "3-Resources"
```

## Cron Job Schema

Each registered cron job should have this structure in its skill's SKILL.md or in `~/.hermes/config.yaml`:

```yaml
cron:
  - id: morning-brief
    schedule: "0 7 * * *"
    timezone: Europe/London
    skill: routine-optimizer
    command: hermes routine-optimizer analyze --days 1
    estimated_duration_min: 3
    description: "Generate morning routine brief from yesterday's data"
```

The visualizer reads:
1. All `SKILL.md` files under `~/.hermes/skills/` for `cron:` sections
2. The `cron:` section in `~/.hermes/config.yaml`
3. Any dynamically registered jobs via `hermes cron list`

## Integration with Hermes Cron System

```bash
# The visualizer wraps standard hermes cron commands
hermes cron list          # Source of truth for active jobs
hermes cron add           # Used by edit to suggest new schedules
hermes cron update <id>   # Used by edit to modify schedules
hermes cron disable <id>  # Used by edit to turn off jobs
hermes cron enable <id>   # Used by edit to turn on jobs
```

## Overlap Detection Logic

The check command simulates a full time range using cron expression evaluation:

```python
# Pseudocode for overlap detection
for each minute in time_range:
    running_jobs = [j for j in jobs if j.would_run_at(minute)]
    active_at = []
    for job in running_jobs:
        active_at.append((job, minute, minute + job.duration))
    # Check if any two active windows overlap
    for pair in combinations(active_at, 2):
        if windows_overlap(pair[0], pair[1]):
            report_overlap(pair)
```

Overlap severity levels:
- **info**: Jobs overlap but share no resources (< 2 min overlap)
- **warning**: Jobs overlap and may contend for CPU/memory
- **critical**: Jobs overlap, both resource-heavy, or one depends on the other's output

## Gap Detection Logic

Gaps are reported when there are extended periods with no scheduled tasks during likely working hours:

- **Mon-Fri 08:00-22:00**: Gaps > 4 hours flagged
- **Sat-Sun 09:00-21:00**: Gaps > 6 hours flagged
- **Night hours (22:00-06:00)**: Gaps are expected and not flagged

## Pitfalls

1. **Cron expression parsing**: Complex cron expressions (ranges, step values like `*/15`) must be fully supported. The skill uses the `croniter` library for accurate next-run calculations — verify edge cases like leap years and month-end dates.
2. **Timezone handling**: Jobs may be configured in different timezones. Always display in the configured default timezone (Europe/London for UK-based users) and note when a job's native timezone differs.
3. **Duration estimates**: Overlap detection depends on accurate duration estimates. New or unknown jobs default to 2 minutes, which may over/under-estimate. Update `duration_overrides` in config.
4. **Dynamic jobs**: Some skills register cron jobs dynamically (e.g., temporary monitoring tasks). The visualizer refreshes from `hermes cron list` on each run to catch these.
5. **Stale SKILL.md entries**: If a skill is removed but its cron entry remains in `~/.hermes/config.yaml`, it will still show up. Use `hermes cron-visualizer list --filter active` and `hermes cron list` to cross-reference.
6. **iCal export limitations**: Recurring rules in iCal use RRULE format. Complex cron expressions (e.g., `0 8 1-7 */3 1`) may not have a clean RRULE equivalent and are approximated. Always verify exported `.ics` files in your calendar app.
7. **Resource contention on low-spec hardware**: If running on a Raspberry Pi or similar, even minor overlaps can cause issues. Set aggressive overlap detection and increase `default_duration_min` to model real-world jitter.
8. **Large schedule ranges**: Checking 30 days of minute-by-minute overlap is computationally manageable but can be slow if you have 50+ cron jobs. The skill falls back to sampling if >100 unique job schedules exist.

## Verification Steps

1. **Basic listing**: Run `hermes cron-visualizer list` — should display all registered cron jobs or a "no cron jobs found" message, not an error.
2. **Timeline display**: Run `hermes cron-visualizer show` — should render a readable timeline. If no jobs exist, should prompt to create one.
3. **Overlap detection**: Create two jobs at the same time (e.g., `hermes cron add test1 --schedule "0 7 * * *"` and `hermes cron add test2 --schedule "0 7 * * *"`), then run `hermes cron-visualizer check` — should report the overlap.
4. **Gap detection**: Run `hermes cron-visualizer check --range 7d` with only a morning job configured — should flag the afternoon gap.
5. **Markdown export**: Run `hermes cron-visualizer export --format markdown` — should produce a clean markdown schedule to stdout.
6. **iCal export**: Run `hermes cron-visualizer export --format ical --output /tmp/test.ics` — should create a valid `.ics` file. Validate by importing into a calendar app or running `icalvalidate /tmp/test.ics`.
7. **Edit command**: Run `hermes cron-visualizer edit --job test1 --schedule "0 8 * * *"` — should update the schedule. Verify with `hermes cron list`.
8. **Obsidian integration**: Run `hermes cron-visualizer export --format markdown --output ~/obsidian-vault/3-Resources/cron-schedule.md` — file should appear in your vault and be visible in Obsidian.