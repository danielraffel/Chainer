# ⛓️ Chainer

Universal plugin orchestration for Claude Code - chain any plugins together with simple configuration.

## What is Chainer?

Chainer lets you combine multiple Claude Code plugins into automated workflows called "chains". Instead of manually running each plugin one after another, define a chain once and execute your entire workflow with a single command.

## Features

- **Config-Driven**: Define chains in YAML - no code required
- **Universal**: Chain any Claude Code skills or plugins together
- **Visual Editor**: Use `settings.html` to build chains with drag-and-drop
- **Variable Substitution**: Pass outputs from one step to the next
- **Built-in Chains**: Get started immediately with pre-configured workflows

## Quick Start

### Installation

```bash
# Clone the repository
git clone https://github.com/danielraffel/Chainer ~/.claude/plugins/chainer

# Copy default configuration
cp ~/.claude/plugins/chainer/defaults/chainer.local.md ~/.claude/chainer.local.md

# Open settings page to customize
open ~/.claude/plugins/chainer/settings.html
```

### Basic Usage

```bash
# List available chains
/chainer:list

# Run a chain
/chainer:run plan-and-implement \
  --prompt="Build OAuth authentication" \
  --feature_name="oauth"

# Check running chains
/chainer:status
```

## Built-in Chains

### `plan-and-implement`
Complete feature development workflow:
1. Plan with `feature-dev:feature-dev`
2. Implement with `ralph-wiggum` loop

```bash
/chainer:run plan-and-implement \
  --prompt="Build user dashboard" \
  --feature_name="dashboard"
```

### `plan-only`
Just planning, no implementation:

```bash
/chainer:run plan-only \
  --prompt="Design payment system" \
  --feature_name="payments"
```

### `implement-only`
Implement from an existing spec:

```bash
/chainer:run implement-only \
  --spec_file="audit/oauth.md"
```

## Creating Custom Chains

### Configuration File

Create or edit `~/.claude/chainer.local.md`:

```yaml
---
chains:
  my-workflow:
    enabled: true
    description: "Custom development workflow"
    inputs:
      task: { required: true, description: "What to build" }
    steps:
      - name: plan
        type: skill
        skill: feature-dev:feature-dev
        args: "{{task}}"
      - name: test
        type: script
        script: npm test
      - name: build
        type: script
        script: npm run build

defaults:
  spec_directory: audit
  max_iterations: 50
---

# Your notes here
```

### Step Types

| Type | Description | Example |
|------|-------------|---------|
| `skill` | Invoke Claude Code skill | `feature-dev:feature-dev` |
| `script` | Run bash commands | `npm test && npm run build` |
| `mcp` | Call MCP server tool | Coming in v0.2 |
| `prompt` | Ask user mid-chain | Coming in v0.2 |
| `wait` | Wait for file/condition | Coming in v0.2 |

### Variable Substitution

Use `{{variable}}` syntax to reference:

- **Inputs**: `{{prompt}}`, `{{feature_name}}`
- **Step outputs**: `{{spec_file}}` (from previous steps)
- **Special vars**: `{{cwd}}`, `{{home}}`, `{{env.API_KEY}}`

## Visual Settings Editor

Open `settings.html` in your browser for a visual interface:

- Drag-and-drop step reordering
- Enable/disable chains with checkboxes
- Add/remove inputs and steps
- Import/export chains
- Download configuration file

```bash
open ~/.claude/plugins/chainer/settings.html
```

## Integration with Other Plugins

### With Worktree Manager

Create a worktree and run a chain:

```bash
# Two commands
/worktree-manager:start oauth
/chainer:run plan-and-implement --cwd="~/worktrees/oauth" --prompt="OAuth"

# Or use the combined chain (coming in Phase 3)
/chainer:run worktree-plan-implement --feature_name="oauth" --prompt="OAuth"
```

### With Feature Dev

Chainer uses `feature-dev` for planning by default:

```bash
/chainer:run plan-only --prompt="Design API" --feature_name="api"
```

### With Ralph Wiggum

Chainer uses `ralph-wiggum` for implementation loops:

```bash
/chainer:run implement-only --spec_file="audit/api.md"
```

## Advanced Usage

### Working Directory

Run a chain in a specific directory:

```bash
/chainer:run plan-and-implement \
  --cwd="~/worktrees/oauth" \
  --prompt="OAuth" \
  --feature_name="oauth"
```

### Environment Variables

Reference environment variables in chains:

```yaml
steps:
  - name: deploy
    type: script
    script: |
      export API_KEY={{env.API_KEY}}
      ./deploy.sh
```

## Configuration Locations

Chainer looks for configuration in this order:

1. `.claude/chainer.local.md` (project-specific)
2. `~/.claude/chainer.local.md` (global)
3. Plugin defaults

## Community Chains

Share and discover chains in `community-chains/`:

```
community-chains/
├── development/
│   ├── plan-and-implement.yaml
│   ├── tdd-feature.yaml
│   └── design-and-build.yaml
├── content/
│   ├── research-to-deck.yaml
│   └── video-to-doc.yaml
└── marketing/
    └── landing-page.yaml
```

Import from URL:
```bash
# In settings.html
Import → From URL → https://raw.githubusercontent.com/user/repo/main/chain.yaml
```

## Examples

### Full Feature Development

```bash
/chainer:run plan-and-implement \
  --prompt="Add user authentication with OAuth" \
  --feature_name="auth"
```

This will:
1. Plan the feature with `feature-dev`
2. Save spec to `audit/auth.md`
3. Implement with `ralph-wiggum` loop
4. Iterate until complete

### Quick Implementation

Already have a spec? Skip planning:

```bash
/chainer:run implement-only --spec_file="audit/auth.md"
```

### Planning Only

Just want to plan without implementing?

```bash
/chainer:run plan-only \
  --prompt="Design payment system" \
  --feature_name="payments"
```

## Troubleshooting

### Chain not found

```
❌ Chain 'my-chain' not found
```

**Fix**: Check chain name with `/chainer:list` or enable it in config

### Missing plugin

```
❌ Plugin 'feature-dev' not found
```

**Fix**: Install the required plugin

### Missing required input

```
❌ Missing required input: prompt
```

**Fix**: Provide all required inputs: `--prompt="value"`

## Development

Chainer is part of a two-plugin system:

- **Worktree Manager**: Pure git worktree operations
- **Chainer**: Universal plugin orchestration

See [FEATURE-PLAN-CHAINER-SPLIT.md](https://github.com/danielraffel/worktree-manager/blob/main/FEATURE-PLAN-CHAINER-SPLIT.md) for architecture details.

## Roadmap

- **v0.1** (Current): Config-driven chains, visual editor
- **v0.2** (Phase 4): Parallel execution, status tracking, tmux integration
- **v0.3** (Phase 5): Inline pipe syntax, import/export, community chains
- **v1.0** (Phase 6): Production ready, comprehensive docs

## Contributing

Contributions welcome! See `community-chains/` for examples of shareable chains.

## License

MIT License - see LICENSE file for details

## Credits

Built by [Daniel Raffel](https://github.com/danielraffel) for the Claude Code community.

## Links

- [Website](https://danielraffel.github.io/Chainer)
- [GitHub](https://github.com/danielraffel/Chainer)
- [Worktree Manager](https://github.com/danielraffel/worktree-manager)
- [Feature Plan](https://github.com/danielraffel/worktree-manager/blob/main/FEATURE-PLAN-CHAINER-SPLIT.md)
