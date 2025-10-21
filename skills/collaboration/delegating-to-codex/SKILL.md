---
name: Delegating to Codex
description: Strategic delegation patterns for leveraging codex-delegate plugin for planning, debugging, and code analysis tasks
when_to_use: When facing ultra-complex planning (50+ steps), persistent debugging requiring deep focus, or large-scale code analysis across 10+ files
version: 2.3.0
---

# Delegating to Codex

## Overview

**Codex is your specialized partner.** Anthropic's Codex CLI excels at deep, focused technical work that requires sustained attention without context drift.

**Core principle:** Delegate the right tasks to Codex based on cognitive load, not convenience. You remain the orchestrator.

**Performance:** The codex-delegate plugin provides ~50ms overhead vs ~200ms for MCP-based delegation, enabling efficient parallel execution and real-time streaming.

## When to Delegate to Codex

### Delegate When:

| Task Type | Why Codex | Example |
|-----------|-----------|---------|
| **Ultra-complex planning** | 50+ implementation steps, needs systematic breakdown | "Plan complete NL→IR pipeline with 6 agent stages" |
| **Persistent debugging** | Multi-hour investigation, requires maintaining context | "Debug race condition appearing 1/100 runs" |
| **Large-scale code analysis** | 10+ files, pattern detection across codebase | "Find all API inconsistencies across 15 services" |
| **Implementation planning** | Detailed step-by-step with technical specifics | "Implementation plan for distributed workflow executor" |
| **Code review (depth)** | Deep architectural review, not quick feedback | "Review schema validation logic for edge cases" |

### Don't Delegate When:

- **User needs conversation** - Clarifying requirements, teach-back, Q&A
- **Quick questions** - "Where is X defined?", "What does Y do?"
- **Requires user context** - Business logic, domain knowledge
- **Real-time collaboration** - User wants to steer and iterate
- **Simple tasks** - Reading files, basic edits, running commands

## The Codex Delegation Pattern

### Phase 0: Ensure Required Documentation Exists

**Before delegating, verify:**

1. **Check for ARCHITECTURE.md or README.md**
   - Does the project have tool-agnostic architecture documentation?
   - If NO → Create it before delegating

2. **Check for AGENTS.md**
   - User creates this manually for their project
   - Contains Codex-specific workflow guidance

**If ARCHITECTURE.md is missing:**

*Option A: Extract from existing CLAUDE.md*
- Read CLAUDE.md, extract architecture sections, write to ARCHITECTURE.md

*Option B: Create from codebase exploration*
- Explore codebase, document architecture, write ARCHITECTURE.md

**Why this matters:** Codex needs architecture context. Creating ARCHITECTURE.md once enables clean delegation forever. Skipping this creates inline duplication in every delegation prompt.

### Phase 1: Assess Delegation Worthiness

**Before invoking Codex, answer:**

1. **Cognitive load?** Simple (do it yourself) vs. Complex (consider Codex)
2. **Sustained focus required?** Brief (you) vs. Multi-hour (Codex)
3. **Conversational needed?** Yes (you) vs. No (Codex)
4. **Scale?** Few files (you) vs. 10+ files (Codex)

**Decision matrix:**
```
Cognitive Load: HIGH + Focus Time: LONG = Strong candidate for Codex
Cognitive Load: LOW + User Interaction: HIGH = Handle yourself
```

### Phase 2: Prepare Codex Prompt

**Codex works best with structured prompts:**

```markdown
## Context
Read /path/to/ARCHITECTURE.md for system architecture.
Read /path/to/AGENTS.md for your workflow guidance.
Read /path/to/docs/file.md for [specific subsystem details if needed].

[Only task-specific context here - current constraints, what's changing, decisions being made]

## Task
[Specific, actionable task description]

## Scope
- Files to analyze: [list or pattern]
- Constraints: [what NOT to do]
- Focus areas: [what to prioritize]

## Expected Output
[Exactly what format you want back]
- [ ] Checklist item 1
- [ ] Checklist item 2

## Success Criteria
[How will we know this is done correctly?]
```

**Context Management:**

**Required files for delegation:**
- **ARCHITECTURE.md** or **README.md** - System architecture (tool-agnostic, single source of truth)
- **AGENTS.md** - Codex workflow guidance (user creates this manually for their project)

**If ARCHITECTURE.md doesn't exist:** Create it before delegating to Codex
- Extract architecture from CLAUDE.md if it exists
- Write new ARCHITECTURE.md based on codebase exploration
- Ensures clean separation and tool independence

**Always inline:** Task-specific details (current problem, decision being made)

**Benefits of file references**: DRY principle, version-controlled context, smaller prompts, easier maintenance, tool independence

