# Progress Visualization Rules

## Overview
After each stage completion and gate confirmation, generate/update progress visualization artifacts to provide real-time visibility into workflow progress.

## Artifacts

### 1. progress.html
- Copy from `rules/progress-template.html`
- Inject current state data from `workflow-state.yaml`
- Serve as the main dashboard for users to monitor progress

### 2. progress.json
- Create/update from `rules/progress-template.json`
- Sync with `workflow-state.yaml`
- Provide data source for the HTML dashboard's AJAX polling

## Progress Calculation

| Stage | Percentage |
|-------|------------|
| env-check | 10% |
| Stage 1 + Gate 1 | 20% |
| Stage 1.3 (Project Type) | 25% |
| Stage 1.5 (Tech Stack) | 30% |
| Stage 1.6 (Assessment) | 35% |
| Stage 2.① + Gate 2 | 40% |
| Stage 2.② + Gate 3 | 50% |
| Stage 2 Group A | 60% |
| Stage 2 Group B | 70% |
| Stage 2 Group C | 80% |
| Stage 3 | 95% |
| Final gate | 100% |

## Update Timing

Update visualization artifacts at these points:
1. After env-check completes
2. After each gate confirmation
3. After each parallel group completes
4. After Stage 3 module completion
5. After smoke test completion
6. Every 5 seconds (auto-refresh via AJAX polling)

## Dashboard Features

### Stage Timeline
- Color-coded stage nodes (completed: green, current: purple, pending: gray)
- Pulse animation for current stage
- Duration display for completed stages

### Module Progress (Stage 3)
- Per-module progress bars
- Status indicators (done/running/failed/pending)
- Completion percentage

### Timing Statistics
- Total workflow duration
- Per-stage duration
- Format: hours/minutes/seconds

### Artifact Table
- Version tracking
- Status badges (confirmed/draft/stale/pending)
- Generation timestamps

### Gate Status
- Confirmed/rejected/pending indicators
- Visual grouping for easy review