---
name: Baseline Exploration Before Planning
description: Verify filesystem reality before creating implementation plans to prevent false assumptions
when_to_use: MANDATORY before brainstorming plans, writing plans, or delegating to Codex
version: 1.0.0
---

# Baseline Exploration Before Planning

## Core Principle

**Never trust documentation over filesystem reality.**

Documentation describes intent. The filesystem describes truth. Before planning any implementation, verify what actually exists.

## When to Use (MANDATORY)

Use this skill BEFORE:
- Brainstorming implementation approaches
- Writing implementation plans
- Delegating to Codex with architectural context
- Creating tasks that assume file existence/structure

DO NOT skip this even when:
- Documentation seems clear and comprehensive
- You "remember" the codebase structure
- The task seems simple
- Time pressure exists

## Quick Reference

```bash
# 1. List actual files (not what docs say exists)
find . -type f | head -20
ls -la target_directory/

# 2. Check file formats
find . -name "*.yaml" -o -name "*.yml"
find . -name "*.json"

# 3. Inspect existing tooling
ls -la scripts/
wc -l scripts/*.py

# 4. Read sample files
cat path/to/actual/file.json | jq '.'

# 5. Compare to documentation claims
grep -r "YAML" docs/
```

## The Mistake Pattern

### What Happened (Real Example)

**Documentation said:**
> "Current State: Repository uses YAML source files (legacy)"
> "Need to migrate YAML → JSON"

**Reality was:**
- 0 YAML files existed
- 8 JSON files already in target format
- Migration scripts existed but referenced non-existent YAML

**Cost:**
- Gave Codex wrong context (30-task plan for non-existent migration)
- Wasted time creating file-creation tasks for existing files
- Would have failed immediately if executed

### Root Cause

**Violated "Exploration Before Execution" principle:**
1. Read documentation → 2. Assumed it was current → 3. Planned based on assumptions

**Should have been:**
1. Read documentation → 2. **VERIFY with filesystem** → 3. Plan based on reality

## Implementation

### Step 1: Document What Docs Claim

Before exploring, capture what you THINK is true:

```markdown
## Assumptions from Documentation
- YAML files in registry/
- Need to create validate.py
- compile.py compiles YAML → JSON
```

### Step 2: Verify Each Assumption

For EACH assumption, run verification command:

```bash
# Assumption: "YAML files in registry/"
find registry/ -name "*.yaml" -o -name "*.yml"
# Result: (no output) = WRONG ASSUMPTION

# Assumption: "Need to create validate.py"
ls -la scripts/validate.py
# Result: -rwxr-xr-x ... validate.py = FILE EXISTS

# Assumption: "compile.py compiles YAML"
head -20 scripts/compile.py
# Result: Contains YAML parsing code, but...
find registry/ -type f | head -10
# Result: All .json and .md files
```

### Step 3: Document Assumptions vs Reality

Create explicit gap analysis:

```markdown
## Assumptions vs Reality

| What I Assumed | What Exists | Impact |
|---|---|---|
| YAML files in registry/ | JSON files exist instead | Scope changed: split JSON, not migrate YAML |
| Need to create validate.py | validate.py already exists (126 lines) | Scope changed: update, not create |
| compile.py actively used | compile.py references non-existent YAML | Tool is dead code, use build_compiled.py instead |
```

### Step 4: Update Context Before Planning

When delegating or planning, use VERIFIED reality:

```markdown
## Context (VERIFIED via baseline exploration)

**Current State:**
- 8 capabilities in JSON format: `registry/{domain}/{action}.v1.json`
- Format: `{ref, version, parameters, returns, discovery, runtime}` (flat)
- Active scripts: `build_compiled.py` (50 lines), `validate_capabilities.py` (152 lines)
- Dead code: `compile.py` (references non-existent YAML)

**NOT TRUE (despite documentation):**
- ~~YAML files in registry/~~ (0 found)
- ~~Need to create validate.py~~ (already exists)
```

## Integration with Other Skills

### With Brainstorming

```markdown
1. User: "Brainstorm how to add capabilities"
2. Read brainstorming skill
3. **BEFORE presenting options**: Run baseline exploration
4. Present options based on verified reality
5. User selects option
6. **BEFORE writing plan**: Re-verify assumptions from selected option
```

### With Writing Plans

```markdown
1. Design approved, ready to write plan
2. Read writing-plans skill
3. **BEFORE breaking down tasks**: Run baseline exploration
4. Verify every "create X file" assumption
5. Write plan using "update X" instead of "create X" where verified
```

### With Delegating to Codex

```markdown
1. Decide to delegate complex planning to Codex
2. Read delegating-to-codex skill
3. **BEFORE crafting Codex prompt**: Run baseline exploration
4. Include VERIFIED reality in Context section (not documentation claims)
5. Codex generates plan based on truth, not assumptions
```

## Checklist

Before ANY planning session, verify:

- [ ] Actual file formats (YAML vs JSON vs other)
- [ ] Existing scripts and their line counts
- [ ] File structure (what directories actually exist)
- [ ] Sample file contents (read at least 1-2 examples)
- [ ] Git history of recent changes (what's been updated)
- [ ] Dependency files (requirements.txt, package.json)

Mark each with actual command outputs, not assumptions.

## Red Flags That You Skipped This

- "According to the documentation..."
- "The YAML files need migration..."
- "We need to create validate.py..."
- Planning tasks without seeing file listings
- Assuming file existence based on docs
- Delegating to Codex with documentation-based context

## Why This Matters

**Time Cost:**
- 5 minutes to verify reality
- vs 30 minutes to create wrong plan + discover error + re-plan

**Quality Cost:**
- Plans based on assumptions = brittle, error-prone
- Plans based on verification = executable first time

**Trust Cost:**
- User catches your wrong assumptions = "didn't check the basics"
- You catch via exploration = "thorough, reliable"

## Summary

**The Rule:**
Filesystem > Documentation > Memory > Assumptions

**The Practice:**
1. Capture what you THINK is true (from docs)
2. Verify EVERY assumption with commands
3. Document gaps (assumptions vs reality)
4. Plan based on verified reality only

**The Integration:**
- Mandatory before brainstorming
- Mandatory before writing-plans
- Mandatory before delegating-to-codex

**The Cost of Skipping:**
Every minute saved skipping exploration = 10 minutes lost fixing wrong plans.
