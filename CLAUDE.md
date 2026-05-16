<policies>

<root_cause>
- Find root cause. Patch root cause.
- Surface errors loudly; fail fast on broken state.
- Symptom in layer A but cause in layer B → fix layer B.
- Recurring class of bugs → fix shape of code, not each instance.
</root_cause>

<verification>
- Treat "I think this field is always set" / "this should work" as hypothesis, not fact.
- Confirm with data (query DB, read logs, run code) before claims touching real systems.
- Disconnect prod port-forwards / live connections immediately after query finishes.
</verification>

<research>
Before writing or modifying code that uses any library, framework, or external API, use the `library-researcher` agent to verify current usage patterns. Do this every time — not just for new dependencies. APIs change between versions, methods get deprecated, and defaults shift.

For each library or API you plan to use, provide:
- The library/API **name**
- **Why** you need it
- The specific **methods or features** you intend to use

Use the researcher's output as your source of truth rather than relying on training data, which may reflect outdated versions. This applies equally to existing code you're modifying — verify that current usage is still correct.

Always complete this step before writing implementation code.
</research>

<parallelism>
- Spend the compute. Bias toward thorough investigation over fast guess.
- Spawn multiple agents concurrently for independent investigations (Explore, general-purpose, library-researcher).
- Single message, multiple Agent tool calls. Run independent work in parallel.
</parallelism>

<backward_compat>
Default to cleanest solution. Refactor wins over `@Deprecated` graveyard. Flag breaking changes and ask whether compat is needed — let user decide, never silently preserve old APIs/fields/interfaces.
</backward_compat>

<comments>
**DO NOT WRITE NEW COMMENTS.** Default = zero comments. Only exception: non-obvious WHY (hidden constraint, subtle invariant, workaround for a specific known bug). If you cannot state the WHY in one short line, the comment is wrong — do not write it.

In any file you touch, **DELETE** comments that look like these:
- `// Fetch user from database` — restates the code. DELETE.
- `// Added for TICKET-1234` / `// Fix from PR #87` — task/PR reference. DELETE.
- `// Now uses the new auth flow` / `// Updated to handle X` — changelog in code. DELETE.
- `// This function validates the input` — docstring-flavored restatement. DELETE or trim to ≤6 words.
- `// TODO: refactor later` — stale aspirational TODO with no owner. DELETE or file a ticket.
- `// Removed unused helper` / `// was previously doing Y` — references deleted code. DELETE.

KEEP only short, single-line WHY notes. Examples:
- `// workaround for TICKET-2401 index bug`
- `// must run before X — Y holds lock`
- `// off-by-one intentional: upstream inclusive`

Rule: if comment spans >1 line OR >~10 words, it is too long. Trim or delete. If WHY needs a paragraph, it belongs in PR/ticket, not code.

EVERY new comment is a failure to make code speak for itself, **UNLESS** it explains something the code CAN'T say. If in doubt, do not write it.
</comments>

</policies>
