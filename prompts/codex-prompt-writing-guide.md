# Codex Prompt Writing Guide

A good Codex prompt is a compact work order. It gives the agent enough context to act, defines success, names constraints, and tells it how to verify the result.

Use prompts for task-specific direction. Put durable rules, repo conventions, and repeatable workflows in `AGENTS.md` or a skill so every prompt does not have to carry them.

## Strong Prompt Shape

Most useful prompts answer five questions:

1. What should Codex accomplish?
2. Where should Codex look?
3. What constraints must it preserve?
4. How should Codex verify the work?
5. What should Codex report back when done?

Keep the prompt outcome-first. Describe the desired end state and evidence of success. Avoid scripting every internal step unless the exact process matters.

## Goal

Start with the concrete result you want:

```text
Fix the settings save bug so notification preferences persist after refresh.
```

For larger work, define the deliverable:

```text
Add a `--json` flag to the CLI that prints the existing summary as JSON.
```

Avoid vague prompts such as "look at settings and make it better" unless you only want exploration.

## Context

Give Codex the best available starting points:

- Relevant files, folders, routes, functions, tickets, screenshots, logs, or stack traces.
- Reproduction steps for bugs.
- Existing behavior that must remain unchanged.
- User-facing behavior that should change.
- Prior decisions that would not be obvious from the code.

Example:

```text
Bug: clicking Save on `/settings/notifications` shows "Saved" but the toggle resets after refresh.

Repro:
1. Run `npm run dev`.
2. Open `/settings/notifications`.
3. Toggle "Email alerts".
4. Click Save.
5. Refresh the page.

Start with `app/settings/notifications` and the preferences API client.
```

## Constraints

Constraints prevent well-intended changes from drifting. Use them for real boundaries, not preferences Codex can infer from the codebase.

```text
Constraints:
- Keep the API response shape unchanged.
- Do not add a new dependency.
- Match the existing test style.
- Keep the patch focused on this bug.
```

## Verification

Codex does better when it can prove the work. Tell it what checks matter:

```text
Verification:
- Add or update the smallest relevant regression test.
- Run the affected test file.
- Run lint if this package has a local lint command.
- Report any command that cannot be run.
```

If manual verification matters, spell it out:

```text
After the fix, tell me the browser route and exact steps to confirm the behavior.
```

## Autonomy

For ordinary implementation, let Codex proceed:

```text
Investigate, implement the fix, and verify it. Ask only if a product or compatibility decision is required.
```

For risky work, narrow the first step:

```text
First inspect the call sites and propose an implementation plan. Do not edit files yet.
```

For reviews, make the stance explicit:

```text
Review this change for correctness, edge cases, and missing tests. Lead with findings and cite file/line references.
```

## Threads

Use one thread per task. Long, mixed-purpose threads create bloated context and weaker results.

Start a new thread when the task changes materially. Continue the same thread when you want Codex to build on the current investigation, plan, or patch.

Avoid running multiple live threads against the same files unless they are isolated with worktrees or clearly non-overlapping.

## Leave Out

Avoid stuffing prompts with:

- Large pasted docs when a link or file reference is enough.
- Tool schemas or implementation details Codex can inspect.
- Durable team rules that belong in `AGENTS.md`.
- Model release notes, pricing, or API configuration unless the task is about those things.
- The current date unless the task depends on a specific timezone or business date.
- Repeated instructions that say the same thing in different words.

## Templates

### Bug Fix

```text
Fix [bug].

Repro:
1. [step]
2. [step]
3. [failing result]

Expected: [desired result]
Context: start with [file/module] and preserve [important behavior/API].
Verification: add/update a regression test if feasible, run [command], and report blockers.
```

### Feature

```text
Add [feature] so users can [outcome].

Scope: touch [area/files] if possible, reuse existing patterns, and do not change [boundary].
Acceptance criteria: [observable behavior], [edge case], [test or validation].

Implement and verify in this thread.
```

### Documentation

```text
Rewrite [doc/path] for [audience].

Goals: make it concise and current, remove [outdated topic], and keep it under [limit].
Use [source docs/links] as input. Edit the file directly and verify the final structure.
```

### Planning Only

```text
Plan [change] without editing files.

Include assumptions, risks, proposed milestones, verification strategy, and open questions.
```

## Final Check

Before sending a prompt, check that it has a clear outcome, useful context, real constraints, verification expectations, and a stopping rule or final-report expectation.

Short, specific prompts beat long, generic ones. Give Codex the destination, the guardrails, and the evidence you need; let it choose the path unless the path is part of the requirement.
