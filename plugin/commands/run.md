---
skill_name: run
description: Execute a chain by name
arguments:
  chain_name:
    description: Name of the chain to execute (positional)
    required: true
  cwd:
    description: Working directory for chain execution
    required: false
---

# Chainer: Run Chain

Execute a configured chain by name.

## Step 1: Parse Arguments

Extract the chain name and any additional arguments:

```javascript
const args = process.argv.slice(2);
const chainName = args[0];
const additionalArgs = {};

// Parse --key=value or --key value arguments
for (let i = 1; i < args.length; i++) {
  if (args[i].startsWith('--')) {
    const key = args[i].substring(2);
    if (args[i].includes('=')) {
      const [k, v] = args[i].substring(2).split('=');
      additionalArgs[k] = v;
    } else if (i + 1 < args.length && !args[i + 1].startsWith('--')) {
      additionalArgs[key] = args[i + 1];
      i++;
    } else {
      additionalArgs[key] = true;
    }
  }
}
```

## Step 2: Load Configuration

Read the chainer configuration from:
1. `.claude/chainer.local.md` (project-specific)
2. `~/.claude/chainer.local.md` (global)
3. Fallback to plugin defaults

Use the Read tool to load and parse the YAML frontmatter.

## Step 3: Validate Chain

Check that:
- Chain exists in configuration
- Chain is enabled
- All required inputs are provided

## Step 4: Conflict Detection

Check if another chain is currently running by reading `.claude/chainer-state.json`.

If a chain is running:
1. Check if it's still alive using `ps -p <pid>`
2. If alive, display conflict options:

```
⚠️  Another chain is already running

Currently running: plan-and-implement (oauth)
  Step 2/2: implement
  Running for: 10 min
  PID: 12345

Options:
  [C] Continue and spawn new tmux window/session (recommended)
  [W] Wait for current chain to finish
  [A] Abort this chain

Choose [C/W/A]:
```

3. Handle user choice:
   - **C (Continue)**: Proceed to Step 5 (tmux spawning)
   - **W (Wait)**: Poll every 10 seconds until running chain completes
   - **A (Abort)**: Exit with message "Aborted by user"

If no chain is running, proceed to Step 6 (Resolve Variables).

## Step 5: tmux Spawning (if needed)

Only execute this step if user chose "Continue" from conflict detection.

### Check Configuration

Read `auto_spawn_strategy` from chainer config (defaults to "ask"):
- **ask**: Prompt user each time
- **always**: Always auto-spawn
- **never**: Show error and abort

### Determine tmux Context

Check if currently in tmux using `echo $TMUX`:
- If `$TMUX` is set → in tmux session
- If empty → not in tmux

### Spawn Strategy

**If in tmux session:**
1. Create new window in current session
2. Use `tmux new-window -n "chainer-<chain-name>"`
3. In new window, execute chain with same arguments

**If not in tmux:**
1. Create new tmux session
2. Use `tmux new-session -d -s "chainer-<timestamp>" "cd <cwd> && /chainer:run <chain-name> <args>"`
3. Display message with attach command

**Commands to execute:**

```bash
# In tmux - create new window
tmux new-window -n "chainer-oauth" -c "~/worktrees/oauth" \
  "claude /chainer:run plan-and-implement --prompt='Build OAuth' --feature_name='oauth'"

# Not in tmux - create new session
tmux new-session -d -s "chainer-1736123456" -c "~/worktrees/oauth" \
  "claude /chainer:run plan-and-implement --prompt='Build OAuth' --feature_name='oauth'"

# Then inform user
echo "Chain started in new tmux session: chainer-1736123456"
echo "Attach with: tmux attach -t chainer-1736123456"
```

After spawning, exit current process since chain is running in tmux.

## Step 6: Resolve Variables

For each step in the chain:
1. Replace `{{variable}}` placeholders with actual values
2. Support inputs from user arguments
3. Support outputs from previous steps
4. Support special variables: `{{cwd}}`, `{{home}}`, `{{env.VAR}}`

## Step 7: Initialize State Tracking

Before executing steps, create/update `.claude/chainer-state.json`:

```json
{
  "running_chains": [
    {
      "chain": "plan-and-implement",
      "started": "2025-01-05T10:30:00Z",
      "cwd": "/Users/user/worktrees/oauth",
      "current_step": 0,
      "total_steps": 2,
      "step_name": "starting",
      "pid": 12345
    }
  ],
  "completed_chains": []
}
```

Get current PID using `echo $$` in Bash.

## Step 8: Execute Steps

For each step in the chain:

1. **Update state** before step execution:
   ```json
   {
     "current_step": 1,
     "step_name": "plan"
   }
   ```

2. **Execute step**:

### Skill Step
```javascript
{
  type: "skill",
  skill: "feature-dev:feature-dev",
  args: "Build OAuth flow"
}
```
Use the Skill tool to invoke: `Skill(skill: "feature-dev:feature-dev", args: "Build OAuth flow")`

### Script Step
```javascript
{
  type: "script",
  script: "npm test && npm run build"
}
```
Use the Bash tool to execute the script.

### Step Execution Notes
- Execute steps sequentially (one after another)
- Wait for each step to complete before starting the next
- If a step fails, stop execution and report error
- Capture outputs if specified in step configuration
- Update state file after each step completes

## Step 9: Report Results and Update State

**On successful completion:**

1. Remove chain from `running_chains`
2. Add to `completed_chains`:
   ```json
   {
     "chain": "plan-and-implement",
     "started": "2025-01-05T10:30:00Z",
     "completed": "2025-01-05T10:45:00Z",
     "cwd": "/Users/user/worktrees/oauth",
     "success": true
   }
   ```

3. Display execution summary:
   ```
   ✅ Chain 'plan-and-implement' completed successfully

   Steps executed:
     1. plan (feature-dev:feature-dev) - ✅ Complete
     2. implement (ralph-wiggum) - ✅ Complete

   Results:
     - Spec file: audit/oauth.md
     - Implementation: Complete

   Duration: 15 minutes
   ```

**On failure:**

1. Remove chain from `running_chains`
2. Add to `completed_chains` with `success: false`
3. Keep last 10 completed chains only

## Error Handling

If chain execution fails:
```
❌ Chain 'plan-and-implement' failed at step 2: implement

Error: ralph-wiggum plugin not found

To fix:
  - Install ralph-wiggum plugin
  - Check plugin is enabled
```

Update state file to mark as failed.

## Implementation Notes

You are implementing this command. You should:

1. **Read the configuration file** using the Read tool
2. **Parse YAML frontmatter** to extract chain definitions
3. **Validate inputs** against chain requirements
4. **Resolve variables** in step configurations
5. **Execute steps sequentially** using appropriate tools:
   - Skill tool for skill steps
   - Bash tool for script steps
6. **Report progress and results** clearly

Remember:
- Change to `--cwd` directory if specified before execution
- Handle missing plugins gracefully
- Provide helpful error messages
- Track step outputs for variable substitution
