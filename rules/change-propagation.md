# Change Propagation Rule

## Dependency Graph

```
requirement
    ↓
product_design
    ↓
prd
    ↓
┌─────────────────────────────────┐
│  Group A (parallel)             │
│  ┌─────────┐   ┌─────────────┐ │
│  │prototype│   │backend_arch │ │
│  └────┬────┘   └──────┬──────┘ │
└───────┼───────────────┼────────┘
        ↓               ↓
┌───────┴───────┐ ┌─────┴──────────┐
│ Group B       │ │ Group C        │
│ frontend_arch │ │ test_case      │
└───────────────┘ └────────────────┘
```

## Stale Marking Rules

When an artifact's `status` changes from `confirmed` back to `draft`:

1. Mark all direct and indirect downstream artifacts as `stale` in state file
2. Update parallel group status to `stale` if group contains stale tasks
3. Notify user with list of affected artifacts and ask:
   - Option A: Re-generate all stale artifacts
   - Option B: Only re-generate specified artifacts, keep others stale
   - Option C: Cancel change, revert to previous state

## Propagation Matrix

| Modified | Marks stale |
|----------|-------------|
| requirement | product_design, prd, prototype, frontend_arch, backend_arch, test_case, generated code |
| product_design | prd, prototype, frontend_arch, backend_arch, test_case, generated code |
| prd | prototype, frontend_arch, backend_arch, test_case, generated code |
| prototype | frontend_arch, generated frontend code |
| backend_arch | test_case, generated backend code |
| frontend_arch | generated frontend code |

## Generated Code Handling

If Stage 3 has started, do NOT delete generated code automatically. Set `code_stale: true` in state file and ask user whether to regenerate.

## Versioning

- Version increments: v1 → v2 → v3
- Old files are preserved (e.g. `prd-v2.md` alongside `prd-v1.md`)
- Record version history in `iterations` section of state file

## Regeneration Order

1. Group A (prototype + backend_arch, parallel)
2. Group B (frontend_arch, after prototype)
3. Group C (test_case, after backend_arch)
