# Prompt Rewriting Reference for Claude Opus 4.7

When this file is attached alongside a user prompt, rewrite that prompt using the techniques below.

**Delivery.** Write the full rewritten prompt to a new file (e.g., `prompts/<task-slug>.md`) with `Write`, then reply with only the file path and a one-line summary. The rewrite is meant to be attached to a fresh session that will execute the task, so it must stand alone — no references to the current conversation, no "as we discussed" shorthand, nothing that depends on context the new session won't have. Do not paste the rewritten prompt inline in chat; pasting makes it awkward to attach and tempts truncation. If the original is already strong, skip the file write, say so, and suggest at most one or two targeted tweaks.

**Preserve the user's intent exactly.** Improve clarity, structure, and steerability. Never change scope.

---

## Mental model

Treat Opus 4.7 like a brilliant, literal-minded new hire with deep domain knowledge and zero context on your norms. It follows instructions more precisely than prior models, which cuts both ways:

- Vagueness costs you quality — the model won't infer unstated preferences.
- Overspecified prohibitions and aggressive phrasing cause overtriggering and defensive padding.
- Unstated scope gets interpreted narrowly — instructions don't silently generalize.

**Litmus test:** Hand the prompt to a colleague with no context. If they'd hesitate, guess, or ask a clarifying question, the prompt is under-specified.

---

## 1. Be specific and direct

State the desired output, format, constraints, and thoroughness bar explicitly. Opus 4.7 calibrates response length and effort to the apparent complexity of the ask — if you want depth, say so.

| Weak | Strong |
|------|--------|
| `Create an analytics dashboard` | `Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.` |
| `Improve this function` | `Refactor this function to reduce cyclomatic complexity below 10. Preserve the public API and all existing behavior.` |
| `Summarize the meeting` | `Summarize the meeting in 5 bullets or fewer. For each, name the decision, the owner, and the deadline.` |

- Use numbered steps when order matters; bullets when completeness matters.
- State the scope of each instruction. "Apply this formatting to every section, not just the first" beats "apply this formatting."
- If you want "above and beyond" behavior, ask for it in those words.

---

## 2. Give the reason, not just the rule

Explaining *why* lets Claude generalize to edge cases you didn't enumerate. A rule without a reason is a rule without a model.

| Weak | Strong |
|------|--------|
| `Never use ellipses` | `The response will be read aloud by a TTS engine that can't pronounce ellipses — use full sentences instead.` |
| `Keep responses short` | `This displays in a mobile notification capped at 100 characters, so respond in one tight sentence.` |
| `Use simple language` | `The audience is non-technical PMs. Explain concepts with everyday analogies and avoid jargon. If a term is unavoidable, define it in parentheses on first use.` |

---

## 3. Prefer directives over prohibitions

"Don't X" leaves ambiguity about what *should* happen. "Do Y" fills the space and is more reliable.

| Prohibition | Directive |
|-------------|-----------|
| `Don't use markdown` | `Write in flowing prose paragraphs. Reserve markdown for inline code and code blocks only.` |
| `Don't be verbose` | `Respond in 2–3 sentences.` |
| `Don't assume` | `State each assumption explicitly before acting on it.` |
| `Don't use LaTeX` | `Format math in plain text: / for division, * for multiplication, ^ for exponents.` |
| `Don't make up data` | `If a value isn't in the source material, write "UNKNOWN" and move on.` |

---

## 4. Structure mixed content with XML tags

When a prompt combines instructions, context, examples, and variable inputs, wrap each in descriptive tags so the model parses them unambiguously.

```xml
<role>Distinguished backend engineer reviewing Python distributed-systems code.</role>

<instructions>
Review the code for correctness, performance, and readability. Flag any security concerns.
</instructions>

<code>
{{USER_CODE}}
</code>

<output_format>
For each issue: file, line number, severity (low/medium/high), one-line fix suggestion.
</output_format>
```

