<policies>

<root_cause>
Find root cause, patch root cause. Surface errors loudly; fail fast on broken state. If symptom is in layer A but cause is in layer B, fix layer B. If a class of bugs recurs, fix the shape of the code instead of patching each instance.
</root_cause>

<verification>
Treat "I think this field is always set" or "this should work" as a hypothesis, not a fact. Confirm with data (query the DB, read the logs, run the code) before making claims that touch real systems. Disconnect any prod port-forward or live connection as soon as the query finishes.
</verification>

<research>
Before using a library, framework, or external API you have not verified in this session, spawn the `library-researcher` agent. Trigger applies to: new dependencies, version bumps, APIs you have not touched recently, or any call where the signature is not visible in the current file. Skip for libraries you have already verified this session or whose usage is obvious from surrounding code.

When spawning, provide:
- Library or API name
- Why you need it
- Specific methods or features you plan to use

Use the researcher's output as source of truth over training data.
</research>

<backward_compat>
Default to the cleanest solution; refactor over `@Deprecated` graveyards. Flag breaking changes and ask whether compat is required — let the user decide. Never silently preserve old APIs, fields, or interfaces.
</backward_compat>

<comments>
Write WHY-only comments. Default is zero comments. A comment earns its place only when it captures something the code cannot say: a hidden constraint, a subtle invariant, a workaround for a known bug. If the WHY does not fit in one short line, the comment is wrong.

In files you touch, remove comments that restate the code, reference tickets or PRs, log a changelog ("now uses X"), or describe deleted code.

Good:
- `// workaround for TICKET-2401 index bug`
- `// must run before X — Y holds the lock`
- `// off-by-one intentional: upstream is inclusive`
</comments>

</policies>
