---
name: claudezempic
description: >
  AI workflow audit and optimization compound. Identifies bloated prompts,
  wasted tokens, redundant tool calls, and inefficient patterns. Trims the
  fat from any AI-powered workflow so it runs lean, fast, and cheap.
---

# ClaudeZempic: AI Workflow Optimization

Your AI workflows are bloated. Not all of them. But some are hemorrhaging tokens like a leaky faucet. ClaudeZempic is the audit tool that cuts the fat, tightens the code, and makes your AI runs lean.

This isn't about making your models dumber. It's about making them smarter — by removing the noise that buries the signal.

## The Diagnosis: Signs Your Workflow is Bloated

Before you optimize, you need to know you have a problem. Here are the red flags:

**Token Waste**: You're spending more tokens than necessary to get the same quality output. Your prompts have explanations that could be bullets. Instructions that repeat. Context that doesn't move the needle.

**Slow Responses**: Every call takes forever. You're loading entire files when you need three lines. You're running sequential steps that could run in parallel. The model is wading through your context like it's walking through molasses.

**High Costs**: Your token spend is climbing month over month. You're using Opus when Haiku would do. You're making multiple calls when one would suffice. Your batch jobs are running hot.

**Inconsistent Outputs**: Same prompt, different results. You're over-specifying in some places and under-specifying in others. Your instructions are buried in prose instead of structured clearly.

**Prompt Spaghetti**: You can't remember what's in your prompts. You've added rules on top of rules. Nobody on your team can maintain it. It reads like it was written by ten different people (it probably was).

**Tool Call Waste**: You're calling tools that return information you don't need. You're calling the same tool multiple times when you could batch. You're asking the model to use tools when a simple prompt would work.

If any of these sound familiar, you're bloated. Let's cut it.

## Audit Framework: How to Find the Waste

### Step 1: Map the Current Flow

Write down what your workflow actually does. Not what you think it does.

- **Inputs**: What data comes in? How much? In what format?
- **Prompts**: What exactly are you asking the model? Write it out verbatim.
- **Tool Calls**: What tools are you calling? When? What do you do with the output?
- **Outputs**: What comes out? How much of it is used? How much is discarded?
- **Loops**: Where do you call the model multiple times? Why?

Be honest. If you can't map it clearly, nobody else can either.

### Step 2: Measure Waste

Now count the damage.

- **Prompt Tokens**: How many tokens in your system prompt? Your instructions? Count every word that doesn't change the output.
- **Context Tokens**: How much context are you loading per call? How much of it actually gets used by the model?
- **Tool Tokens**: How many tokens are consumed by tool calls and responses? Are you getting value from each one?
- **Redundancy**: What instructions appear twice? What context is loaded every call? What explanations could be cut?

Use your API logs. Count. Don't estimate.

### Step 3: Score Efficiency

Calculate your output quality versus token spend.

For each workflow variant:
- Measure output quality (accuracy, completeness, correctness)
- Record total tokens spent
- Calculate tokens-per-unit-of-quality

The variant with the best ratio wins.

### Step 4: Identify Cuts

For each bloat signal, ask:

- **Can I delete this?** If the output doesn't change when it's gone, it's dead weight.
- **Can I compress this?** Can 50 words become 10? Can prose become bullets?
- **Can I defer this?** Does it need to load now or can it load on demand?
- **Can I batch this?** Can multiple calls become one?
- **Can I parallelize this?** Can sequential steps run together?

Cut ruthlessly. Test as you go.

## Prompt Optimization Patterns: Lean Language

Your instructions are verbose. They don't need to be.

### Remove Filler

These words add nothing:

```
❌ Please make sure to carefully review the output and ensure it is correct.
✓ Verify the output is correct.

❌ Remember that you should always follow these guidelines:
✓ Guidelines:

❌ It is important that you consider the following:
✓ Consider:
```

Cut "please", "remember", "make sure", "important", "carefully", "consider". The model doesn't need emotional scaffolding.

### Consolidate Redundant Rules

If you're saying the same thing twice, you're saying it once too many.

```
❌
Output should be JSON.
Always format output as JSON.
The response must be in JSON format.
Return JSON only.

✓
Output: JSON only
```

### Use XML for Structure

XML replaces verbose prose with machine-readable clarity:

```
❌
For each item in the list, provide:
1. A brief name (2-3 words)
2. A description (1-2 sentences)
3. A relevance score from 1-10
4. Whether it's actionable (yes/no)

✓
<item>
  <name>...</name>
  <description>...</description>
  <relevance>1-10</relevance>
  <actionable>yes/no</actionable>
</item>
```

XML is tighter, parseable, and leaves no room for ambiguity.

### Front-Load Critical Instructions

Put the most important rule at the top. The model will weight it harder.

