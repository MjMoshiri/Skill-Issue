<guide name="prompting-guide" target="Claude Opus 4.8">

When this file is attached alongside a user prompt, rewrite that prompt using the techniques below.

<delivery>
Write the full rewritten prompt to a new file (e.g., `prompts/<task-slug>.md`) with `Write`, then reply with only the file path and a one-line summary. The rewrite is meant to be attached to a fresh session that will execute the task, so it must stand alone — no references to the current conversation, no "as we discussed" shorthand, nothing that depends on context the new session will not have.

Do not paste the rewritten prompt inline in chat; pasting makes it awkward to attach and tempts truncation. If the original is already strong, skip the file write, say so, and suggest at most one or two targeted tweaks.

Preserve the user's intent exactly. Improve clarity, structure, and steerability. Do not narrow, expand, or invent scope the user left open.

**When the deliverable is underspecified, do not manufacture one.** A thin deliverable is the common case — the user usually knows the task but not yet the exact shape of the output. Your job is to maximize the rules and context, not to author a deliverable the user never specified. When it's missing, pick one:

- **Ask first.** Before writing the file, ask a single batched set of tight questions covering only what actually changes the output. This is the one time it's right to reply with questions instead of a file.
- **Clarify-gate.** If the prompt must stand alone, bake the gate into the rewrite: instruct the executing agent to confirm or *propose* the deliverable (e.g. "propose 2–3 options and let the user pick one before building") and to surface its assumptions before producing — rather than charging ahead on a guess.
</delivery>

<mental_model>
Treat Opus 4.8 like a brilliant, literal-minded expert with deep domain knowledge and zero context on your norms. It follows instructions precisely and is highly autonomous, which cuts both ways:

- Vague *rules and context* cost you quality — the model will not infer unstated preferences or constraints.
- Overspecified prohibitions and aggressive phrasing cause overtriggering and defensive padding.
- Unstated scope gets interpreted narrowly — instructions do not silently generalize.
- A capable model trusted with rich context and clear constraints will pick a better execution path than one boxed in by prescriptive steps. Give it the *what* and the *why*; let it own the *how*.

**Two layers, two rules.** Separate *context and rules* — constraints, domain facts, success criteria, gotchas, file pointers, reversibility — from *the deliverable and its scope* — what to build, its shape, its acceptance bar:

- **Context and rules — enrich freely.** Richer is better; this is where the rewrite earns its keep.
- **Deliverable and scope — never invent.** If the user didn't specify the deliverable, that is not a hole to fill with a plausible guess. It is a question to raise.

**Self-check:** if a fresh session would have to guess at the *rules, context, or constraints*, enrich the prompt. If it would have to guess at the *deliverable or scope*, do not guess for it — ask the user, or build a clarify-gate into the rewrite (see <delivery>).
</mental_model>

<section name="be_specific_and_direct">
State the desired output, format, constraints, and thoroughness bar explicitly. Opus 4.8 calibrates response length and effort to the apparent complexity of the ask — if you want depth, say so.

| Weak | Strong |
|------|--------|
| `Create an analytics dashboard` | `Create an analytics dashboard. Include as many relevant features and interactions as possible. Go beyond the basics to create a fully-featured implementation.` |
| `Improve this function` | `Refactor this function to reduce cyclomatic complexity below 10. Preserve the public API and all existing behavior.` |
| `Summarize the meeting` | `Summarize the meeting in 5 bullets or fewer. For each, name the decision, the owner, and the deadline.` |

Use numbered steps when order matters; bullets when completeness matters. State the scope of each instruction. If you want "above and beyond" behavior, ask for it in those words.
</section>

<section name="give_the_reason">
Explaining *why* lets Claude generalize to edge cases you did not enumerate. A rule without a reason is a rule without a model.

