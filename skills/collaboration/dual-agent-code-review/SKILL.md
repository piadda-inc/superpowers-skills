---
name: Dual-Agent Code Review
description: Leverage both Claude and Codex for comprehensive code reviews that catch more issues through complementary perspectives
when_to_use: When reviewing critical code (security, architecture, complex logic), large changes (500+ lines), or code with high blast radius
version: 2.1.0
---

# Dual-Agent Code Review

## Overview

**Two perspectives are better than one.** Claude and Codex have complementary strengths that catch different types of issues.

**Core principle:** Use both agents independently, then synthesize findings. Don't show one agent's findings to the other until after both complete their reviews.

## When to Use Dual-Agent Review

### Use When:

| Scenario | Why Dual-Agent | Impact |
|----------|---------------|--------|
| **Critical code** | Security, auth, payments | High risk of bugs = high review rigor |
| **Large changes** | 500+ lines, 5+ files | Easy to miss issues in large diffs |
| **Architecture changes** | New patterns, refactoring | Multiple perspectives reveal trade-offs |
| **Complex logic** | Algorithms, state machines | Different agents catch different edge cases |
| **Pre-merge review** | Before production deploy | Final quality gate |

### Don't Use When:

- **Simple changes** - Single-file typo fixes, formatting
- **Obvious changes** - Renaming, moving files
- **User needs quick feedback** - Time-sensitive reviews
- **Low-risk code** - Documentation, comments, tests

## The Dual-Agent Review Process

### Phase 1: Independent Reviews (No Cross-Contamination)

**CRITICAL: Don't share your findings with Codex, and vice versa.**

Why? Independent reviews catch different issues. If you bias Codex with your findings, you lose the benefit of an independent perspective.

#### 1a. Claude's Review (You)

**Your strengths:**
- High-level architecture and design patterns
- Business logic alignment with requirements
- User experience and API design
- Cross-cutting concerns (security, performance, maintainability)
- Context from conversation history

**Review focus:**
```markdown
# Your review checklist:
- [ ] Does this align with project architecture (CLAUDE.md)?
- [ ] Are there security concerns (auth, input validation)?
- [ ] Is the user-facing behavior correct?
- [ ] Are there better design patterns?
- [ ] Is error handling comprehensive?
- [ ] Are there performance implications?
- [ ] Is the code maintainable?
```

#### 1b. Codex's Review (Independent)

**Codex's strengths:**
- Deep technical analysis across multiple files
- Pattern consistency detection
- Edge case identification
- Code quality metrics (complexity, duplication)
- Language-specific idioms and best practices

**Codex review prompt template:**
```python
codex_review_prompt = """
## Context
[Brief description of what this code does]

## Task
Perform a comprehensive code review of the following changes.

## Files Changed
[List files or git diff]

## Review Criteria
1. **Correctness**: Logic errors, edge cases, off-by-one errors
2. **Quality**: Code duplication, complexity, naming
3. **Patterns**: Consistency with existing codebase patterns
4. **Edge Cases**: Input validation, null handling, boundary conditions
5. **Best Practices**: Language idioms, anti-patterns, common pitfalls

## Expected Output
For each issue found:
1. Severity: Critical / High / Medium / Low
2. Location: File:Line
3. Issue: What's wrong
4. Impact: What could happen
5. Recommendation: How to fix

Format as a structured list.
"""

from codex_delegate import delegate, launch_background, get_task_result

# Option 1: Synchronous (blocks until complete)
codex_result = delegate(
    prompt=codex_review_prompt,
    cwd="/path/to/project",
    sandbox="read-only",
    timeout=300
)

# Option 2: In-session async (parallel with your review)
codex_task = delegate(
    prompt=codex_review_prompt,
    cwd="/path/to/project",
    sandbox="read-only",
    background=True,
    timeout=300
)
# Do your review while Codex runs...
codex_result = codex_task.wait()

# Option 3: Cross-session async (for very large codebases)
# Launch review, continue other work, check results later
task_id = launch_background(
    name="security_review",
    prompt=codex_review_prompt,
    cwd="/path/to/project",
    sandbox="read-only",
    timeout=600  # Longer timeout for deep analysis
)
# Later (same or different Claude session):
# codex_result = get_task_result(task_id)
```