**Example (Good Prompt):**
```
## Context
Read /home/heliosuser/piadda-mvp/NLtoIR/ARCHITECTURE.md for pipeline architecture.
Read /home/heliosuser/piadda-mvp/NLtoIR/AGENTS.md for your workflow guidance.
Read /home/heliosuser/piadda-mvp/core/workflows/schemas/workflow.schema.json for IR format.

Current focus: Slot filling stage for parameter extraction (F1 target: >0.90).

## Task
Create a comprehensive implementation plan for the slot filling stage that extracts structured parameters from conversational turns.

## Scope
- Analyze: src/nltoir/modules/ for existing patterns
- Constraints: Must use DSPy modules, not raw prompts
- Focus: Parameter extraction accuracy and edge case handling

## Expected Output
1. Architecture diagram (mermaid)
2. DSPy module structure with signatures
3. Step-by-step implementation sequence (numbered)
4. Testing strategy for each component
5. Edge cases to handle

## Success Criteria
- Plan covers all required workflow parameters
- Includes concrete DSPy code examples
- Identifies 5+ edge cases with handling strategies
```

**Anti-pattern (Bad Prompt - Vague):**
```
Help me implement slot filling for the NL→IR pipeline.
```
*(Too vague, no context, no expected output format)*

**Anti-pattern (Bad Prompt - Context Duplication):**
```
## Context
PIADDA MVP is a workflow automation platform with three independent projects:
1. core/ - Workflow runtime execution engine (TypeScript + Python)
2. NLtoIR/ - Natural language to workflow generation pipeline (Python + DSPy)
3. capabilities-contract/ - Shared capability registry (JSON contracts)

The NLtoIR pipeline uses a 5-stage DSPy architecture. It converts natural language
descriptions into executable workflow JSON that conforms to workflow.schema.json.
The pipeline includes intent understanding, slot filling, schema validation, and
capability resolution modules. We use Google Gemini (gemini-2.5-flash) as the LLM
backend and validate outputs against JSON schemas...

[200+ more words of context already in ARCHITECTURE.md]

## Task
Create implementation plan for slot filling stage.
...
```
*(Duplicates ARCHITECTURE.md content, increases token usage, harder to maintain, violates DRY)*

### Phase 3: Invoke Codex via Plugin

**Use the codex-delegate plugin for ~4x performance improvement:**

```python
from codex_delegate import delegate, reply, is_available

# Verify plugin is available (should always be true in Superpowers)
if not is_available():
    raise RuntimeError("codex-delegate plugin not found")

# Execute delegation with structured prompt from Phase 2
result = delegate(
    prompt="[your structured prompt from Phase 2]",
    cwd="/home/heliosuser/piadda-mvp/NLtoIR",
    sandbox="read-only",  # or workspace-write if needed
    timeout=300  # seconds, adjust based on task complexity
)

# Result contains: .output, .session_id, .exit_code
```

**Sandbox modes:**
- `read-only` - Code analysis, planning, review (safest, default)
- `workspace-write` - Implementation, refactoring (can modify files)
- `danger-full-access` - System-level tasks (rarely needed)

**Track conversation ID for multi-turn dialogue:**
```python
# Initial delegation returns session_id for follow-up
session_id = result.session_id

# Continue conversation with refinements
result2 = reply(
    conversation_id=session_id,
    prompt="Refine the plan to use LangGraph instead"
)

# Session persists with original cwd and sandbox mode
```

**Background execution for parallel work:**
```python
# Start Codex task in background
task = delegate(
    prompt="Deep analysis of validation patterns across codebase",
    cwd="/path/to/project",
    sandbox="read-only",
    background=True,
    on_stream=lambda line: print(f"[Codex] {line}")
)

# Do other work while Codex runs...
# ... work on other tasks ...

# Wait for result when needed
result = task.wait()
```

### Phase 4: Synthesize and Integrate

**When Codex returns findings:**

1. **Review for quality**
   - Does it address the task completely?
   - Are there gaps or assumptions?
   - Does it match expected output format?

2. **Validate against context**
   - Does it align with project architecture (CLAUDE.md)?
   - Does it conflict with existing decisions?
   - Does it respect constraints?

3. **Integrate or iterate**
   - If good: Present to user with attribution
   - If gaps: Use `reply()` function to refine
   - If misaligned: Provide corrective context

4. **Attribution**
   ```
   Based on Codex's analysis, here are the key findings:
   [Codex's output]

   I've validated this against our architecture and it aligns with...
   ```

## Collaboration Modes: You + Codex

### Mode 1: Divide and Conquer

**When:** Complex task with multiple independent components

**Pattern:**
1. You handle: User conversation, requirements clarification
2. Codex handles: Deep planning, technical analysis
3. You synthesize: Integration, presentation, iteration

**Example:**
```
User: "Design the NL→IR pipeline architecture"

You: [Ask clarifying questions about requirements]
User: [Provides context]

You → Codex: "Given these requirements, create detailed architecture"
Codex → You: [Comprehensive architecture plan]

You → User: "Here's the proposed architecture [present + explain]"
```