| Weak | Strong |
|------|--------|
| `Never use ellipses` | `The response will be read aloud by a TTS engine that can't pronounce ellipses — use full sentences instead.` |
| `Keep responses short` | `This displays in a mobile notification capped at 100 characters, so respond in one tight sentence.` |
| `Use simple language` | `The audience is non-technical PMs. Explain concepts with everyday analogies and avoid jargon. If a term is unavoidable, define it in parentheses on first use.` |
</section>

<section name="prefer_directives_over_prohibitions">
"Don't X" leaves ambiguity about what *should* happen. "Do Y" fills the space and is more reliable.

| Prohibition | Directive |
|-------------|-----------|
| `Don't use markdown` | `Write in flowing prose paragraphs. Reserve markdown for inline code and code blocks only.` |
| `Don't be verbose` | `Respond in 2–3 sentences.` |
| `Don't assume` | `State each assumption explicitly before acting on it.` |
| `Don't use LaTeX` | `Format math in plain text: / for division, * for multiplication, ^ for exponents.` |
| `Don't make up data` | `If a value isn't in the source material, write "UNKNOWN" and move on.` |
</section>

<section name="structure_with_xml">
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

Use consistent tag names across prompts you reuse. Nest hierarchically when content has natural structure: `<documents>` > `<document index="n">` > `<source>` + `<document_content>`. Tags double as output-format indicators. "Put the analysis in `<analysis>` tags" constrains the shape naturally.
</section>

<section name="assign_a_role">
One sentence focuses tone, vocabulary, and depth of expertise. Specificity beats generality.

Claude Code's system prompt is already loaded by the time these prompts run (tools, skills, memory, CLAUDE.md). Do not fight it — scope the role to the current task at the top of the user message, so it overrides tone and perspective for this turn only.

- `For this next task, act as a staff backend engineer specializing in Python distributed systems, with deep experience in asyncio and database tuning.`
- `For the following request, respond as a principal copy editor for a technical publication read by software architects. Favor precise, concrete language over abstraction.`
- `For this task, adopt the perspective of a distinguished security reviewer. Assume every input is hostile until proven otherwise.`

Weak: `You are a helpful assistant.` — no specificity, no steering effect.

**An XML `<role>` tag alone does not do this work.** Tags label content; they do not override the loaded persona. The imperative scoping verb ("For this next task, act as…") is what switches perspective. Combine them if you like — `<role>For this next task, act as…</role>` — but never rely on the tag without the scoping phrase inside it.
</section>

<section name="few_shot_examples">
Three to five examples are one of the most reliable ways to steer output. Wrap each in `<example>` tags inside an `<examples>` block so they are distinguishable from instructions.

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

Good examples are **relevant** (mirror the actual task), **diverse** (cover edge cases), and **format-accurate** (the output format will match what the examples show, down to punctuation).
</section>

<section name="control_output_format">
Pick the mechanism that matches the constraint:

- **Structured data:** spell out the exact schema in the prompt and show one filled example. Opus 4.8 matches a clearly described shape reliably when told to.
- **No preamble:** `Respond directly. Do not open with "Here is…", "Based on…", or similar framing.`
- **Minimal markdown:** `Write in flowing prose. Reserve markdown for inline code and code blocks only.`
- **Plain-text math:** `Format math in plain text — no LaTeX, no MathJax, no \( \) or $...$. Use /, *, ^.`
- **Match the prompt style to the desired output.** A prose prompt encourages prose; a bulleted prompt encourages bullets. If outputs are too markdown-heavy, strip markdown from the prompt itself.
</section>

<section name="calibrate_for_4_8">
Prompts tuned for older Claude models often need these adjustments.

**Drop aggressive language.** `CRITICAL: You MUST use this tool when…` → `Use this tool when…`. Opus 4.8 is highly responsive; shouting causes overtriggering — spurious tool calls, defensive padding, overqualified answers.

**Remove anti-laziness prompting.** "Be thorough," "Don't be lazy," "If in doubt, use [tool]" now cause excessive work or overtriggering. Delete them. If you still want depth, describe the standard concretely ("include at least one edge case per function") rather than cheerleading.

