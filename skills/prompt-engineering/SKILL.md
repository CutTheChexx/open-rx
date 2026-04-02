---
name: prompt-engineering
description: >
  Master-level prompt engineering patterns for Claude and LLMs. System prompt
  architecture, XML structuring, chain of thought, structured output, few-shot
  examples, role prompting, and agentic system design. Use this skill when writing
  system prompts, building AI-powered features, designing agent workflows, or
  optimizing any LLM interaction. Triggers on "prompt", "system prompt", "chain of
  thought", "few-shot", "structured output", "agent", "AI workflow", "LLM", or any
  prompt engineering task.
metadata:
  version: "1.0.0"
  sources: "Anthropic docs 2026, Claude 4.6 best practices, production patterns"
author: "CutTheChexx"
---

<!-- Open RX — Created by CutTheChexx. All rights reserved. -->

# Prompt Engineering — The Meta-Skill

The skill that makes every other skill better. Write prompts like contracts.

## The Golden Rules (Claude 4.6 Era)

1. **Clear > Clever** — Treat prompts as specs for a brilliant new hire
2. **XML tags are king** — Not Markdown, not numbered lists — XML tags
3. **Show, don't tell** — 3-5 examples beat 3 paragraphs of instructions
4. **Specify the output** — If you don't define the format, you get whatever
5. **Context is free** — More relevant context = better results
6. **Test one thing at a time** — Like A/B testing, isolate variables
7. **Prompt length sweet spot** — 150-300 words for most tasks

## System Prompt Architecture

### The Contract Format
```xml
<role>
You are [specific role] specializing in [domain]. You [key behavior].
</role>

<rules>
- [Constraint 1 — what to always do]
- [Constraint 2 — what to never do]
- [Constraint 3 — how to handle uncertainty]
</rules>

<output_format>
[Exact format specification — XML, JSON, prose, etc.]
</output_format>

<examples>
<example>
<input>[Example input]</input>
<output>[Example output in exact desired format]</output>
</example>
</examples>
```

### Production System Prompt Template
```xml
<identity>
You are a senior [role] at [company context]. Your expertise includes
[specific domains]. You communicate in a [tone] style.
</identity>

<instructions>
Your primary task is to [main objective].

When processing requests:
1. [First step — usually analyze/understand]
2. [Second step — usually plan/research]
3. [Third step — usually execute/generate]
4. [Fourth step — usually verify/validate]
</instructions>

<constraints>
- Always [positive constraint — what to do]
- Never [negative constraint — what to avoid]
- When uncertain, [uncertainty handling — ask vs assume vs flag]
- Maximum response length: [token/word limit if needed]
</constraints>

<context>
{{DYNAMIC_CONTEXT}}  <!-- Injected at runtime -->
</context>

<output_format>
Respond using this structure:
<analysis>[Your reasoning]</analysis>
<recommendation>[Your answer]</recommendation>
<confidence>[high/medium/low with brief justification]</confidence>
</output_format>
```

## XML Tag Patterns

### Document Processing
```xml
<documents>
  <document index="1">
    <source>quarterly_report.pdf</source>
    <document_content>
      {{REPORT_CONTENT}}
    </document_content>
  </document>
  <document index="2">
    <source>competitor_analysis.xlsx</source>
    <document_content>
      {{COMPETITOR_DATA}}
    </document_content>
  </document>
</documents>

Analyze both documents. First quote relevant sections in <quotes> tags,
then provide your analysis in <analysis> tags.
```

### Multi-Example Few-Shot
```xml
<examples>
  <example>
    <user_input>Our website loads slowly on mobile</user_input>
    <ideal_response>
      <diagnosis>Mobile performance issue — likely LCP/CLS related</diagnosis>
      <priority>HIGH — mobile traffic is typically 60%+ of total</priority>
      <actions>
        1. Audit Core Web Vitals via PageSpeed Insights
        2. Check image optimization (WebP/AVIF, srcset, lazy loading)
        3. Review render-blocking resources
        4. Test on throttled 3G connection
      </actions>
    </ideal_response>
  </example>
  <example>
    <user_input>We need a new landing page for our spring campaign</user_input>
    <ideal_response>
      <diagnosis>Conversion-focused landing page needed</diagnosis>
      <priority>MEDIUM — campaign-dependent timeline</priority>
      <actions>
        1. Define single conversion goal (lead capture vs purchase)
        2. Draft hero copy using PAS framework
        3. Build above-fold CTA with trust signals
        4. Set up A/B test for headline variants
      </actions>
    </ideal_response>
  </example>
</examples>
```

