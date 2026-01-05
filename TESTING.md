# Chainer Testing Guide

## Overview

Chainer is a command-based Claude Code plugin (no TypeScript/MCP server), so testing is primarily integration testing by running the actual commands.

## Test Environment Setup

1. Install Chainer plugin:
```bash
ln -s /Users/danielraffel/Code/Chainer/plugin ~/.claude/plugins/chainer
```

2. Copy default configuration:
```bash
cp /Users/danielraffel/Code/Chainer/defaults/chainer.local.md ~/.claude/chainer.local.md
```

3. Install required plugins:
- `feature-dev` (for planning)
- `ralph-wiggum` (for implementation)

## Integration Tests

### Test 1: List Chains

**Command:**
```bash
/chainer:list
```

**Expected Output:**
- Shows all three default chains: `plan-and-implement`, `plan-only`, `implement-only`
- Each chain shows description, inputs, and steps
- Configuration file location displayed

**Pass Criteria:**
- ✅ All three chains listed
- ✅ Descriptions are correct
- ✅ Input requirements shown
- ✅ Step counts accurate

---

### Test 2: Run plan-and-implement (Full Chain)

**Command:**
```bash
/chainer:run plan-and-implement \
  --prompt="Build a simple counter component" \
  --feature_name="counter"
```

**Expected Behavior:**
1. Validates required inputs provided
2. Executes step 1: feature-dev planning
3. Creates `audit/counter.md` spec file
4. Executes step 2: ralph-wiggum implementation
5. Shows completion message

**Pass Criteria:**
- ✅ No errors about missing inputs
- ✅ feature-dev skill executes
- ✅ ralph-wiggum skill executes
- ✅ Spec file created at `audit/counter.md`

---

### Test 3: Run plan-only

**Command:**
```bash
/chainer:run plan-only \
  --prompt="Design a search feature" \
  --feature_name="search"
```

**Expected Behavior:**
1. Validates required inputs
2. Executes feature-dev planning only
3. Does NOT start ralph-wiggum
4. Shows completion message

**Pass Criteria:**
- ✅ feature-dev executes
- ✅ ralph-wiggum does NOT execute
- ✅ Spec file created
- ✅ Chain stops after planning

---

### Test 4: Run implement-only

**Setup:**
Create a test spec file first:
```bash
mkdir -p audit
echo "# Test Feature\n\nImplement a test feature." > audit/test-impl.md
```

**Command:**
```bash
/chainer:run implement-only --spec_file="audit/test-impl.md"
```

**Expected Behavior:**
1. Skips planning step
2. Executes ralph-wiggum with spec file
3. Starts implementation loop

**Pass Criteria:**
- ✅ No planning step executed
- ✅ ralph-wiggum starts with correct spec file
- ✅ Implementation begins

---

### Test 5: Missing Required Input

**Command:**
```bash
/chainer:run plan-and-implement --prompt="Test"
# Missing --feature_name
```

**Expected Output:**
```
❌ Missing required input: feature_name

Required inputs for 'plan-and-implement':
  • prompt: What to build
  • feature_name: Feature name for spec file

Usage:
  /chainer:run plan-and-implement \
    --prompt="Your idea" \
    --feature_name="feature-name"
```

**Pass Criteria:**
- ✅ Error message displayed
- ✅ Lists missing input
- ✅ Shows usage example
- ✅ Chain does NOT execute

---

### Test 6: Invalid Chain Name

**Command:**
```bash
/chainer:run nonexistent-chain
```

**Expected Output:**
```
❌ Chain 'nonexistent-chain' not found

Available chains:
  • plan-and-implement
  • plan-only
  • implement-only

Use /chainer:list to see details
```

**Pass Criteria:**
- ✅ Error message displayed
- ✅ Lists available chains
- ✅ Helpful suggestion provided

---

### Test 7: Disabled Chain

**Setup:**
1. Edit `~/.claude/chainer.local.md`
2. Set `implement-only.enabled: false`

**Command:**
```bash
/chainer:run implement-only --spec_file="audit/test.md"
```

**Expected Output:**
```
❌ Chain 'implement-only' is disabled

To enable it:
  1. Edit ~/.claude/chainer.local.md
  2. Set implement-only.enabled: true
  3. Try again
```

**Pass Criteria:**
- ✅ Error about disabled chain
- ✅ Helpful instructions provided
- ✅ Chain does NOT execute

---

### Test 8: Variable Substitution

**Setup:**
Create custom chain in `~/.claude/chainer.local.md`:

```yaml
test-vars:
  enabled: true
  description: "Test variable substitution"
  inputs:
    name: { required: true }
  steps:
    - name: echo-name
      type: script
      script: echo "Hello, {{name}}!"
```

**Command:**
```bash
/chainer:run test-vars --name="World"
```

**Expected Output:**
```
Hello, World!
```

**Pass Criteria:**
- ✅ Variable `{{name}}` replaced with "World"
- ✅ Script executes correctly
- ✅ Output shows substituted value

---

### Test 9: Working Directory (--cwd)

**Command:**
```bash
/chainer:run plan-only \
  --cwd="/tmp/test-dir" \
  --prompt="Test" \
  --feature_name="test"
```

**Expected Behavior:**
1. Changes to `/tmp/test-dir` before execution
2. Creates `audit/test.md` in `/tmp/test-dir/audit/`
3. Returns to original directory after

**Pass Criteria:**
- ✅ Spec file created in correct directory
- ✅ Current directory restored after execution

---

### Test 10: Status Command

**Command:**
```bash
/chainer:status
```

**Expected Output (v0.1):**
```
No chains currently running

Available chains:
  /chainer:list      Show available chains
  /chainer:run       Execute a chain
```

