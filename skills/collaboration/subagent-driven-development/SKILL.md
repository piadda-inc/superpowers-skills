---
name: Subagent-Driven Development
description: Execute implementation plan by dispatching fresh subagent for each task, with code review between tasks
when_to_use: when executing implementation plans with independent tasks in the current session, using fresh subagents with review gates
version: 1.1.0
---

# Subagent-Driven Development

Execute plan by dispatching fresh subagent per task, with code review after each.

**Core principle:** Fresh subagent per task + review between tasks = high quality, fast iteration

## Overview

**vs. Executing Plans (parallel session):**
- Same session (no context switch)
- Fresh subagent per task (no context pollution)
- Code review after each task (catch issues early)
- Faster iteration (no human-in-loop between tasks)

**When to use:**
- Staying in this session
- Tasks are mostly independent
- Want continuous progress with quality gates

**When NOT to use:**
- Need to review plan first (use executing-plans)
- Tasks are tightly coupled (manual execution better)
- Plan needs revision (brainstorm first)

## The Process

### 1. Load Plan

Read plan file, create TodoWrite with all tasks.

### 2. Analyze Task Dependencies

Before executing, identify which tasks are independent (can run in parallel):

**Independent tasks:**
- No shared files
- No data dependencies
- Don't depend on each other's completion

**Example:**
```
Task 1: Database schema (db/schema.sql)       } Independent
Task 2: API endpoints (api/routes.py)         } - can run in parallel
Task 3: Frontend components (ui/components/)  }

Task 4: Integration tests (tests/integration/) - Depends on 1,2,3 - sequential
```

### 3. Execute Tasks (Parallel Where Possible)

**Pattern A: Independent Tasks (Parallel Execution)**

For 2-3 independent tasks, dispatch in parallel:

```python
from codex_delegate import delegate

# Identify independent tasks
independent_tasks = [(1, "Database schema"), (2, "API endpoints"), (3, "Frontend")]

# Option 1: All Task tool agents
agents = []
for task_num, description in independent_tasks:
    agent = Task(
        subagent_type="general-purpose",
        description=f"Implement Task {task_num}: {description}",
        prompt=f"""
You are implementing Task {task_num} from [plan-file].

Read that task carefully. Your job is to:
1. Implement exactly what the task specifies
2. Write tests (following TDD if task says to)
3. Verify implementation works
4. Commit your work
5. Report back

Work from: [directory]

Report: What you implemented, what you tested, test results, files changed, any issues
"""
    )
    agents.append(agent)

# All agents run concurrently
results = [agent.wait() for agent in agents]
```

**Option 2: Mix of Task tool + Codex agents**

When one task needs deep analysis, use Codex in parallel:

```python
# Tasks 1 & 2: Standard implementation (Task tool)
agent1 = Task(subagent_type="general-purpose", description="Task 1: Database", ...)
agent2 = Task(subagent_type="general-purpose", description="Task 2: API", ...)

# Task 3: Complex analysis requiring deep investigation (Codex)
codex_agent = delegate(
    prompt="""
## Context
Read docs/plans/plan.md Task 3

## Task
Implement Task 3: Frontend components with comprehensive accessibility analysis

## Requirements
[Full task requirements from plan]

## Expected Output
- Implementation with tests
- Accessibility audit report
- Performance analysis
- Commit summary
""",
    cwd=project_path,
    sandbox="workspace-write",  # Needs to write code
    background=True,  # Parallel execution
    timeout=600
)

# Wait for all to complete
results = [agent1.wait(), agent2.wait(), codex_agent.wait()]
```

**Pattern B: Sequential Tasks**

For dependent tasks, dispatch one at a time:

```
Task tool (general-purpose):
  description: "Implement Task N: [task name]"
  prompt: |
    You are implementing Task N from [plan-file].

    Read that task carefully. Your job is to:
    1. Implement exactly what the task specifies
    2. Write tests (following TDD if task says to)
    3. Verify implementation works
    4. Commit your work
    5. Report back

    Work from: [directory]

    Report: What you implemented, what you tested, test results, files changed, any issues
```