### Phase 2: Synthesis (Compare and Integrate)

**After both reviews are complete:**

1. **Identify Agreement**
   - Issues both agents found = definitely need fixing
   - High confidence these are real problems

2. **Identify Unique Findings**
   - Issues only Claude found = likely architectural/contextual
   - Issues only Codex found = likely technical/edge-case
   - Both are valuable!

3. **Assess Severity**
   - Critical: Security, data loss, crashes
   - High: Incorrect behavior, performance issues
   - Medium: Code quality, maintainability
   - Low: Style, minor improvements

4. **Prioritize**
   - Fix critical/high before merging
   - Plan medium/low for future work
   - Document decisions

### Phase 3: Present Unified Review

**To the user:**

```markdown
# Code Review: [Feature/PR Name]

## Summary
Reviewed by Claude + Codex. Found X critical, Y high, Z medium issues.

## Critical Issues (Fix Before Merge)

### 1. [Issue Name]
**Found by:** Both agents
**Location:** file.py:123
**Issue:** [Description]
**Impact:** [What could happen]
**Recommendation:** [How to fix]

### 2. [Issue Name]
**Found by:** Codex
**Location:** file.py:456
**Issue:** [Description]
...

## High Priority Issues

[Similar format]

## Medium/Low Issues

[Similar format]

## Positive Findings

- [What was done well]
- [Good patterns to reinforce]

## Recommendation

- [ ] Fix critical issues before merging
- [ ] Consider high-priority improvements
- [ ] Document medium issues for follow-up
```

## Collaboration Patterns

### Pattern 1: Sequential Review

**When:** Standard code review, no time pressure

**Flow:**
```
1. Claude reviews first (you have conversation context)
2. Codex reviews independently (technical deep dive)
3. Synthesize findings
4. Present to user
```

### Pattern 2: Parallel Review (Faster - RECOMMENDED)

**When:** Large changes, want fastest turnaround

**Flow:**
```
1. Start Codex review in background (background=True)
2. Claude reviews simultaneously
3. Wait for Codex to complete
4. Synthesize findings
5. Present to user
```

**How to do TRUE parallel execution (in-session):**
```python
from codex_delegate import delegate

# Start Codex review in background (non-blocking)
codex_task = delegate(
    prompt=codex_review_prompt,  # From template above
    cwd="/path/to/project",
    sandbox="read-only",
    background=True,  # KEY: Runs in parallel
    on_stream=lambda line: print(f"[Codex] {line}")
)

# While Codex works in parallel, do your own review
your_findings = perform_claude_review()  # Your review logic

# When ready, wait for Codex to complete
codex_result = codex_task.wait()
codex_findings = parse_codex_findings(codex_result.output)

# Synthesize both reviews
unified_review = synthesize_findings(your_findings, codex_findings)
```

**Time savings:** With parallel execution, total time = max(claude_time, codex_time) instead of sum. Typically 40-50% faster.

### Pattern 3: Launch and Continue (Cross-Session)

**When:** Very large codebase (10,000+ lines), deep security audit, or when you need to continue other work immediately

**Flow:**
```
SESSION 1:
1. Launch Codex review(s) as persistent background task(s)
2. Continue with other work immediately (no blocking)
3. Optionally start your own review

SESSION 2 (later, same or different Claude instance):
4. Check if Codex review(s) completed
5. Retrieve results
6. Synthesize with your findings (if done) or do your review now
7. Present unified review to user
```

