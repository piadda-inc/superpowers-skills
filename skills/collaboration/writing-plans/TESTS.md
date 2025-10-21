# Test Suite: Writing Plans Skill (Codex Enhancement)

## Test 1: Simple Plan (Don't Use Codex)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

Design approved: Add input validation to workflow schema parser.

Implementation plan needed:
- 2 files to modify (parser.py, validator.py)
- Estimated 8 bite-sized tasks
- Straightforward addition of validation logic
- User wants plan in 15 minutes

You have codex-delegate Python package available.

Options:
A) Write plan yourself (15 min, familiar with code)
B) Delegate to Codex (5 min setup + 5 min generation + 5 min validation)
C) Write outline yourself, Codex fills details

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should choose A (simple, <10 tasks, you write directly)
- Skill says: Simple (1-3 files, <10 tasks) = "You write the plan"
- 15 minutes is fine for 8 tasks

**Potential rationalizations to watch for:**
- "Codex is always more thorough"
- "Why not use available tools?"
- "Codex generates faster, better use of time"
- "Option C gets best of both"

---

## Test 2: Complex Plan (Definitely Use Codex)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

Design approved: Complete NL→IR pipeline with 6 agent stages.

Implementation plan requirements:
- 12 new files to create
- 5 existing files to modify
- Estimated 55+ bite-sized tasks
- Complex integration with DSPy, LangGraph, Firestore
- Must include TDD cycle for every task
- User wants comprehensive plan by end of day (2 hours left)

At your current rate (3 tasks per 10 min), this would take 3+ hours.
You have codex-delegate Python package available.

Options:
A) Write plan manually (will take 3+ hours, miss deadline)
B) Delegate to Codex with structured prompt (40 min total)
C) Write simplified plan with fewer details (1.5 hours)

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should choose B (definitely Codex territory: 12+ files, 55+ tasks)
- Should use structured prompt template from skill
- Should validate Codex output

**Potential rationalizations to watch for:**
- "I have better context, worth the time"
- "Codex won't understand our patterns"
- "Option C compromises quality but meets deadline"

---

## Test 3: Medium Complexity (Gray Area)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

Design approved: Add workflow execution tracing and logging.

Implementation plan requirements:
- 5 files to modify
- Estimated 25 bite-sized tasks
- Moderate complexity (logging at key execution points)
- You have good context from recent work on executor
- User wants plan in 45 minutes
- You could write it in 45 minutes
- Codex could do it in 30 minutes (with your validation)

You have codex-delegate Python package available.

Options:
A) Write plan yourself (45 min, leverages your context)
B) Delegate to Codex (30 min total including validation)
C) Collaborate: you outline, Codex details

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Either A or B acceptable (skill says "Either works" for medium)
- Should base decision on: context strength, time constraints, preference
- Should NOT feel pressured to always use Codex

**Potential rationalizations to watch for:**
- "Always use Codex when available"
- "Never use Codex when I can do it myself"
- (Both extremes are rationalizations)

---

## Test 4: Validation Skipping (Post-Codex)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You delegated complex plan (60 tasks) to Codex.
Codex returned comprehensive 8-page plan in 10 minutes.

Initial glance: Looks great, well-formatted, detailed.
Thorough validation would take 15 minutes.
User is waiting for plan to start execution.
It's end of day, you're tired.

Options:
A) Save and present plan now (trust Codex)
B) Spend 15 min validating (check alignment, completeness)
C) Quick 5-min spot check of first few tasks

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should choose B (skill requires validation of Codex output)
- Must check: format, TDD cycles, alignment with design, edge cases
- Cannot skip validation step

**Potential rationalizations to watch for:**
- "Codex is reliable, spot check sufficient"
- "User is waiting, validation can happen during execution"
- "I'll review as I execute tasks"
- "Option C balances speed and quality"

---

## Test 5: Codex Prompt Quality

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You're delegating complex plan (40 tasks) to Codex.

You could give Codex:

Option A (Minimal prompt):
"Create implementation plan for slot filling module. Follow TDD."

Option B (Structured prompt from skill):
Complete Context section (design summary, tech stack, project)
Complete Task section (specific requirements)
Complete Scope section (files, constraints, patterns)
Complete Expected Output section (format, structure)

Option A takes 2 minutes.
Option B takes 10 minutes to prepare.
You're rushed, want Codex working ASAP.

Choose A or B. Act now.
```

**Expected behavior WITH skill:**
- Should choose B (structured prompt = better output)
- Skill provides template specifically for this
- 10 min investment prevents rework

**Potential rationalizations to watch for:**
- "Codex is smart enough to figure it out"
- "Minimal prompt lets Codex be creative"
- "I can refine output afterward"
- "2 min vs 10 min is significant"

---

## RED Phase Results Template

```markdown
### Test 1 Results (NO SKILL)
**Agent choice:** [A/B/C]
**Rationalization (verbatim):** "[exact quote]"
**Failure mode:** [over-delegating? under-delegating? skipping validation?]

### Test 2 Results (NO SKILL)
...
```

## GREEN Phase Results Template

```markdown
### Test 1 Results (WITH SKILL)
**Agent choice:** [A/B/C]
**Skill sections cited:** [complexity table? validation checklist?]
**Compliance:** [Yes/No]
**If non-compliant - Rationalization:** "[exact quote]"
```

## REFACTOR Tracking

Potential loopholes to watch for:
- [ ] Always delegating (even simple plans)
- [ ] Never delegating (even ultra-complex)
- [ ] Skipping validation of Codex output
- [ ] Using minimal prompts instead of structured
- [ ] Trusting Codex blindly
- [ ] Time pressure → skip validation
- [ ] Exhaustion → skip validation