**When to use parallel execution:**
- 2-3 truly independent tasks identified
- Tasks touch different files/modules
- No data dependencies between tasks
- Git conflicts unlikely

**When to stay sequential:**
- Tasks depend on each other
- High risk of merge conflicts
- Complex interdependencies
- Unsure about independence

### 4. Review Subagent's Work

**For parallel execution:** Review all tasks together for cross-task issues

**For sequential execution:** Review one task at a time

**Dispatch code-reviewer subagent(s):**

```python
# Option A: Single review of all parallel work
reviewer = Task(
    subagent_type="code-reviewer",
    prompt="""
Review Tasks 1, 2, 3 implemented in parallel.

Use template: skills/collaboration/requesting-code-review/code-reviewer.md

WHAT_WAS_IMPLEMENTED: [from all subagent reports]
PLAN_OR_REQUIREMENTS: Tasks 1-3 from [plan-file]
BASE_SHA: [commit before tasks]
HEAD_SHA: [current commit]
DESCRIPTION: [tasks summary]

**Cross-task checks:**
- Do implementations integrate correctly?
- Any conflicts or inconsistencies?
- Shared interfaces compatible?
"""
)

# Option B: Parallel reviews (faster for independent tasks)
reviewers = []
for task_num, base_sha, head_sha in task_commits:
    reviewer = Task(
        subagent_type="code-reviewer",
        prompt=f"Review Task {task_num}..."
    )
    reviewers.append(reviewer)

review_results = [r.wait() for r in reviewers]
```

**Code reviewer returns:** Strengths, Issues (Critical/Important/Minor), Assessment

### 5. Apply Review Feedback

**If issues found:**
- Fix Critical issues immediately
- Fix Important issues before next batch
- Note Minor issues

**Dispatch follow-up subagent(s):**

```python
# For parallel execution with issues: Can fix in parallel if independent
fix_agents = []
for task_num, issues in task_issues.items():
    if issues:  # Has Critical or Important issues
        agent = Task(
            subagent_type="general-purpose",
            description=f"Fix Task {task_num} issues",
            prompt=f"Fix issues from code review: {issues}"
        )
        fix_agents.append(agent)

# Wait for all fixes
fix_results = [a.wait() for a in fix_agents]
```

### 6. Mark Complete, Next Batch

- Mark completed tasks in TodoWrite
- Identify next batch of independent tasks (or next sequential task)
- Repeat steps 2-6

### 7. Final Review

After all tasks complete, dispatch final code-reviewer:
- Reviews entire implementation
- Checks all plan requirements met
- Validates overall architecture

**Optional: Async Codex architectural review**

For complex implementations, get independent Codex review in parallel with standard review:

```python
# Standard code review
standard_review = Task(subagent_type="code-reviewer", ...)

# Parallel Codex architectural review
codex_review = delegate(
    prompt="""
## Context
Read docs/plans/plan.md
Review all implementation commits

## Task
Perform architectural review of completed implementation

## Review Focus
- Does implementation match planned architecture?
- Are there architectural issues or anti-patterns?
- Integration quality across components
- Performance or scalability concerns
- Security considerations
- Edge cases handled correctly?

## Expected Output
1. Architectural strengths
2. Architectural concerns
3. Recommendations for improvements
""",
    cwd=project_path,
    sandbox="read-only",
    background=True,
    timeout=300
)

# Collect both reviews
reviews = [standard_review.wait(), codex_review.wait()]
# Synthesize findings
```

### 8. Complete Development

After final review passes:
- Announce: "I'm using the Finishing a Development Branch skill to complete this work."
- Switch to skills/collaboration/finishing-a-development-branch
- Follow that skill to verify tests, present options, execute choice

## Example Workflow

