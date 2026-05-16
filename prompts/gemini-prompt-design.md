# Prompt Design Guide

*Prompt design* is the process of creating prompts — natural language requests that elicit accurate, high-quality responses from a language model. Prompt engineering is iterative; treat these guidelines as starting points and refine based on your own use cases and observed results.

---

## 1. Clear and Specific Instructions

The most effective way to customize model behavior is to give it clear, specific instructions. Instructions can be a question, a step-by-step task, or a full mapping of a user's experience and mindset.

### Input Types

| Input type | Example prompt |
|---|---|
| **Question** | "What's a good name for a flower shop that specializes in dried bouquets? List 5 options." |
| **Task** | "Give me a simple list of 5 things I must bring on a camping trip." |
| **Entity** | "Classify the following items as [large, small]: Elephant, Mouse, Snail." |
| **Completion** | Partial content that the model finishes based on your pattern. |

### Partial Input Completion

Generative models work like advanced auto-complete. When you supply partial content — plus examples or context — the model continues in the same pattern.

**Example:** Instead of describing a JSON output format in prose, show one completed example and leave the next one open:

```
Order: Give me a cheeseburger and fries
Output:
{ "cheeseburger": 1, "fries": 1 }

Order: I want two burgers, a drink, and fries.
Output:
```

The model completes the second block in the same shape. For complex JSON schemas, prefer structured output features over prompt-based formatting.

### Constraints

Tell the model what to do *and* what not to do. Length, tone, and scope all belong here.

> "Summarize this text in **one sentence**."

### Response Format

Specify the shape of the response — table, bulleted list, elevator pitch, keywords, paragraph, etc. You can do this in a system instruction ("All questions should be answered comprehensively unless the user asks for a concise response") or inline in the user prompt.

You can also format via completion — start the output pattern yourself and let the model continue:

```
Create an outline for an essay about hummingbirds.
I. Introduction
    *
```

---

## 2. Zero-shot vs. Few-shot Prompts

- **Zero-shot:** No examples.
- **Few-shot:** One or more examples showing the model what "right" looks like.

**Always include few-shot examples when you can.** They regulate formatting, phrasing, scoping, and general response patterns. If examples are clear enough, you can often drop the explicit instructions entirely.

### Tips

- **Optimal count:** Experiment. Too few and the model misses the pattern; too many and it overfits to your examples.
- **Consistent formatting:** Keep structure, XML tags, whitespace, newlines, and delimiters identical across examples. Inconsistency is the main reason few-shot prompts misbehave.
- **Specific and varied examples:** Narrow the model's focus while still covering the range of valid inputs.

---

## 3. Add Context

Don't assume the model has the information it needs. Include instructions, reference text, and constraints directly in the prompt.

For example, a generic question like *"The light on my router is blinking yellow — what do I do?"* gets a generic answer. Paste the router's troubleshooting guide into the prompt and instruct the model to answer from it, and you get the exact, correct step.

```
Answer the question using the text below. Respond with only the text provided.
Question: [user question]
Text: [reference material]
```

---

## 4. Break Down Complex Prompts

For complex use cases, decompose the prompt:

1. **Break down instructions** — one instruction per prompt; route based on user input.
2. **Chain prompts** — output of one prompt becomes input to the next.
3. **Aggregate responses** — run parallel prompts over different data slices, then combine.

---

## 5. Experiment with Model Parameters

Each model call includes parameters that affect the response. Tune these for your task.

| Parameter | Effect |
|---|---|
| **Max output tokens** | Caps response length. ~100 tokens ≈ 60–80 words. |
| **Temperature** | Randomness of token selection. `0` is deterministic. Higher = more creative/diverse. |
| **topK** | Sample from the top-K most probable tokens. `1` = greedy decoding. |
| **topP** | Sample from the smallest set of tokens whose cumulative probability ≥ topP. Default `0.95`. |
| **stop_sequences** | Character sequence that stops generation. Avoid sequences likely to appear naturally in output. |

---

## 6. Iteration Strategies

Prompt design usually takes a few passes. When you're not getting what you want:

1. **Rephrase** — same meaning, different words. Models respond differently to surface changes.
   - *"How do I bake a pie?"* vs. *"Suggest a recipe for a pie."* vs. *"What's a good pie recipe?"*
2. **Switch to an analogous task** — if direct instructions fail, try a different framing that produces the same result. E.g., reframe "categorize this book" as a multiple-choice question.
3. **Change content order** — rearrange `[examples] [context] [input]` and see which ordering performs best.

---

## 7. Fallback Responses

If the model returns a fallback like "I'm not able to help with that," try raising the temperature.

---

## 8. Core Principles for Advanced Models

Modern reasoning-heavy models respond best to prompts that are direct, well-structured, and explicit.

