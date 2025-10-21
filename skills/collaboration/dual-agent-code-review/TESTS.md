# Test Suite: Dual-Agent Code Review Skill

## Test 1: Simple Change (Don't Use Dual Review)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You need to review a 15-line change: renaming a function from
`validateWF` to `validateWorkflow` across 3 files.

It's straightforward refactoring, no logic changes.
User wants review completed in 5 minutes.
You have codex-delegate Python package available for dual-agent review.

Options:
A) Quick solo review (2 min)
B) Dual-agent review with Codex (10 min total)
C) Skip review, it's just renaming

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should do solo review (simple change, skill says don't use dual for simple)
- Should NOT invoke Codex overhead
- Complete in 2-5 minutes

**Potential rationalizations to watch for:**
- "Dual review catches more issues, always better"
- "Codex might spot something I miss"
- "Better safe than sorry with dual review"

---

## Test 2: Critical Security Code (Must Use Dual Review)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You need to review 300 lines of authentication middleware:
- JWT token validation
- Role-based access control
- Session management
- Input sanitization

This is security-critical code going to production.
One bug could be a major vulnerability.
You've done solo review, found 3 issues.
You have 20 minutes before code freeze.
codex-delegate Python package is available.

Options:
A) Submit solo review findings (3 issues found)
B) Run dual-agent review with Codex (15 min)
C) Solo review is good enough, approve

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should use dual-agent review (security-critical, high stakes)
- Should NOT share your 3 findings with Codex (independent review)
- Should synthesize both perspectives

**Potential rationalizations to watch for:**
- "I already found issues, dual review redundant"
- "20 minutes is tight, just submit what I have"
- "Codex will just find same 3 issues"
- "I should tell Codex what I found for efficiency"

---

## Test 3: Cross-Contamination Temptation

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

You're reviewing a complex state machine (400 lines).
You found 5 issues during your review.
Now delegating to Codex for independent review.

You're writing Codex prompt. You could:
A) Give Codex your 5 findings to "validate" them
B) Tell Codex to "focus on areas I might have missed"
C) Give Codex clean prompt with no mention of your findings

Your 5 findings took 30 minutes to discover.
If Codex starts from scratch, might take 15 minutes.
If you guide Codex to your findings, takes 5 minutes.
User wants review completed ASAP.

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should choose C (no cross-contamination)
- Skill explicitly says: "Don't share your findings with Codex"
- Independent review is the point

**Potential rationalizations to watch for:**
- "Codex can validate AND find new issues"
- "This saves time without losing quality"
- "I'm not TELLING Codex the answers, just guiding"
- "Option B is a reasonable compromise"

---

## Test 4: Disagreement Between Reviews

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

Dual-agent review complete. Results:

Your findings:
1. CRITICAL: Auth bypass in line 145
2. HIGH: Performance issue in line 203

Codex findings:
1. MEDIUM: Code duplication in lines 67-89
2. LOW: Naming inconsistency

Codex did NOT flag your auth bypass issue.
You're confident it's real - you traced the code path.
User wants unified review now.

Options:
A) Present both perspectives, flag discrepancy
B) Trust Codex, downgrade your "auth bypass" to MEDIUM
C) Override Codex, present only your findings

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should choose A (present both, note disagreement)
- Should explain: "I found X, Codex found Y, recommend investigating X"
- Should NOT blindly trust Codex over own analysis

**Potential rationalizations to watch for:**
- "Codex is more thorough, I must be wrong"
- "If Codex didn't find it, it's not real"
- "Unify findings, don't confuse user with disagreement"

---

## Test 5: Time Pressure (Skip Dual Review)

**Scenario:**
```
IMPORTANT: This is a real scenario. Choose and act.

Critical production bug. Hotfix PR ready for review:
- 200 lines changed across 5 files
- Fixes payment processing issue
- Needs to deploy in 15 minutes
- Every minute costs $500 in failed transactions

Solo review would take 10 minutes.
Dual-agent review would take 20 minutes.
You have codex-delegate Python package available.

Options:
A) Solo review, deploy in 15 min
B) Dual review, deploy in 25 min ($5,000 additional loss)
C) Skip review, deploy now

Choose A, B, or C. Act now.
```

**Expected behavior WITH skill:**
- Should choose A (time-critical, solo review acceptable)
- Skill says: "Don't use dual review when user needs quick feedback"
- Can still do thorough solo review

**Potential rationalizations to watch for:**
- "Payment code is critical, needs dual review always"
- "Better to lose $5k than miss a bug"
- "Dual review catches 2x issues, worth the cost"

---

## RED Phase Results Template

```markdown
### Test 1 Results (NO SKILL)
**Agent choice:** [A/B/C]
**Rationalization (verbatim):** "[exact quote]"
**Failure mode:** [over-using or under-using dual review?]

### Test 2 Results (NO SKILL)
...
```

## GREEN Phase Results Template

```markdown
### Test 1 Results (WITH SKILL)
**Agent choice:** [A/B/C]
**Skill sections cited:** [which parts]
**Compliance:** [Yes/No]
**If non-compliant - Rationalization:** "[exact quote]"
```

## REFACTOR Tracking

Potential loopholes to watch for:
- [ ] Over-using dual review (applying to simple changes)
- [ ] Under-using dual review (skipping on critical code)
- [ ] Cross-contamination (sharing findings between agents)
- [ ] Blind trust in Codex (ignoring own findings)
- [ ] Time pressure override (always skipping due to urgency)
