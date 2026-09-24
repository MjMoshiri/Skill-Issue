<guide name="quality-review">
This review covers syntax, idiom, and cleanliness: is the code written the way an experienced developer would write it — right APIs, modern conventions, no patterns that compile fine but are suboptimal, deprecated, or verbose.
<intention>
Catch code that works but is not written well — the kind that makes a codebase gradually harder to read, maintain, and onboard into. LLM-generated code is especially prone to this: it compiles and passes tests but reaches for verbose, outdated, or roundabout patterns when the language has a direct, idiomatic alternative.
</intention>
<scope>
Review the diff and report findings; leave the code unchanged. Judge the code against the language and library versions the project actually uses.
</scope>
<reporting>
Report every issue, including uncertain or low-severity ones. A downstream step filters; your job is coverage — better to surface a finding that later gets dropped than to silently miss one.
</reporting>
<output_format>
For each finding: `file:line`, the lens, what is wrong, your confidence, and estimated severity.
</output_format>
<lenses>
Not exhaustive — calibrate on these, then apply judgment. Examples span several languages; apply the principle to the language under review.
<lens name="language_idiom">
Is the code using the language as intended? Manual index loops where iteration constructs exist (`for-of`, `enumerate`, `range`, `Enum`/comprehensions, iterators); string concatenation where interpolation or formatting exists (template literals, f-strings, `#{}`, `fmt.Sprintf`); hand-rolled null/None/nil checks where a dedicated construct exists (optional chaining, `??`, pattern matching, `with`); functional chains where a plain loop is clearer, or vice versa. If the language or stdlib expresses it more cleanly, raise it.
</lens>
<lens name="api_usage">
Are library and framework APIs used correctly and optimally? Deprecated methods or modules; reimplemented built-ins (hand-written sorting, grouping, deep-copying, date math); suboptimal overloads or call signatures; argument-order mistakes; builder/constructor patterns done the hard way; APIs from a different major version than the project pins. Verify with `ctx7` when something looks off.
</lens>
<lens name="naming">
Do names say what they mean? `process()` that could be `validate_and_save()`; a boolean `flag` instead of `is_retryable`; `data` instead of `pending_orders`. Flag names that mislead a reader, not style preferences — plus names violating the language's conventions (camelCase in a snake_case codebase or vice versa).
</lens>
<lens name="unnecessary_complexity">
Code more complicated than it needs to be: nested conditionals that should be a plain `if` or a pattern match; chained transformations that could be one pass; utility wrappers around a single stdlib call; explicit annotations or casts where inference works; verbose null/error handling where a dedicated construct suffices (optional chaining, `?` propagation, `errors.Is`/`errors.As`, context managers, `with`).
</lens>
<lens name="duplication_and_reuse">
Does this write something that already exists? Work down the ladder: existing codebase helper? Stdlib? Native platform feature? Already-installed dependency? Flag copy-pasted or near-duplicate logic that should call the existing implementation, reinvented helpers (a second `formatDate`, a third retry wrapper), and reimplemented stdlib/platform features (a hand-rolled date picker where a native `<input type="date">` works). Also flag fixes patched into one caller when the shared function is the root cause — the fix belongs in the shared function once, not per call site.
</lens>
<lens name="speculative_abstraction">
Flexibility nobody asked for: interfaces or base classes with a single implementation; generics instantiated with exactly one type; config options, feature flags, or plugin hooks with no second consumer; indirection layers (factory, manager, strategy) around one concrete behavior; parameters every caller passes identically. Abstractions are earned by a second real use case, not a hypothetical future. The best outcome for a diff like this is getting shorter. Caveat: one small test or assert-based self-check for non-trivial logic is the minimum, not bloat — don't flag it for deletion.
</lens>
<lens name="dead_code">
Code that no longer participates in the program: unused functions, methods, variables, imports, exports; unreachable branches (always-false conditions, code after an unconditional return, clauses the compiler or type checker warns can never match); commented-out blocks kept "just in case"; debug output left in (`print`, `console.log`, `IO.inspect`, `dbg`); exported symbols with no callers; leftover scaffolding the diff replaced. Version control remembers.
</lens>
<lens name="dependency_minimalism">
Is a new dependency added for something the codebase can already do? Flag packages that duplicate the stdlib, a native platform feature, or an existing dependency (a second HTTP client, a utility library for one function, a date library where stdlib date math suffices). Each dependency is a maintenance and supply-chain cost; it should buy something the existing toolchain genuinely cannot.
</lens>
<lens name="consistency">
Does new code match the codebase's conventions? If the project uses immutable data structures, don't introduce a mutable class-with-setters; if it uses one async style, error-wrapping style, logger, or test framework, don't introduce another. Match what's there unless it's clearly wrong.
</lens>
<lens name="modern_syntax">
Are current language features used where meaningfully clearer or safer? Pattern matching and destructuring; concise data types (dataclasses, structs, typed records) over boilerplate classes; generic helpers from the stdlib (`slices`, `maps`, `min`/`max`) over hand-rolled loops; multiline/raw strings over concatenated fragments; nullish/None-coalescing operators; discriminated unions or tagged tuples for closed variant sets. Don't flag working code just because newer syntax exists — only where it's a real clarity or safety win.
</lens>
<lens name="type_safety">
Does the code weaken the type system's guarantees? Escape-hatch types (`any`, `interface{}`, `Any`, untyped maps passed around as structs) where a proper type exists; unchecked casts and non-null assertions; missing annotations or specs on public interfaces in gradually-typed codebases; overly broad exception handling (bare `except:`, `rescue _`, swallowed error returns) that hides failure modes.
</lens>
</lenses>

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
