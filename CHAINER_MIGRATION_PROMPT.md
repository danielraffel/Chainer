# AI Prompt: Migrate Chainer Plugin to Marketplace with Bundled Dependencies

## Context

The Chainer plugin needs to be migrated to use marketplace distribution with a bundled MCP server (zero build steps for users). Follow the same pattern used for worktree-manager plugin.

## Tasks

### 1. Repository Restructure

**Move plugin files from subdirectory to root**:

```bash
# If plugin files are currently in plugin/ subdirectory:
git mv plugin/.claude-plugin .claude-plugin
git mv plugin/commands commands
git rm -rf plugin/
```

**Update `.claude-plugin/plugin.json`**:
- Change MCP server path from `../mcp-server/dist/index.js` to `mcp-server/dist/index.js`
- Remove any unsupported fields like `category`
- Ensure it follows this structure:

```json
{
  "name": "chainer",
  "version": "2.0.0",
  "description": "Universal plugin orchestration for Claude Code",
  "author": {
    "name": "Daniel Raffel",
    "url": "https://danielraffel.me"
  },
  "homepage": "https://www.generouscorp.com/Chainer/",
  "repository": "https://github.com/danielraffel/chainer",
  "license": "MIT",
  "keywords": ["orchestration", "workflow", "automation", "plugin-orchestration"],
  "commands": "./commands/",
  "mcpServers": {
    "chainer": {
      "type": "stdio",
      "command": "node",
      "args": [
        "${CLAUDE_PLUGIN_ROOT}/mcp-server/dist/index.js"
      ]
    }
  }
}
```

### 2. Bundle MCP Server with esbuild

**Install esbuild**:
```bash
cd mcp-server
npm install --save-dev esbuild
```

**Update `mcp-server/package.json` build script**:
```json
{
  "scripts": {
    "build": "npm run clean && esbuild src/index.ts --bundle --platform=node --format=cjs --outfile=dist/index.js --external:fsevents",
    "clean": "rm -rf dist"
  }
}
```

**Build and test**:
```bash
npm run build

# Test bundle works standalone
mkdir /tmp/test-chainer
cp dist/index.js /tmp/test-chainer/
cd /tmp/test-chainer
node index.js
# Should start MCP server without errors
```

### 3. Update .gitignore

Ensure `mcp-server/dist/` is committed (not ignored):

```gitignore
# Build artifacts
# Note: mcp-server/dist/ is committed for easier plugin installation
build/
*.tsbuildinfo
```

### 4. Update worktree-manager Marketplace to Include Chainer

**In worktree-manager repository**, update `.claude-plugin/marketplace.json`:

```json
{
  "name": "worktree-manager-marketplace",
  "owner": {
    "name": "Generous Corp",
    "email": "dev@generouscorp.com"
  },
  "metadata": {
    "description": "Git worktree management and workflow orchestration tools for Claude Code",
    "version": "1.0.0"
  },
  "plugins": [
    {
      "name": "worktree-manager",
      "source": {
        "source": "url",
        "url": "https://github.com/danielraffel/worktree-manager.git"
      },
      "description": "Effortless git worktrees for parallel development with automatic environment setup",
      "version": "2.0.0",
      "author": {
        "name": "Daniel Raffel",
        "url": "https://danielraffel.me"
      },
      "homepage": "https://www.generouscorp.com/worktree-manager/",
      "repository": "https://github.com/danielraffel/worktree-manager",
      "license": "MIT",
      "keywords": ["git", "worktree", "version-control", "productivity", "parallel-development"]
    },
    {
      "name": "chainer",
      "source": {
        "source": "url",
        "url": "https://github.com/danielraffel/chainer.git"
      },
      "description": "Universal plugin orchestration for Claude Code",
      "version": "2.0.0",
      "author": {
        "name": "Daniel Raffel",
        "url": "https://danielraffel.me"
      },
      "homepage": "https://www.generouscorp.com/Chainer/",
      "repository": "https://github.com/danielraffel/chainer",
      "license": "MIT",
      "keywords": ["orchestration", "workflow", "automation", "plugin-orchestration"]
    }
  ]
}
```