**Implementation:**
```python
from codex_delegate import launch_background, get_task_status, get_task_result, list_background_tasks

# SESSION 1: Launch multiple specialized reviews
# ================================================
user_says = "Please review the entire codebase before our security audit tomorrow"

# Launch 3 specialized Codex reviews that persist across sessions
review_ids = {
    'security': launch_background(
        "security_review",
        "Deep security audit: SQL injection, XSS, auth bypasses, data exposure",
        cwd="/path/to/project",
        timeout=1200  # 20 minutes for thorough analysis
    ),
    'architecture': launch_background(
        "architecture_review",
        "Architecture review: coupling, cohesion, SOLID violations, design patterns",
        cwd="/path/to/project",
        timeout=900
    ),
    'quality': launch_background(
        "quality_review",
        "Code quality: duplication, complexity, maintainability, tech debt",
        cwd="/path/to/project",
        timeout=900
    )
}

tell_user(f"Launched 3 comprehensive code reviews (IDs: {review_ids}). " +
          "These will run in the background. I'll check back in a few hours.")

# Continue with other work or end session...

# SESSION 2: Check and synthesize results
# ========================================
# List all reviews
reviews = list_background_tasks(status='completed')

# Get results
security_result = get_task_result(review_ids['security'])
architecture_result = get_task_result(review_ids['architecture'])
quality_result = get_task_result(review_ids['quality'])

# Synthesize all findings
unified_review = synthesize_all_reviews([
    security_result.output,
    architecture_result.output,
    quality_result.output
])

# Present to user
present_comprehensive_review(unified_review)
```

**Benefits:**
- No blocking on long-running reviews
- Can launch overnight or during off-hours
- Multiple Claude instances can collaborate
- Perfect for pre-launch security audits

### Pattern 4: Second Opinion

**When:** You've already reviewed, want validation

**Flow:**
```
1. You complete review, find N issues
2. Ask Codex for independent review (don't mention your findings)
3. Compare: Did Codex find same issues? Different ones?
4. If major discrepancies → discuss with user which perspective is correct
```

## Quality Metrics

**Track to improve over time:**

| Metric | Target | Meaning |
|--------|--------|---------|
| **Overlap Rate** | 40-60% | % of issues both agents found |
| **Unique Findings** | 30-50% | Issues only one agent found |
| **False Positive Rate** | <10% | "Issues" that aren't real problems |
| **Critical Miss Rate** | 0% | Critical issues neither agent caught |

**If overlap is too low (<20%):** Agents are looking at different things. Align review criteria.

**If overlap is too high (>80%):** Not getting benefit of dual perspective. Adjust Codex prompt.

## Severity Guidelines

### Critical (Must Fix Before Merge)
- Security vulnerabilities (SQL injection, XSS, auth bypass)
- Data loss or corruption
- Crashes or unhandled exceptions in critical paths
- Breaking changes without migration path

### High (Strongly Recommend Fixing)
- Incorrect business logic
- Performance issues (N+1 queries, memory leaks)
- Poor error handling in important flows
- API contract violations

### Medium (Plan for Follow-Up)
- Code duplication
- High complexity (cyclomatic > 10)
- Missing tests for edge cases
- Maintainability issues

### Low (Nice to Have)
- Style inconsistencies
- Minor refactoring opportunities
- Documentation improvements
- Better variable names

## Common Mistakes (Rationalizations)

| Excuse | Reality |
|--------|---------|
| "One review is enough" | Two independent reviews catch 2x issues |
| "Codex will just repeat my findings" | Independent review = different perspective |
| "Too slow to do two reviews" | Parallel reviews take same time as sequential |
| "I'll mention my findings to Codex for efficiency" | This defeats the purpose. No cross-contamination. |
| "Low-risk code doesn't need dual review" | Even simple code benefits, just use lighter criteria |
| "Codex might disagree with me" | Disagreement is valuable! Reveals assumptions. |

## Integration with Other Skills

### With Requesting Code Review

From `skills/collaboration/requesting-code-review`:
- That skill: How to request review from user
- This skill: How to perform thorough review using dual agents

### With Systematic Debugging

If code review finds bugs:
- Use `skills/debugging/systematic-debugging` to investigate root cause
- Use `skills/collaboration/delegating-to-codex` to delegate complex investigations

### With Test-Driven Development

Code review should verify:
- Tests exist for new functionality
- Tests cover edge cases
- Tests follow TDD principles from `skills/testing/test-driven-development`

## Red Flags - Use Dual-Agent Review

If code has any of these characteristics:
- Handles authentication or authorization
- Processes payments or sensitive data
- Modifies core business logic
- Changes shared libraries or APIs
- Has complex state management
- Touches 5+ files
- Adds 500+ lines of code
- Previous similar changes caused bugs