**State scope explicitly.** Opus 4.8 interprets prompts literally and does not silently generalize. If an instruction should apply broadly, say so: `Apply this to every section, not just the first.` If a rule has exceptions, enumerate them. If a list should be exhaustive, write "list all" instead of trusting inference.

**Action vs. suggestion.** Opus 4.8 acts rather than suggests unless told otherwise. Choose deliberately:
- Want action: `Implement these changes.` / `Make these edits.`
- Want suggestions only: `Suggest changes but do not modify any files or run commands.`

Avoid hedged phrasing like "Can you suggest…?" when you actually want action — the model may take you at your word and stop at suggestions.

**Ask for depth when the task needs it.** Opus 4.8 calibrates effort to the apparent complexity of the ask. If a task genuinely needs multi-step reasoning, say so rather than assuming it's inferred:

```text
This task involves multi-step reasoning. Think carefully through the problem before responding.
```

**Thinking and commitment.** If you observe over-deliberation, add:

```text
Choose an approach and commit to it. Do not revisit a decision unless you encounter information that directly contradicts it.
```

**Concision is the default.** Opus 4.8 calibrates length to complexity and self-narrates long agentic traces well — drop any "summarize after every N tool calls" scaffolding from older prompts. If you want a specific cadence or ceiling, describe it concretely and prefer a positive example of the right length over "don't be verbose": `After each tool-using step, give a one-sentence summary of what changed.` / `Respond in under 150 words.`

**Tone.** Opus 4.8 is direct and opinionated, with little validation-forward phrasing and sparing emoji. If you need a warmer voice, prompt for it: `Use a warm, collaborative tone. Acknowledge the user's framing before answering.`
</section>

<section name="long_context">
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
</section>

<section name="agentic_and_multi_step">
- **Front-load intent.** Opus 4.8 is highly autonomous. State the task, success criteria, and relevant constraints in the first user turn — progressive clarification across multiple turns costs tokens and performance. (This is the *rules and context* layer; if the deliverable itself is unclear, raise it rather than padding the prompt with guesses — see <delivery>.)
- **Chain prompts explicitly only when you need to inspect intermediate state** (e.g. draft → review → refine with logging between steps). Otherwise let the model plan internally.
- **Call out reversibility** when the task touches shared systems: `Ask before running destructive or hard-to-reverse actions — deletes, force-push, external posts.`
- **Ground investigation in source.** For codebase questions: `Read the referenced files before answering. Do not speculate about code you have not opened.`
- **Name the tools you want used.** Opus 4.8 favors reasoning over tool calls, which is usually right but can leave a needed tool untouched. If a task depends on a specific tool (web search, a file read, a skill), say when and how to use it: `Use web search to confirm current API signatures before writing code.`
- **Subagents: say when you want fan-out.** Opus 4.8 spawns fewer subagents by default. If the task benefits from parallelism, ask for it explicitly: `Spawn a subagent per file when fanning out across many files; work directly for single-file edits.`
- **When dispatching parallel subagents, spec each brief in full.** Parallel agents do not see each other's work, so each brief must specify its own slice of the surface area — exact files, modules, or symbols — tightly enough that the briefs tile the problem without gaps or overlap.
</section>

<when_rewriting>
1. Apply only the techniques that improve the specific prompt — do not force every technique.
2. Preserve the user's intent exactly. Improve clarity and structure; do not narrow, expand, or invent scope.
3. Enrich rules and context freely; never fabricate a deliverable the user did not specify — ask or clarify-gate instead (see <delivery>).
4. Keep it as short as possible while unambiguous. Padding degrades performance.
5. For mixed instructions / context / inputs, reach for XML tags.
6. When output shape matters most, reach for examples.
7. If the original is already strong, say so and suggest one or two targeted tweaks rather than rewriting.

Return the rewritten prompt directly — no "Here's the rewrite," no trailing explanation. The one exception: when the deliverable itself is unclear, reply with your batched clarifying questions instead of writing the file.
</when_rewriting>

</guide>