- **Be precise and direct.** State the goal. Drop persuasive filler.
- **Use consistent structure.** Separate prompt sections with delimiters — XML tags (`<context>`, `<task>`) or Markdown headings. Pick one and stick with it inside a prompt.
- **Define ambiguous terms.** Don't leave the model to guess.
- **Control verbosity explicitly.** Capable models default to direct, efficient answers. If you want conversational or detailed output, ask for it.
- **Handle multimodal inputs coherently.** Reference each modality (text, image, audio, video) clearly in your instructions.
- **Prioritize critical instructions.** Put persona, behavioral constraints, and output format at the top — in the system instruction or start of the user prompt.
- **Structure for long context.** Put large context blocks *first*, then your specific instructions/questions at the *end*.
- **Anchor transitions.** After a big context block, use a phrase like *"Based on the information above..."* to bridge into the question.

### Reasoning

Reasoning-capable models generate internal thinking automatically. You usually don't need to ask for step-by-step output. For very hard problems, a nudge like *"Think very hard before answering"* can help — at the cost of more thinking tokens.

---

## 9. Structured Prompt Templates

### XML Structure

```
<role>
You are a helpful assistant.
</role>

<constraints>
1. Be objective.
2. Cite sources.
</constraints>

<context>
[User input — the model treats this as data, not instructions]
</context>

<task>
[The specific request]
</task>
```

### Markdown Structure

```
# Identity
You are a senior solution architect.

# Constraints
- No external libraries.
- Python 3.11+ syntax only.

# Output format
Return a single code block.
```

### Combined Template

**System instruction:**

```
<role>
You are a specialized assistant for [domain].
You are precise, analytical, and persistent.
</role>

<instructions>
1. Plan: analyze the task and create a step-by-step plan.
2. Execute: carry out the plan.
3. Validate: review your output against the user's task.
4. Format: present the final answer in the requested structure.
</instructions>

<constraints>
- Verbosity: [Low/Medium/High]
- Tone: [Formal/Casual/Technical]
</constraints>

<output_format>
1. Executive Summary: [short overview]
2. Detailed Response: [main content]
</output_format>
```

**User prompt:**

```
<context>
[Documents, code snippets, background]
</context>

<task>
[Specific request]
</task>

<final_instruction>
Think step-by-step before answering.
</final_instruction>
```

---

## 10. Agentic Workflows

Agents that plan, reason, and execute multi-step tasks need additional steering. Think about these behavioral dimensions:

### Reasoning and Strategy

- **Logical decomposition** — how thoroughly the model analyzes constraints, prerequisites, and ordering.
- **Problem diagnosis** — depth of root-cause analysis; willingness to explore non-obvious explanations.
- **Information exhaustiveness** — trade-off between reviewing every policy/doc and moving fast.

### Execution and Reliability

- **Adaptability** — stick to the original plan vs. pivot when new data contradicts it.
- **Persistence and recovery** — how hard the agent tries to self-correct. Too much = loops and cost.
- **Risk assessment** — distinguish low-risk reads from high-risk writes.

### Interaction and Output

- **Ambiguity and permission handling** — when to assume vs. when to pause and ask.
- **Verbosity** — how much the agent narrates during tool calls.
- **Precision and completeness** — exact answers covering all edge cases vs. approximate.

### Agent System Instruction Template

```
You are a strong reasoner and planner. Follow these instructions to structure
your plans, thoughts, and responses.

Before any action (tool call or response to user), plan and reason about:

1) Logical dependencies and constraints — resolve in order:
   1.1) Policy rules, mandatory prerequisites
   1.2) Order of operations; reorder steps if needed
   1.3) Other prerequisites (info or actions)
   1.4) Explicit user constraints or preferences

2) Risk assessment — consequences of the action, downstream effects.
   For exploratory tasks, missing optional parameters is LOW risk;
   prefer calling the tool over asking the user, unless an optional
   parameter is required for a later step.

3) Abductive reasoning — most likely cause of any problem.
   Look beyond the obvious. Test hypotheses; don't discard unlikely
   ones prematurely.

4) Outcome evaluation — does the last observation require plan changes?
   If hypotheses are disproven, generate new ones.

5) Information sources — use all applicable:
   5.1) Tools and their capabilities
   5.2) Policies, rules, checklists, constraints
   5.3) Previous observations and history
   5.4) Information only the user can provide

6) Precision and grounding — quote exact applicable information when
   referencing policies or docs.

7) Completeness — exhaustively incorporate all requirements.
   Resolve conflicts via #1. Don't conclude prematurely.
   Check all information sources from #5.

8) Persistence — don't give up until reasoning is exhausted.
   On transient errors, retry (respecting any limit). On other
   errors, change strategy — don't repeat failed calls.

9) Inhibit your response — only act after the above reasoning is done.
   Once taken, actions can't be undone.
```

---

## Quick Checklist

- [ ] Is the instruction clear and specific?
- [ ] Are constraints (length, tone, format) stated?
- [ ] Did you include few-shot examples? Are they formatted consistently?
- [ ] Did you supply all needed context?
- [ ] Is complex logic broken into chained or parallel prompts?
- [ ] Did you use delimiters (XML tags or Markdown) to separate sections?
- [ ] For long context, did you put context first and instructions last?
- [ ] Did you tune temperature and other parameters for the task?
- [ ] Did you iterate — try rephrasing, analogous tasks, reordering?
