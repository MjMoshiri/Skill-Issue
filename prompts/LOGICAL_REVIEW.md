# Code Review Guide

The code you are about to review has been written primarily by LLM agents. It will compile. It will pass linting. It will often pass basic tests. It will read cleanly. None of that means it is correct.

LLM-generated code is fluent — it *looks* like it was written by someone who knows what they're doing. The patterns are familiar, the naming is reasonable, the structure follows conventions. This is what makes it dangerous. The most critical flaws are in what the code *does* — or doesn't do, or does unnecessarily. **Logical correctness is the primary concern of this review**, but anything that would cause problems in production is fair game — performance, security, error handling, placement, naming that misleads, tests that don't actually test anything.

The goal is to catch real problems. Logic comes first. Everything else is evaluated in service of that.

---

## Step 1: Understand the Intent

The foundation of a good review is understanding what the change is *trying to do* before evaluating whether it does it correctly. Without this anchor, every line of code looks reasonable in isolation and there is no basis for judging whether something is missing, excess, or misplaced.

PR descriptions and ticket links are often missing, vague, or unhelpful. That makes this step harder, but more important — not less.

Read the diff and the surrounding context and produce a short, abstract summary — no code evaluation, no correctness judgments, no line-level commentary. Just an answer to:

- What is this change trying to do? (Add a feature? Fix a bug? Refactor? Integrate with something?)
- What is the high-level approach? (New service introduced? Existing flow modified? Third-party integration added?)
- What is the scope? (Narrow and surgical, or broad and sweeping?)

The output of this step is a short plain-language summary of the intent. This becomes the anchor for everything that follows.

---

## Step 2: Review the Code

With the intent established, the actual review begins. Every piece of code in the diff is now measured against that intent. Logic is the primary lens, but the review is not artificially constrained — anything that would cause a real problem gets flagged.

### Tracing data, not control flow

The natural instinct is to read code top-to-bottom, function-by-function. A more revealing approach is to pick up a piece of data at its origin — a user input, a database row, an API response — and follow it through the system. Where is it created? Where is it transformed? Where is it stored? Where does it exit?

This surfaces things that reading control flow misses: a value fetched, transformed, stored, and then fetched again from a less reliable source. Data that passes through three layers of mapping when one transformation would do. An aggregation computed in application code from rows the database could have aggregated directly. A field that is set but never read, or read but set to a stale value.

Tracing data reveals the *actual work* the code does, stripped of structural ceremony — and where it does unnecessary work.

### Identifying shadow state

Every time the code maintains state — a counter, a flag, a collected set, a cached value — the question is: *"Is this the authoritative source for this information, or is it a shadow of something that already exists elsewhere?"*

LLM agents love shadow state. They increment a counter in a loop and return it, when the result list's `.size()` gives the same answer. They build a `Set<String>` of processed IDs alongside database inserts, then use the Set for the final result instead of querying the database. They track a `boolean hasErrors` flag when the exception itself, or an empty error list, already answers the question.

The danger is not that shadow state is wrong today — it is that it *can drift*. The source of truth — the database, the collection, the framework's own state — cannot drift, because it *is* the reality. Shadow state is a copy of reality, and copies go stale.

The test: can a variable be deleted and the same answer obtained from somewhere more authoritative? If yes, it is shadow state.

### Thinking at scale

Before evaluating whether a loop body is correct, ask: *"What if this collection has 10,000 elements?"*

LLM-generated code hides devastating performance characteristics behind clean, readable syntax. A `.map()` that calls a repository per element. A `forEach` that issues an HTTP request per item. A stream chain that triggers lazy loading on each entity, producing N+1 queries. Individual `save()` calls inside a loop instead of a batch insert.

These all look clean. They all work in tests with 3 items. They all fall over in production.

The pattern to watch for: iteration over a collection combined with any form of I/O — database, network, file system. If the number of I/O operations scales linearly with the collection size, it is almost certainly wrong.

### Questioning abstractions

Unjustified abstraction is actively harmful. It spreads logic across files, makes the codebase harder to navigate, and creates the illusion of flexibility where none is needed.

An interface with one implementation. A factory that constructs one type. An abstract base class with a single subclass. A configuration object for values that are hardcoded at every call site. A Service → Helper → Util chain where each layer delegates without adding logic.

A useful test: *"If this were inlined, would the code become harder to understand or easier?"* If inlining makes it easier, the abstraction is not earning its place.

### Verifying assumptions

LLM-generated code looks correct. Method names are plausible, argument orders seem right, defaults match expectations. Except sometimes they don't — and the code still reads fluently enough that a surface-level review will miss it.

Areas that are easy to get subtly wrong: library methods where argument order matters and the types do not enforce it. Default behaviors (sort order, timezone, encoding, null handling) that vary across libraries. Methods that have changed semantics across versions. Regex patterns, date parsing formats, and cryptographic parameters — domains where "close" is wrong. Collection types with ordering guarantees that the code assumes but the type does not provide.

When code relies on specific behavior from an external dependency, question whether that behavior has been verified.

### Reading what is NOT in the code

The most dangerous bugs live in code that was never written. An effective review evaluates not just what is present but what is absent.

At system boundaries — user input, external API responses, database results, file I/O — the questions are: what happens when this is null? Empty? Malformed? What happens when the service is slow, or down, or returns a 500? What happens when the database query returns zero rows, or more rows than expected?

LLMs tend toward happy-path thinking. They write code that works with well-formed inputs and available services. But the failure modes that cause production incidents are specific to *this* system, *this* data, *these* users.

Mentally inject failure at every boundary and check whether the code handles it, propagates it sensibly, or silently ignores it.

### Evaluating placement

Logic can be correct and still be in the wrong place. Correct code in the wrong layer creates coupling, bypasses business rules, and makes the system harder to change.

A useful mental test: *"If this same logic needed to be invoked from a different entry point — a CLI, a message queue, a scheduled job — could it be reused without pulling in HTTP concerns or framework dependencies?"* If not, the logic is likely misplaced.

Common misplacements: business rules in controllers, raw database queries bypassing the service layer, authorization scattered across multiple layers, transaction boundaries managed in controllers instead of services.

### Evaluating tests

Coverage is a proxy metric. The more meaningful question is: *"If a bug were introduced in the code under test, would this test fail?"*

LLM-generated tests are particularly deceptive. They achieve high coverage with tests that verify wiring (asserting a mock was called), mirror the implementation (so if the code is wrong, the test is wrong the same way), or only exercise the happy path with trivially simple inputs.

If the tests would not fail when the code is wrong, they are not serving their purpose.

### Excess code

LLM agents are prolific completionists. They see a `create` endpoint and add `update` and `delete`. They see an entity and generate `toString`, `equals`, `hashCode`. They see a potential edge case three layers away and add defensive handling. Each piece looks reasonable in isolation. Together, they double the surface area of the change for no business reason.

This only becomes visible when measured against the intent from Step 1. Code that does not trace back to the intent is not "nice to have." It is code that must be maintained, tested, understood, and debugged, with no corresponding value.

The question is not "is this well-written?" but "does this need to exist?"