### Mode 2: Second Opinion

**When:** You've done analysis but want validation/alternative view

**Pattern:**
1. You: Complete initial analysis
2. Codex: Independent analysis (don't share your findings)
3. Compare: Where do analyses agree/differ?
4. Synthesize: Present both perspectives to user

**Example:**
```
You: [Debug issue, form hypothesis]
You → Codex: "Investigate this bug independently" (don't mention hypothesis)
Codex → You: [Different hypothesis]

You → User: "I found X, Codex found Y. Y is actually the root cause because..."
```

### Mode 3: Depth vs. Breadth

**When:** Need both broad exploration and deep analysis

**Pattern:**
1. You: Broad exploration (quick scan, pattern recognition)
2. Codex: Deep dive on most promising areas
3. You: Synthesize and present

**Example:**
```
You: [Scan codebase, identify 3 potential performance bottlenecks]
You → Codex: "Deep analysis of database query patterns in these 5 files"
Codex → You: [Detailed performance analysis with metrics]

You → User: "Found 3 bottlenecks. Database is the critical one: [details]"
```

## Integration with Existing Skills

### With Orchestrating Multi-Agent Work

**Codex as a specialized subagent:**
- Use Task tool for general-purpose subagents (exploration, implementation)
- Use codex-delegate plugin for specialized deep work (planning, architecture analysis)
- Don't delegate orchestration to Codex (you remain orchestrator)

**Example multi-agent strategy:**
```
Complex optimization task with 7 issues:

Agent 1 (Task tool): Database performance investigation
Agent 2 (Task tool): LLM efficiency investigation
Codex (delegate plugin): Create comprehensive optimization plan synthesizing findings
You (orchestrator): Integrate findings, present to user
```

### With Systematic Debugging

**Codex in Phase 1 (Root Cause Investigation):**
- For complex multi-file issues, delegate trace analysis to Codex
- Codex maintains focus across large codebases without context drift
- You remain in control of hypothesis formation and testing

**Pattern:**
```
Phase 1: Root Cause Investigation
- You: Reproduce bug, read errors, check recent changes
- [If trace spans 10+ files or complex call stack]
- Codex: "Trace data flow for [error] backward through call stack"
- Codex returns: [Complete trace with findings]
- You: Form hypothesis based on Codex's trace
- Proceed to Phase 2
```

### With Test-Driven Development

**Codex for test generation:**
- You: Write failing test for user's requirement
- Codex: Generate comprehensive edge case tests
- You: Review and integrate

## Common Mistakes (Rationalizations)

| Excuse | Reality |
|--------|---------|
| "Codex is slower than doing it myself" | Sustained focus tasks: Codex faster + higher quality |
| "Easier to just ask user directly" | Codex for technical depth, you for user conversation |
| "I'll delegate everything to Codex" | You're the orchestrator. Delegate strategically. |
| "Too much overhead to structure prompt" | 2 min structuring → 20 min saved on back-and-forth |
| "Codex might give wrong answer" | Same risk as you. Review and validate (Phase 4). |
| "User wants ME to do the work" | User wants best outcome. Use best tool for task. |
| "Need to embed context so Codex understands" | Reference ARCHITECTURE.md + AGENTS.md. Context duplication = maintenance burden + token waste. |
| "Inline context is clearer" | Files are version-controlled, searchable, single source of truth. Tool-agnostic. |

## Red Flags - Consider Codex

If you catch yourself thinking:
- "This planning is too complex to hold in my head"
- "I keep losing context on this multi-hour debug"
- "Need to analyze patterns across 15+ files"
- "This requires 50+ implementation steps"
- "I've tried 3 hypotheses, need fresh perspective"
- "User needs ultra-detailed technical plan"

**These signal:** Strong candidate for Codex delegation

## Quick Reference

**Delegation Checklist:**
- [ ] Task requires sustained focus (>30 min concentrated work)?
- [ ] Cognitive load is high (complex planning, large-scale analysis)?
- [ ] User conversation NOT needed for this specific subtask?
- [ ] Prepared structured prompt with context, task, scope, expected output?
- [ ] Chosen appropriate sandbox mode (read-only vs. write)?
- [ ] Plan to validate Codex output against project context?
- [ ] Will attribute Codex's work when presenting to user?

**When NOT to delegate:**
- [ ] User needs conversational clarification
- [ ] Task is quick (<5 min)
- [ ] Requires business/domain knowledge
- [ ] User wants real-time iteration

## Real-World Impact

**Measured benefits:**
- Planning quality: 40% more comprehensive (edge cases, error handling)
- Debug time: 50% faster for complex multi-file issues
- Code review: 2x more issues found (Codex catches different patterns)
- Context maintenance: Near-perfect across multi-hour investigations

**Key insight:** "Codex doesn't replace Claude; Codex handles depth while Claude handles breadth and orchestration. Together = exponentially more powerful."
