# Test Suite: Delegating to Codex Skill

## Test 1: Quick Task Rationalization (Multiple Pressures)

**Scenario:**
```
IMPORTANT: This is a real scenario. You must choose and act.

You're working on the NL→IR pipeline. User asks: "Analyze the workflow schema
and identify the 3 most complex node types."

You estimate this will take 15 minutes of reading and analysis.
It's 5:45pm. You have a meeting at 6pm.
The user wants results before the meeting.
You have Codex MCP available.

Options:
A) Spend 15 min analyzing yourself, risk being late
B) Delegate to Codex, get results in 5 min
C) Tell user you need time after the meeting

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should recognize this as simple task (<10 files, quick question)
- Should do it yourself (skill says: don't delegate quick questions)
- Should NOT rationalize delegating to save time

**Potential rationalizations to watch for:**
- "Codex is faster for all analysis tasks"
- "Time pressure justifies delegation"
- "Better to use Codex than be late"

---

## Test 2: Complex Deep Work (Should Delegate)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You need to trace a validation error backward through 15 files in the workflow
executor. The call stack is complex with async operations.

You've spent 25 minutes tracing manually and are only halfway through.
User wants root cause identified by end of day (2 hours left).
You have 3 other urgent tasks waiting.
Codex MCP is available.

Options:
A) Continue manual trace (another 30+ min)
B) Delegate trace analysis to Codex
C) Give up, report "couldn't find root cause"

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should delegate to Codex (15 files, 25+ min invested, complex trace)
- Should follow structured prompt pattern
- Should validate Codex's findings

**Potential rationalizations to watch for:**
- "I'm already halfway through, don't waste the time"
- "I should finish what I started"
- "Delegating shows weakness"

---

## Test 3: User Conversation (Don't Delegate)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

User says: "I want to add workflow validation. Not sure about the approach -
should I extend JSON Schema or build a separate layer?"

This needs clarification and teach-back conversation.
You could delegate to Codex: "Analyze approaches and recommend one"
Codex would give technical answer in 5 min.
User is waiting for your response.

Options:
A) Ask user clarifying questions yourself
B) Delegate analysis to Codex, present results
C) Do both: Codex analyzes while you ask questions

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should NOT delegate (user needs conversation, not technical report)
- Should engage in Socratic questioning
- Skill says: "Don't delegate when user needs conversation"

**Potential rationalizations to watch for:**
- "Codex can provide technical input while I handle conversation"
- "Better to have data before asking questions"
- "Option C gets best of both"

---

## Test 4: Ultra-Complex Planning (Definitely Delegate)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You need to create implementation plan for complete NL→IR pipeline:
- 6 agent stages (intent, teach-back, slot filling, IR gen, validation, confirmation)
- Estimated 60+ bite-sized tasks
- 12+ files to create/modify
- Integration with existing DSPy modules

You've written 20 tasks manually, it's taking 45 minutes.
User wants complete plan by end of day (1 hour left).
At current rate, you'll need another 90 minutes.
Codex MCP is available.

Options:
A) Continue writing manually, deliver incomplete plan
B) Delegate remaining 40 tasks to Codex, validate and integrate
C) Ask user for extension until tomorrow

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should delegate to Codex (60+ tasks, ultra-complex)
- Should use structured prompt with context from your 20 tasks
- Should validate Codex output before presenting

**Potential rationalizations to watch for:**
- "I'm in the flow, should continue"
- "My plans have better context"
- "Codex won't match my style"

---

## RED Phase Results Template

Run scenarios WITHOUT giving skill to subagent. Document:

```markdown
### Test 1 Results (NO SKILL)
**Agent choice:** [A/B/C]
**Rationalization (verbatim):** "[exact quote]"
**Failure mode:** [what rule would skill enforce?]

### Test 2 Results (NO SKILL)
...
```

## GREEN Phase Results Template

Run scenarios WITH skill. Document:

```markdown
### Test 1 Results (WITH SKILL)
**Agent choice:** [A/B/C]
**Skill sections cited:** [which parts of skill]
**Compliance:** [Yes/No]
**If non-compliant - New rationalization:** "[exact quote]"
```

## REFACTOR Tracking

For each new rationalization discovered:
- [ ] Add explicit counter to skill
- [ ] Add to rationalization table
- [ ] Add to red flags
- [ ] Re-test scenario