- Use consistent tag names across prompts you reuse.
- Nest hierarchically when content has natural structure: `<documents>` > `<document index="n">` > `<source>` + `<document_content>`.
- Tags double as output-format indicators. "Put the analysis in `<analysis>` tags" constrains the shape naturally.

---

## 5. Assign a specific role, scoped to the task

One sentence focuses tone, vocabulary, and depth of expertise. Specificity beats generality.

Claude Code's system prompt is already loaded by the time these prompts run (tools, skills, memory, CLAUDE.md). Don't fight it — scope the role to the current task at the top of the user message, so it overrides tone and perspective for this turn only.

- `For this next task, act as a staff backend engineer specializing in Python distributed systems, with deep experience in asyncio and database tuning.`
- `For the following request, respond as a principal copy editor for a technical publication read by software architects. Favor precise, concrete language over abstraction.`
- `For this task, adopt the perspective of a distinguished security reviewer. Assume every input is hostile until proven otherwise.`

Weak: `You are a helpful assistant.` — no specificity, no steering effect.

**An XML `<role>` tag alone does not do this work.** Tags label content; they don't override the loaded persona. The imperative scoping verb ("For this next task, act as…") is what switches perspective. Combine them if you like — `<role>For this next task, act as…</role>` — but never rely on the tag without the scoping phrase inside it.

---

## 6. Use few-shot examples for format, tone, and edge cases

Three to five examples are one of the most reliable ways to steer output. Wrap each in `<example>` tags inside an `<examples>` block so they're distinguishable from instructions.

```xml
<examples>
  <example>
    <input>The server crashed when processing large files</input>
    <output>BUG-2341: File processor OOMs on inputs >2GB — switch to a streaming parser.</output>
  </example>
  <example>
    <input>Users want dark mode</input>
    <output>FEAT-0892: Dark mode toggle — implement CSS variable-based theming.</output>
  </example>
  <example>
    <input>Login works but logout doesn't clear the session cookie</input>
    <output>BUG-2415: Logout fails to invalidate session — clear cookie and revoke server-side token.</output>
  </example>
</examples>
```

Good examples are:
- **Relevant** — mirror the actual task, not a toy version of it.
- **Diverse** — cover edge cases so the model doesn't overfit to one shape.
- **Format-accurate** — the output format will match what the examples show, down to punctuation.

---

## 7. Control output format explicitly

Pick the mechanism that matches the constraint:

- **Structured data:** Prefer structured outputs or tool definitions with JSON schemas over asking for JSON in prose. Opus 4.7 follows schemas reliably when told to.
- **No preamble:** `Respond directly. Do not open with "Here is…", "Based on…", or similar framing.`
- **Minimal markdown:** `Write in flowing prose. Reserve markdown for inline code and code blocks only.`
- **Plain-text math:** `Format math in plain text — no LaTeX, no MathJax, no \( \) or $...$. Use /, *, ^.`
- **Match the prompt style to the desired output.** A prose prompt encourages prose; a bulleted prompt encourages bullets. If outputs are too markdown-heavy, strip markdown from the prompt itself.

---

## 8. Calibrate for Opus 4.7

Prompts tuned for older Claude models often need these adjustments. Each item below describes a behavior shift in 4.7 and how to prompt around it.

### Drop aggressive language

`CRITICAL: You MUST use this tool when…` → `Use this tool when…`. Opus 4.7 is highly responsive to instructions; shouting causes overtriggering — spurious tool calls, defensive padding, overqualified answers. Normal phrasing works.

### Remove anti-laziness prompting

Instructions like "Be thorough," "Don't be lazy," "If in doubt, use [tool]" — which older models needed — now cause excessive work or overtriggering. Delete them. If you still want depth, describe the standard concretely ("include at least one edge case per function") rather than cheerleading.

### State scope explicitly

Opus 4.7 interprets prompts literally and does not silently generalize. If an instruction should apply broadly, say so: `Apply this to every section, not just the first.` If a rule has exceptions, enumerate them. If a list should be exhaustive, write "list all" instead of trusting inference.