### 5. Update Installation Documentation

**README.md** - Replace installation section:

```markdown
## Installation

### 1. Add the Marketplace

In Claude Code, run:

```bash
/plugin marketplace add danielraffel/worktree-manager
```

### 2. Install the Plugin

```bash
/plugin install chainer@worktree-manager-marketplace
```

Or use the interactive installer:
1. Type `/plugin`
2. Navigate to "worktree-manager-marketplace"
3. Select "chainer"
4. Click "Install for you (user scope)"

### 3. Restart Claude Code

Quit and reopen Claude Code to load the plugin.

### 4. Verify Installation

```bash
/plugin list
# Should show chainer

/chainer:run --help
# Test a command
```
```

**index.html** - Update installation section:

```html
<div class="installation-steps">
  <h3>Quick Install</h3>
  <pre><code><span class="code-comment"># 1. Add the marketplace (in Claude Code)</span>
/plugin marketplace add danielraffel/worktree-manager

<span class="code-comment"># 2. Install Chainer</span>
/plugin install chainer@worktree-manager-marketplace

<span class="code-comment"># 3. Restart Claude Code</span>
<span class="code-comment"># Quit and reopen Claude Code to load the plugin</span>
</code></pre>
</div>
```

**Update FAQ section** (if present):

Replace any `--plugin-dir` installation instructions with marketplace installation. Change questions like:

OLD:
```
Q: How do I install plugins?
A: Use --plugin-dir flag when starting Claude Code
```

NEW:
```
Q: How do I install plugins?
A: Use the marketplace system:
   1. /plugin marketplace add danielraffel/worktree-manager
   2. /plugin install chainer@worktree-manager-marketplace
   3. Restart Claude Code
```

### 6. Commit Changes

**Chainer repository**:
```bash
git add .
git commit -m "Migrate to marketplace distribution with bundled dependencies

- Restructure: Move plugin files from subdirectory to repository root
- Bundle MCP server dependencies with esbuild (zero build steps for users)
- Commit bundled dist/index.js to repository
- Update installation docs to use marketplace
- Remove --plugin-dir references from documentation"

git push origin main
```

**worktree-manager repository** (to add Chainer to marketplace):
```bash
git add .claude-plugin/marketplace.json
git commit -m "Add Chainer plugin to marketplace"
git push origin main
```

### 7. Testing

**Test the full installation flow**:

```bash
# In Claude Code:
/plugin marketplace update
/plugin install chainer@worktree-manager-marketplace

# Restart Claude Code

# Verify:
/plugin list
# Should show: chainer

# Test a command:
/chainer:run plan-and-implement --help
# Should connect to MCP server (not fall back to bash)
```

## Success Criteria

- [ ] Plugin files at repository root (not in subdirectory)
- [ ] MCP server bundled into single `dist/index.js` file
- [ ] Bundled dist/ committed to repository
- [ ] Chainer added to worktree-manager marketplace
- [ ] README.md uses marketplace installation
- [ ] index.html uses marketplace installation
- [ ] No `--plugin-dir` references in documentation
- [ ] Installation requires zero build steps
- [ ] MCP server connects successfully (no bash fallback)

## Expected User Experience

**Before** (manual installation):
```bash
git clone https://github.com/danielraffel/chainer.git --plugin-dir=...
cd chainer/mcp-server
npm install
npm run build
# Restart Claude Code
```

**After** (marketplace installation):
```bash
/plugin marketplace add danielraffel/worktree-manager
/plugin install chainer@worktree-manager-marketplace
# Restart Claude Code
```

## Notes

- Use HTTPS URLs in marketplace.json (not SSH or GitHub shorthand)
- Bundle must include all dependencies (test without node_modules)
- Both worktree-manager AND chainer repositories need updates
- Users will install from the same marketplace for both plugins
- Marketplace update adds Chainer entry to worktree-manager's marketplace.json
