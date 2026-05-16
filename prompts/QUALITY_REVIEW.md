# Code Quality Review Guide

This review covers syntax, idiom, and cleanliness — not logic. Logical correctness is handled separately. The goal here is to ensure the code is written the way an experienced Java/TypeScript developer would write it: using the right APIs, following modern conventions, and avoiding patterns that compile fine but are suboptimal, deprecated, or unnecessarily verbose.

## Intention

This is not a nitpick pass. The purpose is to catch code that works but isn't written well — the kind of thing that makes a codebase gradually harder to read, maintain, and onboard into. LLM-generated code is particularly prone to this: it produces code that compiles and passes tests but reaches for verbose, outdated, or roundabout patterns when the language has a direct, idiomatic way to express the same thing.

The bar is: would a senior developer on the team accept this in a PR without commenting? If they'd leave a "you can simplify this" or "there's a better API for this" — that's what this review catches. The goal is clean, high-quality code, not perfection. Use judgment.

## What to Look For

The following are starting points, not a checklist. Use them to calibrate what this review cares about, then apply your own judgment to whatever you find in the code.

**Language idiom** — Is the code using the language the way it's meant to be used? Think: Java streams where loops would be clearer (or vice versa), TypeScript `any` where a proper type exists, manual null checks instead of `Optional`, string concatenation instead of template literals, index-based loops over collections that support `for-of`. If the standard library has a cleaner way, it's worth raising.

**API usage** — Are library and framework APIs being used correctly and optimally? Use context7 MCP (or equivalent docs lookup) to verify when something looks off: deprecated methods, reimplemented built-ins, suboptimal overloads, argument ordering mistakes, builder patterns done the hard way.

**Naming** — Do names say what they mean? A method called `process()` that could be `validateAndSave()`. A boolean named `flag` instead of `isRetryable`. A list called `data` instead of `pendingOrders`. Flag names that would mislead a reader, not style preferences.

**Unnecessary complexity** — Code that works but is more complicated than it needs to be. Nested ternaries that should be an `if`. Chained transformations that could be one pass. Utility wrappers around a single standard library call. Type assertions where inference would work. Verbose null-handling where optional chaining or `Optional` suffices.

**Consistency** — Does the new code match the conventions already in the codebase? If the project uses records, don't introduce a POJO. If existing code uses `async/await`, don't introduce `.then()` chains. Match what's there unless what's there is clearly wrong.

**Modern syntax** — Is the code taking advantage of current language features where they'd be meaningfully clearer or safer? Java: records, sealed classes, pattern matching, text blocks. TypeScript: `satisfies`, const assertions, discriminated unions, nullish coalescing. Don't flag working code just because newer syntax exists.

**Type safety** — TypeScript: type assertions (`as`), non-null assertions (`!`), `any`, missing return types on public functions. Java: raw types, unchecked casts, overly broad catches (`catch (Exception e)`).