```
❌
[30 lines of context]
[20 lines of explanation]
[5 lines] Output JSON only.

✓
Output JSON only.
[context]
[explanation]
```

### Use Examples Instead of Explanations

One example replaces ten sentences of description.

```
❌
When the user provides a date, parse it into ISO 8601 format.
This means year-month-day with hyphens. For example, if they say
"March 15, 2024", convert it to "2024-03-15". If the year is
omitted, assume the current year. If the day is omitted, use the
first of the month.

✓
Parse dates to ISO 8601:
Input: "March 15, 2024" → Output: "2024-03-15"
Input: "March" → Output: "2024-03-01"
```

## Context Window Management: Load What You Need

Your context window is expensive. Use it smartly.

### Load vs Reference

**Load now (in the prompt)**:
- Instructions that change behavior (rules, constraints, format)
- Examples (few-shot learning)
- Critical reference data (< 1KB)

**Reference on demand (via tool call)**:
- Large files (> 10KB)
- Data that might not be needed
- Updated data that changes between calls

### Batch vs Sequential

**One big call**:
- Lower latency
- Coherent reasoning across items
- Wastes tokens if you don't need all responses

**Multiple calls**:
- Higher latency
- Can process items independently
- Each call is smaller

Choose based on dependency. If item B needs output from item A, batch. If they're independent, sequence.

### Caching

Load expensive, static context once. Cache it.

```
- Load: instructions, examples, reference data
- Cache: don't reload between calls
- Invalidate: only when source changes
```

### Context Budgeting

You have a window. Budget it:

```
System prompt:     200 tokens
Input context:     2,000 tokens
User prompt:       500 tokens
Output buffer:     2,000 tokens
---
Remaining:         9,300 tokens (from 12k limit)
```

Stay under your budget. Plan for both typical and worst-case inputs.

## Cost Optimization: Smart Model Selection

Different models cost different amounts. Use the right tool for the job.

### Model Selection

| Task | Model | Why |
|------|-------|-----|
| Simple completion | Haiku | Fast, cheap, good enough |
| Reasoning, analysis | Sonnet | Best balance of cost/capability |
| Complex multi-step | Opus | Only if Sonnet can't do it |
| Batch processing | Haiku | High volume, low cost |

You probably use Opus more than you need to. Try Sonnet. Try Haiku.

### Batch Processing

Don't call the API once per item. Batch.

```
❌ 100 items, 100 calls, 100 API invocations
✓ 100 items, 1 call with batch processing
```

Batch costs less and runs faster.

### Response Length Control

Don't let the model ramble.

```
❌ "Generate a comprehensive analysis..."
✓ "Generate a 2-sentence summary..."
```

Constrain output length. You save tokens per response.

### Tools: When to Use Them

Use tools when:
- You need external data
- You need to take action
- The output format matters (structured data)

Don't use tools when:
- The model can answer from its weights
- You're just asking it to format something
- You don't need the overhead

One tool call costs tokens. Make it count.

## Workflow Patterns: Lean Architecture

Different problems need different flow patterns.

### Single-Shot

One prompt, one response. Fastest, cheapest.

Use when: The model can solve it in one pass.

### Multi-Turn

Multiple back-and-forth exchanges. Conversation.

Use when: You need refinement, clarification, or iterative improvement.

Cost: Higher (context grows each turn).

Optimization: Summarize old context. Don't repeat it.

### Agent Loops

The model calls tools, acts, calls tools again. Autonomous.

Use when: The task requires multiple steps and decision-making.

Cost: Highest (multiple model calls + tool calls).

Optimization: Define clear exit conditions. Cap the loop iterations. Pre-filter tool results.

### Parallel Processing

Multiple independent subtasks run simultaneously.

Use when: You have many independent items to process.

Cost: Same total tokens, but distributed.

Benefit: Faster wall-clock time if you have parallel capacity.

### Chain vs Single Call

```
❌ Chain: Step 1 → Step 2 → Step 3 (3 API calls)
✓ Single: All steps in one prompt (1 API call)
```

If the steps depend on each other, you must chain. If they're independent, consolidate.

## Error Handling Without Waste

Don't validate by re-running.

```
❌ Generate output → Validate → If invalid, regenerate
   (wastes tokens regenerating)

✓ Generate output → Check syntax → If invalid, fix in post
   (uses cheap post-processing)
```

Use cheaper validation strategies:

- Syntax check (regex, not model)
- Schema validation (JSON schema, not model)
- Spot-check (sample, don't validate all)
- Regenerate only on real failure (logic error, not formatting)

## Anti-Patterns: The Biggest Wastes

### Load Entire Codebases When You Need One File

You have a 10,000 line codebase. You need the `login` function. Load the one file, not the whole thing.

**Cost**: 50,000 tokens of waste per call.

### System Prompts That Are Novels

Your system prompt is 2,000 tokens. It has rules you never enforce. Instructions nobody reads. Cut it in half. Then cut it again.

**Cost**: 2,000 wasted tokens per call. Scales across millions of calls.

### "Think Step by Step" When It Doesn't Need To

The model doesn't need permission to think. It does it automatically. This prompt just burns tokens.

```
❌ "Think step by step and then provide the answer."
✓ "Answer:"
```

**Cost**: 10-20 tokens of waste per call.

### Re-explaining Context Every Turn

In a multi-turn conversation, you explain the context in the system prompt AND the user message. The model already has it.

```
❌
System: "You are a code reviewer. Your job is to review code..."
User: "Review this code. Remember, you are a code reviewer..."

✓
System: "You are a code reviewer."
User: "Review this code."
```

**Cost**: 200+ wasted tokens per turn in long conversations.

### Tool Calls That Return Unused Data

You call a tool that returns 500 tokens of data. You use 50 tokens. You waste 450.

Filter on the tool side. Constrain responses. Call smaller tools.

## Anti-Patterns: Workflow Waste

### Sequential When Parallel Would Work

```
❌ For each of 100 items: call model (100 sequential calls)
✓ Batch 100 items, call model once
```

**Cost**: 50x more latency, same tokens.

### Multiple Calls Instead of One

```
❌ Call 1: Analyze data
   Call 2: Generate summary
   Call 3: Format output

✓ Call 1: Analyze, summarize, and format
```

**Cost**: 3x API overhead + 3x context setup.

### Loading Context That Never Gets Used

You load reference data that the model never references.

Track it. If it's not used in 90% of calls, move it to on-demand loading.

## The ClaudeZempic Audit Template

Use this template to audit any workflow.

```
# Workflow Audit: [Workflow Name]

## Current State
- **Purpose**: [What does this workflow do?]
- **Frequency**: [How often does it run?]
- **Volume**: [How many calls per period?]

## Flow Map
[Diagram: inputs → prompt → tools → outputs]

Input example:
```
[sample input]
```

Prompt (exact):
```
[exact system prompt]
[exact user prompt]
```

Tool calls:
- Tool 1: [when] → [result size]
- Tool 2: [when] → [result size]

Output example:
```
[sample output]
```

## Metrics
- Avg input tokens: [count]
- Avg prompt tokens: [count]
- Avg output tokens: [count]
- Total avg tokens per call: [count]
- Calls per day: [count]
- Daily token spend: [count]
- Monthly cost: [count]

## Quality Score
- Metric 1: [e.g., accuracy 95%]
- Metric 2: [e.g., latency 2s]
- Metric 3: [e.g., user satisfaction 4.2/5]

## Waste Detected

| Waste Type | Location | Severity | Tokens/Call |
|-----------|----------|----------|------------|
| [e.g., redundant instructions] | [section] | [high/med/low] | [count] |

Total waste: [count] tokens/call ([percentage]% of total)

## Optimization Plan

| Cut | Type | Impact | Risk | Effort |
|-----|------|--------|------|--------|
| [e.g., remove explanations] | [compression] | -[count] tokens | low | low |
| [e.g., batch tool calls] | [batching] | -[count] tokens | low | med |
| [e.g., use Haiku instead of Opus] | [model] | -[count] cost | med | low |

## Before/After

**Before**:
- Tokens per call: [count]
- Cost per call: $[amount]
- Latency: [time]

**After** (projected):
- Tokens per call: [count] (-[percentage]%)
- Cost per call: $[amount] (-[percentage]%)
- Latency: [time] (-[percentage]%)

**Quality impact**: [unchanged / improved / slight decrease]

## Implementation Checklist

- [ ] Apply compression cuts
- [ ] Batch tool calls
- [ ] Switch models if needed
- [ ] Test output quality
- [ ] Measure actual vs projected
- [ ] Iterate

## Sign-off

Audited by: [name]
Date: [date]
Status: [ready to implement / on hold / monitoring]
```

## Running the Audit

1. **Pick a workflow** that feels bloated or expensive.
2. **Instrument it**: Log prompts, tokens, tool calls, outputs.
3. **Gather 10-50 real examples** (not synthetic).
4. **Fill the template** above with real data.
5. **Identify the top 3 waste items** (biggest token savings).
6. **Cut ruthlessly**: Apply the cuts one at a time.
7. **Measure the impact**: Compare before/after quality and cost.
8. **Iterate**: Keep what works. Revert what breaks.

## The ClaudeZempic Philosophy

Your AI workflows are tools. Tools should be efficient. Bloated workflows are broken workflows.

You don't need a bigger model. You need tighter prompts.

You don't need more context. You need better questions.

You don't need more tools. You need smarter tool selection.

Cut the fat. Keep the muscle. Run lean.

---

<!-- Open RX | claudezempic | By CutTheChexx -->