**These signal:** High-value candidate for dual-agent review

## Quick Reference

**Dual-Agent Review Checklist:**
- [ ] Determined this code warrants dual review?
- [ ] Prepared Codex review prompt with clear criteria?
- [ ] Completed your own independent review first?
- [ ] Invoked Codex review WITHOUT sharing your findings?
- [ ] Waited for Codex to complete before comparing?
- [ ] Synthesized findings (agreement + unique issues)?
- [ ] Categorized by severity (Critical/High/Medium/Low)?
- [ ] Presented unified review to user with recommendations?

**Time Investment:**
- Simple dual review: +5 minutes (parallel)
- Complex dual review: +15 minutes (synthesis time)
- Bugs prevented: Typically 3-5 additional issues caught

## Real-World Impact

**Measured benefits from dual-agent reviews:**
- 2x more issues found vs. single-agent review
- 60% of critical bugs caught by both agents
- 40% of issues found by only one agent (unique perspectives!)
- False positive rate <5% (both agents rarely wrong about same issue)

**Key insight:** "Claude catches architectural and contextual issues. Codex catches technical and edge-case issues. Together they provide defense in depth for code quality."

## Example: Full Dual-Agent Review

**Scenario:** Review new workflow validation logic (300 lines, security-critical)

### Phase 1: Independent Reviews

**Claude's Review (You):**
```markdown
Found 4 issues:
1. CRITICAL: Missing input validation on user-provided workflow JSON
2. HIGH: Error messages leak internal structure
3. MEDIUM: Validation logic duplicated in 2 places
4. LOW: Function name not descriptive (validateWF → validateWorkflow)
```

**Codex's Review (Independent):**
```markdown
Found 5 issues:
1. CRITICAL: Missing input validation on workflow JSON (same as Claude)
2. HIGH: Off-by-one error in node index validation (line 145)
3. MEDIUM: Inefficient O(n²) validation loop (line 178)
4. MEDIUM: Missing null check on optional field (line 203)
5. LOW: Inconsistent variable naming (snake_case vs camelCase)
```

### Phase 2: Synthesis

**Agreement:**
- Both found: Input validation missing → **DEFINITELY fix**

**Unique to Claude:**
- Error message leakage → **Contextual/security concern**
- Duplication → **Architectural observation**

**Unique to Codex:**
- Off-by-one error → **Technical edge case**
- Performance issue → **Code analysis**
- Null check → **Safety issue**

### Phase 3: Unified Review

**To User:**
```markdown
# Code Review: Workflow Validation Logic

## Summary
Reviewed 300 lines by Claude + Codex. Found 1 critical (both), 3 high, 3 medium issues.

## Critical Issues (Fix Before Merge)

### 1. Missing Input Validation
**Found by:** Both Claude and Codex
**Location:** workflow_validator.py:78
**Issue:** User-provided workflow JSON not validated before processing
**Impact:** Could cause crashes or security issues with malicious input
**Recommendation:** Add JSON schema validation before processing

## High Priority Issues

### 2. Off-by-One Error in Node Validation
**Found by:** Codex
**Location:** workflow_validator.py:145
**Issue:** `for i in range(len(nodes))` should be `range(len(nodes) - 1)` when checking next node
**Impact:** Array out of bounds exception on last node
**Recommendation:** Fix loop bounds or use enumerate()

### 3. Error Messages Leak Internal Structure
**Found by:** Claude
**Location:** workflow_validator.py:112-115
**Issue:** Stack traces and internal paths exposed to users
**Impact:** Information disclosure vulnerability
**Recommendation:** Log full errors internally, show generic messages to users

### 4. Performance Issue in Validation
**Found by:** Codex
**Location:** workflow_validator.py:178
**Issue:** Nested loop creates O(n²) complexity for node validation
**Impact:** Slow validation for workflows with 100+ nodes
**Recommendation:** Use set for O(1) lookups instead of list scanning

## Recommendation

✅ Fix critical issue #1 (input validation) - **Required**
✅ Fix high issues #2-4 - **Strongly recommended**
📋 Document medium issues for follow-up work
```

---

This pattern ensures comprehensive review coverage while leveraging each agent's unique strengths.
