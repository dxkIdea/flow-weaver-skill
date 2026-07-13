# Execution Logging & Audit Rules

## Log Entry Format

Each log entry in `execution_log` must follow:

```yaml
- timestamp: "<ISO-8601>"
  level: INFO           # DEBUG | INFO | WARN | ERROR
  stage: stage1
  activity: "Stage started"
  duration_ms: null
  sub_skill: brainstorming
  details: "Requirement brainstorming initiated"
```

## Mandatory Log Points

### Stage Lifecycle
- **Stage Started**: When a stage begins
- **Stage Completed**: When a stage finishes
- **Stage Failed**: When a stage fails
- **Stage Timeout**: When a stage times out

### Gate Events
- **Gate Entered / Confirmed / Rejected**

### Sub-skill Events
- **Sub-skill Invoked / Failed**: Only for fallbacks

### Artifact Events
- **Artifact Created**: When generated
- **Artifact Marked Stale**: When upstream changes

### Error Events
- **Error Occurred / Recovered / Escalated**

## Log Levels

- **INFO**: Stage lifecycle, gate changes, artifact generation
- **WARN**: Fallback activation, missing optional tools
- **ERROR**: Stage failures, unrecoverable errors

## Audit Trail

- All gate confirmations/rejections
- All fallback activations
- All error escalations to humans

## Log Rules

1. Append to `execution_log` array in workflow-state.yaml
2. Archive when array exceeds 1000 entries
3. Never log passwords, API keys, or personal data
