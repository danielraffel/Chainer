---
skill_name: status
description: Show running chains
---

# Chainer: Show Running Chains

Display status of currently running chains.

## Step 1: Check for State File

Look for `.claude/chainer-state.json` in the current directory.

## Step 2: Parse State

If state file exists, parse the JSON to get running chains:

```json
{
  "running_chains": [
    {
      "chain": "plan-and-implement",
      "started": "2025-01-05T10:30:00Z",
      "cwd": "~/worktrees/oauth",
      "current_step": 2,
      "total_steps": 2,
      "pid": 12345
    }
  ]
}
```

## Step 3: Display Status

Format the output:

```
┌─────────────────────────────────────────┐
│ Chainer Status                          │
├─────────────────────────────────────────┤
│ 🔄 plan-and-implement (oauth)           │
│    Step 2/2: implement                  │
│    Directory: ~/worktrees/oauth         │
│    Running: 10 min                      │
│                                         │
│ ✅ plan-only (billing)                  │
│    Completed 5 min ago                  │
│    Directory: ~/worktrees/billing       │
└─────────────────────────────────────────┘

💡 Tip: Use Ctrl+C to stop a running chain
```

## No Running Chains

If no state file or no running chains:

```
No chains currently running

Available chains:
  /chainer:list      Show available chains
  /chainer:run       Execute a chain
```

## Implementation Notes

You are implementing this command. You should:

1. **Check for state file** at `.claude/chainer-state.json`
2. **Parse JSON** if file exists
3. **Validate running chains** (check if PIDs still exist)
4. **Calculate time running** from started timestamp
5. **Format output** with visual indicators:
   - 🔄 for running
   - ✅ for completed
   - ❌ for failed
6. **Show helpful message** if no chains running

Remember:
- State tracking is future work (Phase 4)
- For now, just show "No chains currently running"
- This command will be fully implemented in Phase 4
