---
name: Brainstorming Ideas Into Designs
description: Interactive idea refinement using Socratic method to develop fully-formed designs
when_to_use: when partner describes any feature or project idea, before writing code or implementation plans
version: 2.2.0
---

# Brainstorming Ideas Into Designs

## Overview

Transform rough ideas into fully-formed designs through structured questioning and alternative exploration.

**Core principle:** Ask questions to understand, explore alternatives, present design incrementally for validation.

**Announce at start:** "I'm using the Brainstorming skill to refine your idea into a design."

## The Process

### Phase 1: Understanding
- Check current project state in working directory
- Ask ONE question at a time to refine the idea
- Prefer multiple choice when possible
- Gather: Purpose, constraints, success criteria

### Phase 2: Exploration

**Step 1: Identify candidate approaches**
- Based on Phase 1 understanding, identify 2-3 different approaches
- For each: Core architecture, trade-offs, complexity assessment

**Step 2: Optional Async Codex Technical Opinion**

For complex or high-risk features, launch Codex analysis in parallel while you prepare your exploration:

**When to get async Codex opinion:**
- Uncertain about technical feasibility of approaches
- Need deeper analysis of existing codebase patterns
- Want to validate that proposed architectures fit existing system
- Multiple approaches with unclear trade-offs
- Complex feature spanning 5+ files

**Async Pattern (Non-Blocking):**
```python
from codex_delegate import delegate

# Launch Codex analysis in background (non-blocking)
codex_task = delegate(
    prompt="""
## Context
User wants to add workflow validation for NL→IR pipeline. Current system uses JSON Schema validation.

## Task
Analyze existing validation patterns and assess feasibility of 3 approaches:

1. Extend JSON Schema with custom validators
2. Build separate semantic validation layer
3. Hybrid: Schema + post-validation rules

## Expected Output
For each approach:
- Alignment with existing codebase patterns (high/medium/low)
- Implementation complexity (simple/moderate/complex)
- Technical risks or blockers
- Recommendation with reasoning
""",
    cwd=project_path,
    sandbox="read-only",
    background=True,  # KEY: Non-blocking execution
    timeout=180
)

# Continue with your own analysis while Codex works in parallel
# You can prepare your initial thoughts, explore codebase, etc.
```

**Step 3: Prepare your own analysis**

While Codex analyzes in background:
- Explore codebase patterns yourself
- Draft initial approach descriptions
- Identify key trade-offs

**Step 4: Synthesize findings**

```python
# When ready, collect Codex results
codex_result = codex_task.wait()

# Synthesize: Your analysis + Codex technical assessment
# Present unified exploration to user with informed trade-offs
```

**Benefits of async pattern:**
- **Zero wait time** - Codex runs while you work
- **Dual perspective** - Your insights + Codex technical depth
- **Better trade-off analysis** - Technical feasibility validated
- **Time saved:** 10-15 min vs. exploring dead-end approaches

**Don't use Codex for:**
- Simple features with obvious approaches
- When user already specified approach
- Pure business logic decisions (needs user context)
- Quick features (<5 files, straightforward implementation)

**Step 5: Present approaches to user**

Present 2-3 approaches with:
- Your architectural perspective
- Codex's technical feasibility assessment (if used)
- Clear trade-offs for each
- Ask which approach resonates

### Phase 3: Design Presentation
- Present in 200-300 word sections
- Cover: Architecture, components, data flow, error handling, testing
- Ask after each section: "Does this look right so far?"

### Phase 4: Worktree Setup (for implementation)
When design is approved and implementation will follow:
- Announce: "I'm using the Using Git Worktrees skill to set up an isolated workspace."
- Switch to skills/collaboration/using-git-worktrees
- Follow that skill's process for directory selection, safety verification, and setup
- Return here when worktree ready

### Phase 5: Planning Handoff
Ask: "Ready to create the implementation plan?"

When your human partner confirms (any affirmative response):
- Announce: "I'm using the Writing Plans skill to create the implementation plan."
- Switch to skills/collaboration/writing-plans skill
- Create detailed plan in the worktree

## When to Revisit Earlier Phases

**You can and should go backward when:**
- Partner reveals new constraint during Phase 2 or 3 → Return to Phase 1 to understand it
- Validation shows fundamental gap in requirements → Return to Phase 1
- Partner questions approach during Phase 3 → Return to Phase 2 to explore alternatives
- Something doesn't make sense → Go back and clarify

**Don't force forward linearly** when going backward would give better results.

## Related Skills

**During exploration:**
- Technical feasibility analysis: skills/collaboration/delegating-to-codex
- When approaches have genuine trade-offs: skills/architecture/preserving-productive-tensions

**Before proposing changes to existing code:**
- Understand why it exists: skills/research/tracing-knowledge-lineages

## Remember
- One question per message during Phase 1
- Apply YAGNI ruthlessly
- Explore 2-3 alternatives before settling
- Present incrementally, validate as you go
- Go backward when needed - flexibility > rigid progression
- Announce skill usage at start
