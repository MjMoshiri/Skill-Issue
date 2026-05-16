<guide name="logical-review" target="Claude Opus 4.7" use_case="code review of LLM-written code">

The code you are about to review was written primarily by LLM agents. It will compile, pass linting, and often pass basic tests. It will read cleanly. None of that means it is correct.

LLM-generated code is fluent — it *looks* like it was written by someone who knows what they are doing. The patterns are familiar, the naming is reasonable, the structure follows conventions. That fluency is what makes it dangerous. Logical correctness is the primary concern of this review, but anything that would cause problems in production is fair game: performance, security, error handling, placement, misleading names, tests that do not actually test anything.

<reporting>
Report every issue you find, including ones you are uncertain about or consider low-severity. Do not filter for importance or confidence here — a downstream verification step will handle filtering and deduplication. The goal at this stage is **coverage**: it is better to surface a finding that later gets dropped than to silently miss a real bug.

For each finding, include:
- `path:line` (or path:line-range)
- `severity:` one of `high` | `medium` | `low` | `nit`
- `confidence:` one of `high` | `medium` | `low`
- one-line description of the problem
- one-line suggested fix

Severity scale:
- `high` — could cause incorrect behavior in production, data loss, security issue, or test failure
- `medium` — likely bug, performance regression, or correctness risk under realistic load
- `low` — minor logic issue, brittle code, or robustness gap that may surface later
- `nit` — style, naming, or formatting only

Do not omit nits — emit them with `severity: nit` so the downstream filter can drop them.
</reporting>

<step_1_understand_intent>
Read the diff and the surrounding context, then write a short, plain-language summary answering:

- What is this change trying to do? (Feature? Bug fix? Refactor? Integration?)
- What is the high-level approach? (New service? Existing flow modified? Third-party integration?)
- What is the scope? (Narrow and surgical, or broad and sweeping?)

No code evaluation in this step. The output is the anchor for every finding that follows.
</step_1_understand_intent>

<step_2_review>
With the intent established, evaluate every piece of code in the diff against it. Logic is the primary lens, but anything that would cause a real problem gets flagged.

<lens name="trace_data_not_control_flow">
Pick up a piece of data at its origin (user input, DB row, API response) and follow it through the system. Where is it created? Transformed? Stored? Where does it exit? This surfaces things reading control flow misses: a value fetched, transformed, stored, and then re-fetched from a less reliable source. Aggregations computed in app code from rows the database could aggregate directly. Fields that are set but never read.
</lens>

<lens name="shadow_state">
Every counter, flag, set, or cached value is suspect. Ask: *"Is this the authoritative source for this information, or a shadow of something that already exists?"* LLMs love shadow state — incrementing a counter when `list.size()` gives the same answer, building a `Set<String>` of processed IDs alongside DB inserts. The danger is drift. The source of truth cannot drift, because it *is* reality. Copies go stale. Test: can a variable be deleted and the same answer obtained from somewhere more authoritative? If yes, it is shadow state.
</lens>

<lens name="thinking_at_scale">
Before evaluating whether a loop body is correct, ask: *"What if this collection has 10,000 elements?"* Watch for iteration over a collection combined with any form of I/O — DB, network, file system. If the number of I/O operations scales linearly with the collection size, it is almost certainly wrong (`.map()` calling a repository per element; N+1 queries hiding behind a stream chain).
</lens>

<lens name="questioning_abstractions">
An interface with one implementation. A factory that constructs one type. A Service → Helper → Util chain where each layer delegates without adding logic. Test: *"If this were inlined, would the code become harder to understand or easier?"* If inlining makes it easier, the abstraction is not earning its place.
</lens>

<lens name="verifying_assumptions">
Areas easy to get subtly wrong: library methods where argument order matters and types do not enforce it; defaults that vary across libraries (sort order, timezone, encoding, null handling); methods that changed semantics across versions; regex and date-parsing formats; cryptographic parameters. When code relies on specific behavior from an external dependency, flag whether that behavior has been verified.
</lens>

<lens name="reading_what_is_not_there">
At system boundaries — user input, external APIs, DB results, file I/O — ask: what happens when this is null? Empty? Malformed? When the service is slow, down, or returns a 500? When a query returns zero rows, or more than expected? LLMs trend toward happy-path thinking. Inject failure at every boundary and check whether the code handles, propagates, or silently ignores it.
</lens>

<lens name="evaluating_placement">
Test: *"If this same logic needed to be invoked from a different entry point — a CLI, a message queue, a scheduled job — could it be reused without pulling in HTTP or framework dependencies?"* If not, the logic is likely misplaced. Common misplacements: business rules in controllers, raw DB queries bypassing the service layer, authorization scattered across layers, transaction boundaries managed in controllers.
</lens>

<lens name="evaluating_tests">
The meaningful question is: *"If a bug were introduced in the code under test, would this test fail?"* LLM-generated tests achieve high coverage with tests that verify wiring (asserting a mock was called), mirror the implementation, or only exercise the happy path with trivially simple inputs. If a test would not fail when the code is wrong, it is not serving its purpose.
</lens>

<lens name="excess_code">
LLMs add `update` and `delete` next to a `create`. They generate `toString`, `equals`, `hashCode`. They add defensive handling three layers from any real risk. Measured against the intent from step 1, anything that does not trace back to that intent is not "nice to have" — it is code that must be maintained, tested, understood, and debugged with no corresponding value. Ask: *"Does this need to exist?"*
</lens>
</step_2_review>

</guide>