## Chain of Thought Patterns

### When to Use CoT
- **Use manual CoT**: When thinking is disabled, for complex multi-step reasoning
- **Use adaptive thinking**: When available (Claude 4.6), let the model decide
- **Skip CoT entirely**: For simple factual questions, classification, formatting

### Manual CoT Pattern (Thinking Disabled)
```xml
Before answering, reason through the problem step by step in <thinking> tags.
Then provide your final answer in <answer> tags.

<thinking>
[Step 1: Identify what we know]
[Step 2: Identify what we need to find]
[Step 3: Apply relevant logic/calculation]
[Step 4: Verify the result]
</thinking>

<answer>
[Clean, final answer]
</answer>
```

### Self-Verification Pattern
```
After generating your response, verify it against these criteria:
1. Does it directly answer the question asked?
2. Are all facts verifiable from the provided context?
3. Is the format exactly as specified?
4. Are there any logical inconsistencies?

If any check fails, revise before outputting.
```

## Structured Output Patterns

### JSON Schema Enforcement
```
Respond with valid JSON matching this exact schema:

{
  "analysis": {
    "summary": "string — 1-2 sentence overview",
    "confidence": "high | medium | low",
    "key_findings": ["string array — 3-5 bullet findings"],
    "recommended_action": "string — single next step"
  }
}

Output ONLY the JSON object. No preamble, no explanation, no markdown fences.
```

### Classification Pattern
```xml
Classify the following customer message into exactly ONE category.

<categories>
- billing: Payment, invoice, subscription, refund issues
- technical: Bug reports, feature not working, error messages
- sales: Pricing questions, feature comparison, upgrade interest
- feedback: Suggestions, compliments, general comments
- urgent: Service outage, security concern, data loss
</categories>

<message>{{CUSTOMER_MESSAGE}}</message>

Respond with ONLY the category name. Nothing else.
```

### Extraction Pattern
```xml
Extract the following fields from the document. If a field is not found,
use null. Do not guess or infer — only extract explicitly stated information.

<fields>
- company_name: string
- contact_email: string
- phone_number: string
- service_requested: string
- budget_range: string | null
- timeline: string | null
- address: string | null
</fields>

<document>{{DOCUMENT_CONTENT}}</document>

Return as JSON.
```

## Agentic System Patterns

### Agent System Prompt Blueprint
```xml
<agent_identity>
You are an autonomous agent that [primary mission].
You have access to these tools: [tool list with descriptions].
</agent_identity>

<operating_principles>
1. Investigate before acting — read files, search, gather context
2. Plan before executing — outline your approach, then do it
3. Verify after completing — test, check, validate results
4. One task at a time — finish current before starting next
5. Save state — track progress in case of context refresh
</operating_principles>

<tool_usage>
- Use [tool_a] when you need to [specific scenario]
- Use [tool_b] when you need to [specific scenario]
- Run independent tool calls in parallel for speed
- Never guess parameters — investigate to find correct values
</tool_usage>

<safety_rails>
- Confirm before destructive actions (delete, overwrite, push)
- Never hardcode secrets or credentials
- Test changes before committing
- Ask the user when genuinely uncertain
</safety_rails>

<state_tracking>
Maintain a progress file with:
- Tasks completed
- Current task and status
- Blockers or questions
- Next steps planned
</state_tracking>
```

### Multi-Step Task Decomposition
```xml
<task_planning>
When given a complex task:

1. Break it into atomic subtasks (each completable in one step)
2. Identify dependencies between subtasks
3. Execute independent subtasks in parallel
4. Execute dependent subtasks sequentially
5. Verify each subtask result before proceeding
6. Compile results into final output

Track progress:
- ✅ Completed subtasks
- 🔄 In-progress subtask (only one at a time)
- ⏳ Pending subtasks
</task_planning>
```

