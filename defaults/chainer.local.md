---
chains:
  plan-and-implement:
    enabled: true
    description: "Plan with feature-dev, implement with ralph-wiggum"
    inputs:
      prompt: { required: true, description: "What to build" }
      feature_name: { required: true, description: "Feature name for spec file" }
    steps:
      - name: plan
        type: skill
        skill: feature-dev:feature-dev
        args: "{{prompt}}"
        output:
          spec_file: "audit/{{feature_name}}.md"
      - name: implement
        type: script
        script: |
          SCRIPT_PATH="$(find ~/.claude/plugins -name 'setup-ralph-loop.sh' -path '*ralph-wiggum*' 2>/dev/null | head -1)"
          bash "$SCRIPT_PATH" "Implement features from {{spec_file}}" --max-iterations 50 --completion-promise DONE

  plan-only:
    enabled: true
    description: "Just plan with feature-dev"
    inputs:
      prompt: { required: true, description: "What to build" }
      feature_name: { required: true, description: "Feature name for spec file" }
    steps:
      - name: plan
        type: skill
        skill: feature-dev:feature-dev
        args: "{{prompt}}"

  implement-only:
    enabled: true
    description: "Implement from existing spec"
    inputs:
      spec_file: { required: true, description: "Path to spec file" }
    steps:
      - name: implement
        type: script
        script: |
          SCRIPT_PATH="$(find ~/.claude/plugins -name 'setup-ralph-loop.sh' -path '*ralph-wiggum*' 2>/dev/null | head -1)"
          bash "$SCRIPT_PATH" "Implement features from {{spec_file}}" --max-iterations 50 --completion-promise DONE

defaults:
  spec_directory: audit
  max_iterations: 50
---

# Chainer Configuration

This file contains your custom chain definitions. Copy it to `~/.claude/chainer.local.md` (global) or `.claude/chainer.local.md` (project-specific) to use.

## Usage

```bash
# Run a chain
/chainer:run plan-and-implement --prompt="Build OAuth flow" --feature_name="oauth"

# List available chains
/chainer:list

# Check running chains
/chainer:status
```

## Creating Custom Chains

Add new chains to the YAML frontmatter above. Each chain has:
- `enabled`: true/false
- `description`: What the chain does
- `inputs`: Required/optional parameters
- `steps`: Sequential actions to execute

### Step Types

- **skill**: Invoke a Claude Code skill/plugin
- **script**: Run bash commands
- **mcp**: Call an MCP server tool (future)
- **prompt**: Ask user mid-chain (future)
- **wait**: Wait for file/condition (future)

### Variable Substitution

Use `{{variable_name}}` to reference inputs or outputs from previous steps.

## Examples

See `community-chains/` directory for shareable chain examples.
