---
name: Delegating to Codex
description: Strategic delegation patterns for leveraging Codex MCP server for planning, debugging, and code analysis tasks
when_to_use: When facing ultra-complex planning (50+ steps), persistent debugging requiring deep focus, or large-scale code analysis across 10+ files
version: 1.0.0
---

# Delegating to Codex

## Overview

**Codex is your specialized partner.** OpenAI's Codex excels at deep, focused technical work that requires sustained attention without context drift.

**Core principle:** Delegate the right tasks to Codex based on cognitive load, not convenience. You remain the orchestrator.

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
[1-2 paragraphs: What is this codebase/system?]

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

**Example (Good Prompt):**
```
## Context
NLtoIR pipeline converts natural language to workflow JSON validated against workflow.schema.json.

## Task
Create a comprehensive implementation plan for the slot filling stage that extracts structured parameters from conversational turns.

## Scope
- Read: workflow.schema.json, NL_to_IR_Pipeline_Context.md
- Constraints: Must use DSPy modules, not raw prompts
- Focus: Parameter extraction accuracy (F1 > 0.90)

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

**Anti-pattern (Bad Prompt):**
```
Help me implement slot filling for the NL→IR pipeline.
```
*(Too vague, no context, no expected output format)*

### Phase 3: Invoke Codex via MCP

**Use the Codex MCP tool:**

```
mcp__codex__codex tool:
  prompt: [your structured prompt from Phase 2]
  cwd: /home/heliosuser/piadda-mvp/NLtoIR
  sandbox: read-only  (or workspace-write if needed)
  include-plan-tool: true  (for planning tasks)
```

**Sandbox modes:**
- `read-only` - Code analysis, planning, review (safest)
- `workspace-write` - Implementation, refactoring
- `danger-full-access` - System-level tasks (rarely needed)

**Track conversation ID for follow-up:**
```
Codex returns: {"conversationId": "abc123", "response": "..."}

For follow-up, use mcp__codex__codex-reply:
  conversationId: "abc123"
  prompt: "Refine the plan to use LangGraph instead"
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
   - If gaps: Use `codex-reply` to refine
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
- Use Codex MCP for specialized deep work (planning, architecture analysis)
- Don't delegate orchestration to Codex (you remain orchestrator)

**Example multi-agent strategy:**
```
Complex optimization task with 7 issues:

Agent 1 (Task tool): Database performance investigation
Agent 2 (Task tool): LLM efficiency investigation
Codex (MCP): Create comprehensive optimization plan synthesizing findings
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