### Action vs. suggestion

Opus 4.7 acts rather than suggests unless told otherwise. Choose deliberately:

- Want action: `Implement these changes.` / `Make these edits.`
- Want suggestions only: `Suggest changes but do not modify any files or run commands.`

Avoid hedged phrasing like "Can you suggest…?" when you actually want action — the model may take you at your word and stop at suggestions.

### Thinking and commitment

Opus 4.7 uses adaptive thinking: it decides when and how deeply to reason. If you observe over-deliberation, add:

```text
Choose an approach and commit to it. Do not revisit a decision unless you encounter information that directly contradicts it.
```

If you observe under-thinking on complex problems, raise the `effort` API parameter (`high` or `xhigh` for coding and agentic work) rather than prompting around it.

### Concision is the default

Opus 4.7 calibrates length to complexity and may skip summaries after tool use. If you want visibility, request it: `After each tool-using step, provide a one-sentence summary of what changed.` If you want brevity on a complex task, state the ceiling: `Respond in under 150 words.`

### Tone

Opus 4.7 is more direct and opinionated than 4.6, with less validation-forward phrasing and fewer emoji. If you need a warmer voice, prompt for it: `Use a warm, collaborative tone. Acknowledge the user's framing before answering.`

### Prefill is deprecated

Prefilled responses on the last assistant turn are no longer supported on current models. Replace prefill tactics with:
- **Format control** → structured outputs or XML-tagged output.
- **Eliminate preamble** → `Respond directly without preamble.`
- **Continuations** → move the partial text into the user turn: `Your previous response was interrupted ending with "[text]". Continue from there.`

---

## 9. Long-context prompts (20k+ tokens)

- **Put long documents at the top**, above instructions and the final query. Query-at-the-end improves response quality by up to 30% on complex multi-document inputs.
- **Tag documents with metadata:**
  ```xml
  <documents>
    <document index="1">
      <source>annual_report_2023.pdf</source>
      <document_content>{{ANNUAL_REPORT}}</document_content>
    </document>
  </documents>
  ```
- **Ground answers in quotes.** Ask Claude to extract relevant quotes into `<quotes>` tags before analyzing. It cuts through noise and reduces hallucination on long inputs.

---

## 10. Agentic and multi-step tasks

- **Front-load intent.** Opus 4.7 is more autonomous than prior models. State the task, success criteria, and relevant constraints in the first user turn — progressive clarification across multiple turns costs tokens and performance.
- **Chain prompts explicitly only when you need to inspect intermediate state** (e.g. draft → review → refine with logging between steps). Otherwise let the model plan internally.
- **Call out reversibility** when the task touches shared systems: `Ask before running destructive or hard-to-reverse actions — deletes, force-push, external posts.`
- **Ground investigation in source.** For codebase questions: `Read the referenced files before answering. Do not speculate about code you have not opened.`
- **When dispatching parallel subagents, spec each brief in full.** Overall-prompt concision is not a license for vague per-agent instructions. Parallel agents don't see each other's work, so each brief must specify its own slice of the surface area — exact files, modules, or symbols to audit — tightly enough that the briefs tile the problem without gaps or overlap. For a multi-repo audit, list the specific entities/endpoints/modules each agent owns. A short overall prompt with three thin subagent briefs will miss coverage; a longer prompt with three specific briefs won't.

---

## When rewriting a prompt

1. Apply only the techniques that improve the specific prompt — don't force all ten.
2. Preserve the user's intent exactly. Improve clarity and structure; never change scope.
3. Keep it as short as possible while unambiguous. Padding degrades performance.
4. For mixed instructions / context / inputs, reach for XML tags.
5. When output shape matters most, reach for examples.
6. If the original is already strong, say so and suggest one or two targeted tweaks rather than rewriting.

Return the rewritten prompt directly. No "Here's the rewrite," no trailing explanation.
