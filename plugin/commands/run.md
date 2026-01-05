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

## Step 4: Resolve Variables

For each step in the chain:
1. Replace `{{variable}}` placeholders with actual values
2. Support inputs from user arguments
3. Support outputs from previous steps
4. Support special variables: `{{cwd}}`, `{{home}}`, `{{env.VAR}}`

## Step 5: Execute Steps

For each step in the chain:

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

## Step 6: Report Results

Display execution summary:
```
✅ Chain 'plan-and-implement' completed successfully

Steps executed:
  1. plan (feature-dev:feature-dev) - ✅ Complete
  2. implement (ralph-wiggum) - ✅ Complete

Results:
  - Spec file: audit/oauth.md
  - Implementation: Complete
```

## Error Handling

If chain execution fails:
```
❌ Chain 'plan-and-implement' failed at step 2: implement

Error: ralph-wiggum plugin not found

To fix:
  - Install ralph-wiggum plugin
  - Check plugin is enabled
```

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