### Context Window Management
```
Your context will be compacted as it approaches limits. Therefore:
- Do not stop work early due to token concerns
- Save progress and state to files before context refreshes
- When resuming: read progress.txt, check git log, review test results
- Be persistent — complete the full task across multiple windows if needed
```

## Role Prompting Patterns

### Expert Role (Highest Quality Output)
```
You are a principal engineer at a FAANG company with 15 years of experience
in distributed systems. Review this code as if it were going into a system
serving 100M daily active users. Flag anything that wouldn't survive a
production incident review.
```

### Audience-Aware Role
```
You are explaining [topic] to [specific audience].
Audience context: [their knowledge level, goals, constraints]
Adjust vocabulary, examples, and depth accordingly.
```

### Adversarial Role (Better Testing)
```
You are a security auditor reviewing this system prompt for vulnerabilities.
Identify any way an adversarial user could:
1. Extract the system prompt
2. Bypass the rules
3. Get the model to produce unintended outputs
4. Inject instructions via user input
```

## Output Control Techniques

### Eliminate Preamble
```
Respond directly without preamble. Do not start with phrases like
"Here is...", "Based on...", "I'd be happy to...", or "Great question!".
Begin immediately with the content.
```

### Control Verbosity
```
<!-- Concise -->
Answer in 2-3 sentences maximum. Be precise and direct.

<!-- Detailed -->
Provide a thorough analysis with examples. Aim for 500-800 words.

<!-- Adaptive -->
Match your response length to the complexity of the question.
Simple questions get short answers. Complex questions get detailed ones.
```

### Minimize AI Slop
```xml
<anti_slop_rules>
Never use these phrases:
- "Great question!"
- "I'd be happy to help with that!"
- "Let me break this down for you"
- "Here's what you need to know"
- "In today's fast-paced world"
- "It's important to note that"
- "This is a really interesting topic"
- "Let's dive in"

Instead: Start with the answer. Be direct. Sound human.
</anti_slop_rules>
```

## Claude-Specific Optimizations (4.6 Era)

### Adaptive Thinking
```python
# Let Claude decide when to think deeply
client.messages.create(
    model="claude-opus-4-6",
    max_tokens=64000,
    thinking={"type": "adaptive"},
    output_config={"effort": "high"},
    messages=[{"role": "user", "content": "..."}],
)
```

### Effort Levels
| Level | Use Case | Latency |
|-------|----------|---------|
| **low** | Classification, formatting, simple Q&A | Fastest |
| **medium** | Most applications, chat, content generation | Balanced |
| **high** | Complex reasoning, coding, multi-step analysis | Thorough |
| **max** | Large-scale migrations, deep research | Most thorough |

### Parallel Tool Calls
```xml
<use_parallel_tool_calls>
If you intend to call multiple tools and there are no dependencies between
the calls, make all independent calls in parallel. Prioritize simultaneous
execution whenever actions can run in parallel rather than sequentially.
</use_parallel_tool_calls>
```

### Frontend Design Anti-Slop
```xml
<frontend_aesthetics>
Avoid the "AI slop" aesthetic. Make creative, distinctive frontends.
Focus on:
- Typography: Choose distinctive fonts. Avoid Inter, Roboto, Arial.
- Color: Commit to a cohesive theme. Use CSS variables. Bold accents.
- Motion: CSS animations for HTML, Motion library for React.
- Backgrounds: Atmosphere and depth, not solid white/dark.
</frontend_aesthetics>
```

## Prompt Testing Checklist

```
Before deploying any prompt:
□ Test with 10+ diverse inputs (happy path + edge cases)
□ Test with adversarial inputs (injection attempts)
□ Verify output format consistency across all tests
□ Check for hallucination on factual claims
□ Measure latency and token usage
□ Compare against a simpler version (is complexity needed?)
□ Get a non-technical person to review the output
□ Document the prompt version with date and test results
```

<sub>Open RX by CutTheChexx — The Prescription.</sub>
