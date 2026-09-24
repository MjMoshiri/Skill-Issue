<guide name="logical-review">
This review covers logical correctness of code written primarily by LLM agents. It will compile, pass linting, often pass basic tests, and read cleanly. None of that means it is correct.
<intention>
LLM-generated code is fluent — it *looks* like it was written by someone who knows what they are doing: familiar patterns, reasonable naming, conventional structure. That fluency is what makes it dangerous. Logic is the primary concern, but anything that would cause problems in production is fair game: performance, security, error handling, placement, misleading names, tests that do not actually test anything.
</intention>
<scope>
Review the diff and report findings; leave the code unchanged. Judge the code against the language and library versions the project actually uses.
</scope>
<reporting>
Report every issue, including uncertain or low-severity ones. A downstream step filters and deduplicates; your job is coverage — better to surface a finding that later gets dropped than to silently miss a real bug.
</reporting>
<output_format>
For each finding: `file:line`, the lens, what is wrong, your confidence, and estimated severity.
</output_format>
<step_1_understand_intent>
Read the diff and surrounding context, then write a short plain-language summary: what is this change trying to do (feature, fix, refactor, integration)? What is the high-level approach? What is the scope (narrow and surgical, or broad and sweeping)? No code evaluation in this step — the output is the anchor for every finding that follows.
</step_1_understand_intent>
<step_2_review>
With the intent established, evaluate every piece of code in the diff against it. Logic is the primary lens, but anything that would cause a real problem gets flagged. The lenses below are not exhaustive — calibrate on them, then apply judgment. Examples span several languages; apply the principle to the language under review.
<lens name="trace_data_not_control_flow">
Pick up a piece of data at its origin (user input, DB row, API response) and follow it through the system: where is it created, transformed, stored, and where does it exit? This surfaces what reading control flow misses: a value fetched, transformed, stored, then re-fetched from a less reliable source; aggregations computed in app code from rows the database could aggregate directly; fields that are set but never read.
</lens>
<lens name="shadow_state">
Every counter, flag, set, or cached value is suspect: is it the authoritative source for this information, or a shadow of something that already exists? LLMs love shadow state — incrementing a counter when the collection's length gives the same answer, keeping a set of processed IDs alongside DB inserts, copying server data fetched by a query cache into a client-side store. The source of truth cannot drift, because it *is* reality; copies go stale. Test: can the variable be deleted and the same answer obtained from somewhere more authoritative? If yes, it is shadow state.
</lens>
<lens name="thinking_at_scale">
Before evaluating whether a loop body is correct, ask: what if this collection has 10,000 elements? Watch for iteration combined with any form of I/O — DB, network, file system. If the number of I/O operations scales linearly with collection size, it is almost certainly wrong (a per-element query inside `Enum.map` or a `for` loop; N+1 queries hiding behind lazy-loaded relations or missing preloads). The same applies to serialization points: a single process, lock, or GenServer that every request funnels through.
</lens>
<lens name="concurrency_and_lifetimes">
Code that runs concurrently or outlives its caller: goroutines or tasks started from a request context that is cancelled when the handler returns; background work detached with no owner, so its failures vanish (a fire-and-forget task whose reference is dropped, an un-awaited coroutine); one DB session or connection shared across concurrent tasks; blocking calls inside async code that stall the event loop; resources opened without a guaranteed close (cursors, streams, files). Ask: who owns this work, what cancels it, and where do its errors go?
</lens>
<lens name="questioning_abstractions">
An interface with one implementation; a factory that constructs one type; a Service → Helper → Util chain where each layer delegates without adding logic. Test: if this were inlined, would the code become harder to understand or easier? If inlining makes it easier, the abstraction is not earning its place.
</lens>
<lens name="verifying_assumptions">
Areas easy to get subtly wrong: library methods where argument order matters and types do not enforce it; defaults that vary across libraries (sort order, timezone, encoding, null handling, naive vs aware datetimes); methods that changed semantics across major versions; dedup or uniqueness features that prevent duplicate inserts but not duplicate execution; regex and date-parsing formats; cryptographic parameters. When code relies on specific behavior from an external dependency, check it with `ctx7`.
</lens>
<lens name="reading_what_is_not_there">
At system boundaries — user input, external APIs, DB results, file I/O — ask: what happens when this is null, empty, or malformed? When the service is slow, down, or returns a 500? When a query returns zero rows, or more than expected? When a job or webhook is delivered twice? LLMs trend toward happy-path thinking. Inject failure at every boundary and check whether the code handles, propagates, or silently ignores it — including error branches that collapse distinct failures into one (a catch-all `else`, a swallowed error return).
</lens>
<lens name="evaluating_placement">
Test: if this same logic needed to be invoked from a different entry point — a CLI, a message queue, a scheduled job — could it be reused without pulling in HTTP or framework dependencies? If not, the logic is likely misplaced. Common misplacements: business rules in controllers or handlers, raw DB queries bypassing the service layer, authorization scattered across layers, transaction boundaries managed at the edge.
</lens>
<lens name="evaluating_tests">
The meaningful question: if a bug were introduced in the code under test, would this test fail? LLM-generated tests achieve high coverage with tests that verify wiring (asserting a mock was called), mirror the implementation, or only exercise the happy path with trivially simple inputs. A test that would not fail when the code is wrong is not serving its purpose.
</lens>
<lens name="excess_code">
LLMs add `update` and `delete` next to a `create`; generate equality, hashing, and string-representation boilerplate nobody calls; add defensive handling three layers from any real risk. Measured against the intent from step 1, anything that does not trace back to that intent is not "nice to have" — it is code that must be maintained, tested, understood, and debugged with no corresponding value. Ask: does this need to exist?
</lens>
</step_2_review>

<context7>
Use the `ctx7` CLI to fetch docs and verify usage when something looks off.

```bash
ctx7 library <name> <query>                   # Step 1: resolve library ID (lists available versions)
ctx7 docs <libraryId> <query>                 # Step 2: fetch docs
ctx7 docs <libraryId>/<version> <query>       # docs for the version the project uses
```

- Library IDs need a `/` prefix — `/facebook/react`, not `facebook/react`
- Resolve the ID first — `ctx7 docs react "hooks"` fails without it
</context7>

</guide>
