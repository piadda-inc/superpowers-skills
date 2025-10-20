---
name: Writing Plans
description: Create detailed implementation plans with bite-sized tasks for engineers with zero codebase context
when_to_use: when design is complete and you need detailed implementation tasks for engineers with zero codebase context
version: 2.2.0
---

# Writing Plans

## Overview

Write comprehensive implementation plans assuming the engineer has zero context for our codebase and questionable taste. Document everything they need to know: which files to touch for each task, code, testing, docs they might need to check, how to test it. Give them the whole plan as bite-sized tasks. DRY. YAGNI. TDD. Frequent commits.

Assume they are a skilled developer, but know almost nothing about our toolset or problem domain. Assume they don't know good test design very well.

**Announce at start:** "I'm using the Writing Plans skill to create the implementation plan."

**Context:** This should be run in a dedicated worktree (created by brainstorming skill).

**Save plans to:** `docs/plans/YYYY-MM-DD-<feature-name>.md`

## Planning Approach: You vs. Codex

**Choose the right planner for the job:**

| Complexity | Your Approach | Codex Approach | Recommendation |
|------------|--------------|----------------|----------------|
| **Simple** (1-3 files, <10 tasks) | Fast, direct | Overkill | **You write the plan** |
| **Medium** (3-7 files, 10-30 tasks) | Manageable | Thorough | **Either works** |
| **Complex** (7+ files, 30+ tasks) | Time-consuming | Excels | **Delegate to Codex** |
| **Ultra-complex** (10+ files, 50+ tasks) | Challenging | Ideal | **Definitely Codex** |

### Option A: Codex-Generated Plan (Recommended for Complex Features)

**When to use Codex:**
- Feature spans 7+ files
- Implementation requires 30+ bite-sized tasks
- Complex integration with existing codebase
- Need exhaustive edge case coverage
- Want maximum detail for less experienced engineer

**How to delegate plan writing to Codex:**

**Step 1: Prepare comprehensive context**
```
Gather from brainstorming:
- Design document or conversation summary
- Architecture decisions
- File paths in codebase
- Tech stack and patterns
- Success criteria
- Constraints (DRY, YAGNI, TDD)
```

**Step 2: Invoke Codex via plugin with structured prompt**
```python
from codex_delegate import delegate

result = delegate(
    prompt="""
## Context
[Design summary from brainstorming - 2-3 paragraphs]

Tech Stack: [Python, FastAPI, Firestore, etc.]
Project: [Brief project description]

## Task
Create a comprehensive implementation plan following the Writing Plans skill format.

## Requirements
1. Break into bite-sized tasks (2-5 minutes each)
2. Follow TDD: Write test → Run to see fail → Implement → Run to see pass → Commit
3. Include exact file paths, complete code examples, exact commands
4. DRY, YAGNI principles throughout
5. Frequent commits (every task)

## Plan Structure
- Header with goal, architecture, tech stack
- Tasks with: Files, Step-by-step TDD cycle, Exact code, Exact commands
- Each task = one component or feature unit
- Reference relevant skills where applicable

## Codebase Context
Working directory: [path]
Key files:
- [list important existing files the plan should integrate with]

Pattern: [describe coding patterns to follow from existing codebase]

## Expected Output
Complete implementation plan in markdown ready to save to:
docs/plans/YYYY-MM-DD-<feature-name>.md
""",
    cwd="/path/to/project",
    sandbox="read-only",
    timeout=600  # Complex plans may take 5-10 minutes
)

plan_content = result.output
```

**Step 3: Validate Codex's plan**
```
Review for:
✓ Follows header format (Goal, Architecture, Tech Stack)
✓ Tasks are bite-sized (2-5 min each)
✓ TDD cycle in every task (test → fail → implement → pass → commit)
✓ Exact file paths (not "update validation logic")
✓ Complete code examples (not "add validation")
✓ Exact commands with expected output
✓ Aligns with design from brainstorming
✓ Covers edge cases
```

**Step 4: Save and present**
```
Save Codex's plan to docs/plans/YYYY-MM-DD-<feature-name>.md
Present to user with attribution:
"Codex generated a comprehensive N-task implementation plan. I've reviewed it and it covers..."
```

**Benefits of Codex-generated plans:**
- 40% more comprehensive (edge cases, error handling)
- Exhaustive task breakdown (less likely to miss steps)
- Consistent formatting throughout
- Saves 30-60 min for complex features

### Option B: Manual Plan Writing (You)

**When to write manually:**
- Simple features (<10 tasks)
- You have strong context from conversation
- Quick iterations needed
- User prefers your direct involvement

[Continue with manual plan structure below]

## Bite-Sized Task Granularity

**Each step is one action (2-5 minutes):**
- "Write the failing test" - step
- "Run it to make sure it fails" - step
- "Implement the minimal code to make the test pass" - step
- "Run the tests and make sure they pass" - step
- "Commit" - step

## Plan Document Header

**Every plan MUST start with this header:**

```markdown
# [Feature Name] Implementation Plan

> **For Claude:** Use `${SUPERPOWERS_SKILLS_ROOT}/skills/collaboration/executing-plans/SKILL.md` to implement this plan task-by-task.

**Goal:** [One sentence describing what this builds]

**Architecture:** [2-3 sentences about approach]

**Tech Stack:** [Key technologies/libraries]

---
```

## Task Structure

```markdown
### Task N: [Component Name]

**Files:**
- Create: `exact/path/to/file.py`
- Modify: `exact/path/to/existing.py:123-145`
- Test: `tests/exact/path/to/test.py`

**Step 1: Write the failing test**

```python
def test_specific_behavior():
    result = function(input)
    assert result == expected
```

**Step 2: Run test to verify it fails**

Run: `pytest tests/path/test.py::test_name -v`
Expected: FAIL with "function not defined"

**Step 3: Write minimal implementation**

```python
def function(input):
    return expected
```

**Step 4: Run test to verify it passes**

Run: `pytest tests/path/test.py::test_name -v`
Expected: PASS

**Step 5: Commit**

```bash
git add tests/path/test.py src/path/file.py
git commit -m "feat: add specific feature"
```
```

## Related Skills

**For complex plan generation:**
- skills/collaboration/delegating-to-codex - When and how to delegate plan writing to Codex

## Remember
- Consider Codex for complex features (30+ tasks, 7+ files)
- Validate Codex-generated plans against design requirements
- Exact file paths always
- Complete code in plan (not "add validation")
- Exact commands with expected output
- Reference relevant skills with @ syntax
- DRY, YAGNI, TDD, frequent commits

## Execution Handoff

After saving the plan, offer execution choice:

**"Plan complete and saved to `docs/plans/<filename>.md`. Two execution options:**

**1. Subagent-Driven (this session)** - I dispatch fresh subagent per task, review between tasks, fast iteration

**2. Parallel Session (separate)** - Open new session with executing-plans, batch execution with checkpoints

**Which approach?"**

**If Subagent-Driven chosen:**
- Use skills/collaboration/subagent-driven-development
- Stay in this session
- Fresh subagent per task + code review

**If Parallel Session chosen:**
- Guide them to open new session in worktree
- New session uses skills/collaboration/executing-plans
