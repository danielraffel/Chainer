# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Chainer is a Claude Code plugin for universal plugin orchestration with piping. It allows users to chain together skills, MCP tools, and bash scripts in configurable sequences.

## Architecture

Chainer is a config-driven plugin system with these core components:

- **Plugin Commands** (`plugin/commands/`): Slash commands `/chainer:run`, `/chainer:list`, `/chainer:status`
- **Plugin Manifest** (`plugin/.claude-plugin/plugin.json`): Plugin configuration
- **Chain Configs** (`defaults/chainer.local.md`): YAML-defined chain configurations
- **Community Chains** (`community-chains/`): Shareable chain library
- **Settings UI** (`settings.html`): Visual HTML-based chain editor

### Step Types

Chains support five step types:
1. `skill` - Execute a Claude Code skill
2. `script` - Run a bash script
3. `mcp` - Call an MCP tool
4. `prompt` - Send a prompt
5. `wait` - Wait for a condition

### Design Principles

- Chains are defined in YAML configuration, not hardcoded
- Chains can be enabled/disabled without deletion
- Chains can be imported/exported via GitHub URLs
- v0.1 keeps chains simple (no conditionals, no loops)

## Related Projects

- **Worktree Manager**: Companion plugin at `/Users/danielraffel/Code/worktree-manager`
  - Chainer provides orchestration (what to do)
  - Worktree Manager provides isolation (where to do it)
  - Integration via `--cwd` parameter

## Claude Collaboration Guidelines

### Operating Mode

Claude operates **locally only** in this project:

- You have access to the full codebase and can execute and test the app
- You may launch Xcode and interact with the running environment
- If I send you a URL, fetch its contents and read it before continuing
- All builds and tests happen on my machine

### GitHub Integration

Even though cloud builds are disabled, you can still:

- Respond to `@claude` mentions in GitHub issues and pull requests
- Fetch GitHub issue and PR content to understand feature requests or bugs
- Propose plans or implementations based on the content
- Suggest branches (e.g., `claude/feature-name`) and write code
- Summarize diffs, test plans, or intentions in GitHub comments

---
## Running Log of Learnings

To avoid repeating mistakes or re-solving tricky problems, we maintain a shared log of learnings in:
`ai/learnings.txt`

Add a short entry to this file whenever:
- You solve a non-trivial or time-consuming issue
- We encounter tricky behavior or workarounds
- You discover an undocumented detail that’s important to know

This log helps us (and Claude) avoid wasted time in the future. Keep entries brief, focused, and updated as we learn more.

Tip: Prefer updating this log after a successful build or test pass. This allows you to confirm the solution worked, avoid delaying testing, and ensures the entry is accurate.

Note: `ai/learnings.txt` can be slow to read due to its size or formatting. Consider:
- Keeping entries as short and clear as possible
- Avoiding excessive formatting
- Reviewing only the most recent entries unless necessary
---

## Development Workflow

We use GitHub issues to track work and PRs to review code. Tag Claude-created issues/PRs with `by-claude`. Use the gh bash command to interact with GitHub.

### Setup
- Read the relevant GitHub issue (or create one)
- Checkout `main` and pull latest changes
- Create a new branch: `claude/feature-name`
- **CRITICAL**: Never commit or push directly to `main`
- If working on `main`, always create a feature branch first

### Development

- Claude should commit early and often with clear messages
- Claude should create commits when:
  - A feature is functionally complete
  - A bug fix is implemented
  - Significant progress is made
  - Before switching to a different task
- Commit messages should be descriptive (e.g., "Add reverb parameter controls to audio engine")
- **After committing to feature branches, push to origin: `git push origin branch-name`**
- **Branch Protection**:
  - If on `main` branch: Create a feature branch before any commits
  - Never commit directly to `main`
  - If accidentally on `main` with uncommitted changes: Stop and ask for guidance
- Ask me to test in the app as needed

### Git Safety Rules

1. **Before any commit**, Claude must check current branch: `git branch --show-current`
2. **If on main branch**:
   - DO NOT commit
   - Create a feature branch: `git checkout -b claude/feature-description`
   - Then proceed with commits
3. **If on feature branch**: Commit and push freely
4. **If unsure**: Ask user before proceeding

### Review

- Run `git diff main` to review changes
- Push the branch to GitHub
- Open a PR with:
  - A short title (no issue number)
  - A body starting with the issue number and a description of the changes
  - A test plan covering:
    - New functionality
    - Any existing behavior that might have changed
- Claude will:
  - Generate a test plan based on the issue and changes (if not already provided)
  - Execute available tests (when running locally or with CLI access)
  - Report test results as a comment or PR review, including:
    - Which tests were run
    - Any failures or regressions detected
    - Confirmation of success where appropriate

 ### Examining Old Commits Safely

  When asked to look at old commits:
  - **NEVER use `git checkout <commit-hash>`** - this detaches HEAD
  - Instead use:
    - `git show <commit>:path/to/file` - to read a file from that commit
    - `git diff <commit> -- path/to/file` - to compare with current version
    - `git log -p <commit> -1` - to see what changed in that commit

  If accidentally in detached HEAD state:
  1. Check with `git status`
  2. Discard changes: `git checkout -- .`
  3. Return to branch: `git checkout <branch-name>`

### Fixes

- Use `rebase` or `cherry-pick` to reconcile branches—**never** merge to `main`