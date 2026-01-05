# Chainer - Project Context

## Overview

Chainer is a Claude Code plugin for universal plugin orchestration with piping. It allows users to chain together skills, MCP tools, and bash scripts in configurable sequences.

## Repository Location

`/Users/danielraffel/Code/Chainer`

## Related Projects

- **Worktree Manager**: `/Users/danielraffel/Code/worktree-manager` (companion plugin)
- **Feature Plan**: `/Users/danielraffel/Code/worktree-manager/FEATURE-PLAN-CHAINER-SPLIT.md`

## Current Phase

**Phase 1: Chainer MVP** (Not Started)

## Key Files (To Be Created)

```
Chainer/
├── plugin/
│   ├── .claude-plugin/
│   │   └── plugin.json         # Plugin manifest
│   └── commands/
│       ├── run.md              # /chainer:run command
│       ├── list.md             # /chainer:list command
│       └── status.md           # /chainer:status command
├── defaults/
│   └── chainer.local.md        # Default chain configurations
├── community-chains/           # Shareable chain library
├── settings.html               # Visual chain editor
├── index.html                  # Marketing webpage
└── README.md
```

## Design Decisions

1. **Config-driven**: Chains are defined in YAML, not hardcoded
2. **Settings page**: HTML-based visual editor for chains
3. **Step types**: skill, script, mcp, prompt, wait
4. **Enable/disable**: Chains can be toggled without deletion
5. **Import/export**: Chains can be shared via GitHub URLs

## Integration with Worktree Manager

- Chainer provides orchestration (what to do)
- Worktree Manager provides isolation (where to do it)
- They work together via `--cwd` parameter and worktree-plan-implement chain

## Testing Requirements

- Unit tests: config-parser, variable-resolver, chain-executor
- Integration tests: Full chain execution end-to-end
- Settings page: Manual testing of UI interactions

## Notes

- Keep chains simple in v0.1 (no conditionals, no loops)
- tmux support comes in v0.2 (Phase 4)
- IDE integration deferred to post-v1.0