**Pass Criteria:**
- ✅ Shows "No chains currently running" (status tracking is Phase 4)
- ✅ Helpful suggestions provided

---

## Visual Settings Editor Tests

### Test 11: Load Settings Page

**Steps:**
1. Open `settings.html` in browser
2. Verify default chains load

**Pass Criteria:**
- ✅ Page loads without errors
- ✅ Sidebar shows three default chains
- ✅ All chains have checkboxes (enabled)

---

### Test 12: Edit Chain Description

**Steps:**
1. Click on `plan-only` chain
2. Edit description field
3. Change to "Modified description"
4. Click "Save Config"

**Pass Criteria:**
- ✅ Editor shows chain details
- ✅ Description field editable
- ✅ Download triggered with modified config
- ✅ YAML file contains new description

---

### Test 13: Add New Input

**Steps:**
1. Select `plan-only` chain
2. Click "+ Add Input"
3. Enter name: `max_iterations`
4. Set required: true
5. Set description: "Max iterations"

**Pass Criteria:**
- ✅ New input appears in editor
- ✅ Checkbox for required works
- ✅ Save includes new input in YAML

---

### Test 14: Add New Step

**Steps:**
1. Select `plan-only` chain
2. Click "+ Add Step"
3. Name: `test`
4. Type: `script`
5. Script: `npm test`

**Pass Criteria:**
- ✅ New step appears in steps list
- ✅ Step type dropdown works
- ✅ Script textarea editable
- ✅ Save includes new step in YAML

---

### Test 15: Delete Chain

**Steps:**
1. Uncheck `implement-only` checkbox
2. Click "Save Config"

**Pass Criteria:**
- ✅ Chain shows as disabled (unchecked)
- ✅ YAML file has `enabled: false`
- ✅ `/chainer:list` doesn't show disabled chain

---

### Test 16: Export Single Chain

**Steps:**
1. Select `plan-and-implement`
2. Click "Export ▼"
3. Choose "Current Chain"

**Pass Criteria:**
- ✅ Downloads YAML file
- ✅ File named `plan-and-implement.yaml`
- ✅ Contains only that chain's config

---

## Configuration Parser Tests

Since Chainer commands parse YAML, test various YAML formats:

### Test 17: Minimal Chain

**Config:**
```yaml
---
chains:
  minimal:
    enabled: true
    steps:
      - name: test
        type: script
        script: echo "test"
---
```

**Command:**
```bash
/chainer:run minimal
```

**Pass Criteria:**
- ✅ Parses successfully
- ✅ No inputs required
- ✅ Executes script step

---

### Test 18: Complex Variable Substitution

**Config:**
```yaml
multi-var:
  enabled: true
  inputs:
    prefix: { required: true }
    suffix: { required: true }
  steps:
    - name: combined
      type: script
      script: echo "{{prefix}}-middle-{{suffix}}"
```

**Command:**
```bash
/chainer:run multi-var --prefix="start" --suffix="end"
```

**Expected Output:**
```
start-middle-end
```

**Pass Criteria:**
- ✅ Multiple variables substituted correctly
- ✅ Variables can appear multiple times

---

## Error Handling Tests

### Test 19: Missing Plugin

**Setup:**
Temporarily rename feature-dev plugin

**Command:**
```bash
/chainer:run plan-only --prompt="Test" --feature_name="test"
```

**Expected Output:**
```
❌ Chain 'plan-only' failed at step 1: plan

Error: Skill 'feature-dev:feature-dev' not found

To fix:
  - Install feature-dev plugin
  - Check plugin is enabled
  - Verify plugin name is correct
```

**Pass Criteria:**
- ✅ Clear error message
- ✅ Identifies failing step
- ✅ Provides troubleshooting steps

---

### Test 20: Script Failure

**Config:**
```yaml
failing-script:
  enabled: true
  steps:
    - name: fail
      type: script
      script: exit 1
```

**Command:**
```bash
/chainer:run failing-script
```

**Expected Output:**
```
❌ Chain 'failing-script' failed at step 1: fail

Error: Script exited with code 1

Step details:
  Name: fail
  Type: script
  Script: exit 1
```

**Pass Criteria:**
- ✅ Execution stops at failing step
- ✅ Error code reported
- ✅ Step details shown

---

## Test Summary

Run all tests and track results:

| Test # | Test Name | Status | Notes |
|--------|-----------|--------|-------|
| 1 | List Chains | ⬜ | |
| 2 | Run plan-and-implement | ⬜ | |
| 3 | Run plan-only | ⬜ | |
| 4 | Run implement-only | ⬜ | |
| 5 | Missing Required Input | ⬜ | |
| 6 | Invalid Chain Name | ⬜ | |
| 7 | Disabled Chain | ⬜ | |
| 8 | Variable Substitution | ⬜ | |
| 9 | Working Directory | ⬜ | |
| 10 | Status Command | ⬜ | |
| 11 | Load Settings Page | ⬜ | |
| 12 | Edit Chain Description | ⬜ | |
| 13 | Add New Input | ⬜ | |
| 14 | Add New Step | ⬜ | |
| 15 | Delete Chain | ⬜ | |
| 16 | Export Single Chain | ⬜ | |
| 17 | Minimal Chain | ⬜ | |
| 18 | Complex Variables | ⬜ | |
| 19 | Missing Plugin | ⬜ | |
| 20 | Script Failure | ⬜ | |

## Notes

- Chainer v0.1 focuses on basic chain execution
- Status tracking and tmux integration come in Phase 4
- These tests verify core functionality: parsing, validation, execution, error handling