**Sequential Example:**
```
You: I'm using Subagent-Driven Development to execute this plan.

[Load plan, create TodoWrite]

Task 1: Hook installation script

[Dispatch implementation subagent]
Subagent: Implemented install-hook with tests, 5/5 passing

[Get git SHAs, dispatch code-reviewer]
Reviewer: Strengths: Good test coverage. Issues: None. Ready.

[Mark Task 1 complete]

Task 2: Recovery modes

[Dispatch implementation subagent]
Subagent: Added verify/repair, 8/8 tests passing

[Dispatch code-reviewer]
Reviewer: Strengths: Solid. Issues (Important): Missing progress reporting

[Dispatch fix subagent]
Fix subagent: Added progress every 100 conversations

[Verify fix, mark Task 2 complete]

...

[After all tasks]
[Dispatch final code-reviewer]
Final reviewer: All requirements met, ready to merge

Done!
```

**Parallel Example:**
```
You: I'm using Subagent-Driven Development to execute this plan.

[Load plan, create TodoWrite]
[Analyze dependencies: Tasks 1, 2, 3 are independent]

Batch 1: Tasks 1, 2, 3 (Parallel)

[Dispatch 3 implementation subagents in parallel]
- Agent 1: Database schema
- Agent 2: API endpoints
- Agent 3: Frontend components (Codex agent for deep analysis)

[All 3 agents work concurrently - 45% faster than sequential]

Results:
- Agent 1: Schema created, 3/3 tests passing
- Agent 2: Endpoints implemented, 8/8 tests passing
- Agent 3 (Codex): Components + accessibility audit, 12/12 tests passing

[Dispatch single review of all 3 tasks together]
Reviewer: Strengths: Good integration. Issues (Important): Task 2 API missing error handling

[Dispatch fix subagent for Task 2]
Fix subagent: Added error handling, all tests passing

[Mark Tasks 1, 2, 3 complete]

Task 4: Integration tests (depends on 1-3, sequential)

[Dispatch implementation subagent]
Subagent: Integration tests, 15/15 passing

[Dispatch code-reviewer]
Reviewer: Perfect. Ready.

[After all tasks]
[Dispatch final code-reviewer + parallel Codex architectural review]
Standard reviewer: All requirements met
Codex reviewer: Architecture solid, minor performance recommendation

Done! (2x faster due to parallel execution)
```

## Advantages

**vs. Manual execution:**
- Subagents follow TDD naturally
- Fresh context per task (no confusion)
- Parallel execution possible (2-3x faster for independent tasks)
- Codex integration for complex tasks

**vs. Executing Plans:**
- Same session (no handoff)
- Continuous progress (no waiting)
- Review checkpoints automatic
- Parallel execution built-in

**Parallel execution benefits:**
- 2-3x faster for plans with independent tasks
- Tested: 45% speedup with 3 parallel agents
- Mix Task tool + Codex agents for optimal results
- No wait time when Codex runs in background

**Cost:**
- More subagent invocations
- But catches issues early (cheaper than debugging later)
- Parallel execution has minimal overhead

## Red Flags

**Never:**
- Skip code review between task batches
- Proceed with unfixed Critical issues
- Parallelize dependent tasks (causes conflicts)
- Implement without reading plan task
- Parallelize tasks touching same files

**Safe parallel execution:**
- Analyze dependencies first (step 2)
- Only parallelize truly independent tasks
- Review for cross-task integration issues
- Fix in parallel when issues are independent

**If subagent fails task:**
- Dispatch fix subagent with specific instructions
- Don't try to fix manually (context pollution)
- Can parallelize fixes if issues are independent

## Integration

**Pairs with:**
- skills/collaboration/writing-plans (creates the plan)
- skills/collaboration/requesting-code-review (review template)
- skills/testing/test-driven-development (subagents follow this)
- skills/collaboration/delegating-to-codex (Codex as specialized agent)
- skills/collaboration/orchestrating-multi-agent-work (parallel patterns)

**Alternative to:**
- skills/collaboration/executing-plans (parallel session)

**Codex integration patterns:**
- Async Codex agents in parallel with Task tool agents
- Codex for complex tasks requiring deep analysis
- Codex architectural review in parallel with standard review
- Background execution (background=True) for zero wait time

See code-reviewer template: skills/collaboration/requesting-code-review/code-reviewer.md
