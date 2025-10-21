# Skills Update: Critical Monitoring Guidance for Background Codex Tasks

**Date:** 2025-10-21
**Issue:** Users starting watch command too late were missing task completion notifications
**Solution:** Emphasize immediate monitoring in all relevant skills

## The Problem

During end-to-end testing, we discovered:
- Codex tasks complete **fast** (10-30 seconds for simple analysis, 5-30 minutes for complex work)
- Users who delay starting `watch` command miss task completions
- Example: Task launched at 22:59:45, completed by 23:00:00 (15 seconds!)
- When `watch` started later, it reported "No running tasks to watch"

## The Solution

Updated all Codex-related skills to **emphasize immediate monitoring**:

### Changed From (Old Pattern)
```python
# Launch tasks
task_id = launch_background(...)

# Session 2 (later): Check status
# Option 1: List running tasks
# Option 2: Watch and get notified (RECOMMENDED)
#   python3 -m codex_delegate.cli watch --verbose
```

### Changed To (New Pattern)
```python
# Launch tasks
task_id = launch_background(...)

# CRITICAL: Start watch immediately (tasks complete fast!)
# Open a terminal NOW and run:
#   python3 -m codex_delegate.cli watch --verbose
# This monitors all running tasks and notifies when each completes
```

## Skills Updated

### 1. delegating-to-codex (v3.1.0 → v3.2.0)

**Changes:**
- Added **"CRITICAL: Start watch immediately"** warning
- Emphasized tasks complete in 5-30 minutes (not hours)
- Moved watch recommendation from "Session 2 (later)" to immediately after launch
- Changed tone from "Optional monitoring" to "Start NOW"

**Key Addition:**
```python
# CRITICAL: Start watch immediately (tasks complete fast!)
# Open a terminal NOW and run:
#   python3 -m codex_delegate.cli watch --verbose
```

### 2. dual-agent-code-review (v2.2.0 → v2.3.0)

**Changes:**
- Updated `tell_user()` message to include watch command directly
- Added CRITICAL note about starting watch immediately
- Emphasized 5-10 minute completion time for code reviews

**Key Addition:**
```python
tell_user(f"Launched 3 comprehensive code reviews (IDs: {review_ids}). " +
          "Open a terminal and run: python3 -m codex_delegate.cli watch --verbose")

# CRITICAL: Start watch immediately in a terminal
# Tasks can complete in 5-10 minutes, so start monitoring NOW:
#   python3 -m codex_delegate.cli watch --verbose
```

### 3. orchestrating-multi-agent-work (v2.1.0 → v2.2.0)

**Changes:**
- Added new cross-session async example with `launch_background()`
- Added CRITICAL monitoring guidance
- Updated comparison table to include cross-session row
- Distinguished between in-session async (background=True) and cross-session (launch_background)

**Key Additions:**
- New comparison row: `| **Cross-session** | N/A (Task tool only) | Use launch_background() + watch |`
- New example showing launch_background + immediate watch
- CRITICAL note: "Tasks complete fast (5-30 min), so monitor NOW"

## Updated Comparison Table

| Criteria | Use Task Tool | Use Codex Plugin |
|----------|---------------|------------------|
| **Parallel** | Built-in via Task tool | Use background=True or launch_background() |
| **Cross-session** | N/A (Task tool only) | Use launch_background() + watch |

## The Critical Pattern (All Skills)

**Every background task launch must now include:**

```python
# 1. Launch background task(s)
task_id = launch_background(name, prompt, cwd, ...)

# 2. CRITICAL: Start watch IMMEDIATELY
# Open a terminal NOW and run:
#   python3 -m codex_delegate.cli watch --verbose

# 3. Continue with other work or end session...
```

## Testing Evidence

From END_TO_END_TEST_RESULTS.md:

**First Test (Timing Issue):**
- Task launched: 22:59:45
- Task completed: ~23:00:00 (15 seconds)
- Watch started: 23:02:00 (2 minutes later)
- Result: "No running tasks to watch" ❌

**Second Test (Correct Pattern):**
- Task launched: immediately followed by watch
- Watch started: within 2 seconds of launch
- Result: Watch detected task, monitored it, and notified on completion ✅

## Key Learnings

1. **Speed Matters:** Codex completes simple analysis in 10-30 seconds, not minutes
2. **Immediate Action Required:** "Later" means "too late" for fast-completing tasks
3. **User Expectation Mismatch:** Users expect tasks to take longer than they actually do
4. **Clear Communication:** Need CRITICAL/NOW/IMMEDIATELY language to convey urgency

## Impact

**Before:** Users would launch tasks and check back "later", missing completions
**After:** Users know to start watch IMMEDIATELY and get real-time notifications

**Skill Coverage:**
- ✅ delegating-to-codex (primary Codex skill)
- ✅ dual-agent-code-review (uses cross-session async)
- ✅ orchestrating-multi-agent-work (uses parallel Codex agents)
- ✅ subagent-driven-development (uses in-session async, no change needed)

## Recommended Workflow

**For Claude instances helping users:**

When delegating to background Codex:
1. Use `launch_background()` to start task
2. **IMMEDIATELY tell user:** "Open a terminal and run: `python3 -m codex_delegate.cli watch --verbose`"
3. Emphasize: "Tasks complete fast (5-30 min), start monitoring now"
4. Continue with other work
5. Check results when watch notifies completion

## Version Summary

- delegating-to-codex: v3.1.0 → v3.2.0
- dual-agent-code-review: v2.2.0 → v2.3.0
- orchestrating-multi-agent-work: v2.1.0 → v2.2.0

All skills now properly emphasize immediate monitoring for background Codex tasks.
