---
name: typescript-best-practices
description: TypeScript/JavaScript module conventions — single source of truth, inferred types, Zod at trust boundaries, edge-case elimination, destructuring, timeless comments, dependency discipline, async patterns. Use when writing or refactoring any .ts/.tsx logic, types, validation, utilities, adapters, or config; for React UI work also apply react-best-practices.
---

# TypeScript best practices

Rules for TypeScript/JavaScript modules (utilities, adapters, shared types, business logic). React UI behavior lives in [react-best-practices](../react-best-practices/SKILL.md); testing in [testing-best-practices](../testing-best-practices/SKILL.md).

## Single source of truth

Every fact lives in exactly one place; everything else derives from it. Derived things cannot drift out of sync — most staleness bugs are two copies of one truth diverging. This is the principle behind many rules here: types infer from values, a constant names a value once, derived data is computed rather than stored. When two places must agree, make one generate the other — or collapse them into one.

## Types and validation

- Infer types from values: `satisfies`, `ReturnType`, `z.infer`, `keyof typeof` on `as const` objects. Write an explicit type only at a boundary consumers depend on — a type maintained in parallel with its value is two sources of truth.
- At boundaries take `unknown` and narrow with a schema or type guard.
- **Trust boundaries** — forms, `JSON.parse`, fetch responses, env vars, any data you don't control — are parsed with **Zod**. Internal shapes use discriminated unions or type guards: Zod at the edge, not in every layer.

## Adapters

Wrap a dependency behind a contract + adapter only when its shape is unstable (generated OpenAPI types, churning third-party APIs): define the type consumers need, map external → domain in one place, and depend on the contract. A pinned, stable library is used directly — fewer layers beats ceremony. The contract may be an inferred type; write an explicit `interface` only when inference stops being the single source of truth.

## Control flow

Eliminate edge cases; handling them is the fallback. Before writing a conditional, try one of three moves that make the case impossible or identical to the normal one:

- **Normalize at the top**: collapse input variants into one canonical shape on the first lines, so the body handles exactly one case. Parsing external data (Zod) is one instance; so are `Array.isArray(x) ? x : [x]` and `range.end ?? Infinity`.
- **Make the edge the normal case**: pick representations where the special case behaves like every other — an empty array needs no guard before `.map`/`.filter`, `items.join(", ")` deletes last-element separator logic, a discriminated union deletes invalid flag combinations.
- **Move the boundary**: check once where the case can actually occur and let types carry the guarantee downstream — a function that takes `Order` instead of `Order | undefined` has nothing to handle; an exhaustive `switch` on a union has no "shouldn't happen" arm.
- Cases that survive become guard clauses that return early; after the guards, the happy path reads top-to-bottom, unnested, with no defensive re-checks of conditions already excluded.
- Each remaining branch is a cost. An `else`/`else if` chain signals a missing normalization, lookup map, or helper; keep an `else` only when it genuinely reads clearer.
- Prefer declarative constructs (map/filter, object lookups) over imperative bookkeeping — fewer branches, and logic bugs have fewer places to hide.
- Destructuring is underused — reach for it when shaping data: object parameters with defaults (`function f({ limit = 50 }: Opts)`), picking fields (`const { id, name } = row`), rest-omit (`const { secret, ...safe } = row`), tuple returns (`const [value, setValue] = ...`). It states the shape you want instead of assigning field by field, and renames stay compiler-checked.

## Comments

- Comments are **timeless**: they describe the code as it stands and hold only what the code cannot say — intent, tradeoffs, non-obvious invariants.
- Change narration ("changed because the user asked...", "previously this did X", dated notes) belongs in the commit message, never in the code. Committed reasoning reads as law to the next agent, who will trust it over their own analysis long after it is stale.
- JSDoc public APIs, hooks, and utilities whose behavior the types don't convey. One or two lines.
- A spot that seems to need a comment is often code that needs a rethink: refactor first, comment what remains.

## Constants and config

Named constants for every value with meaning in business logic; deployment-specific values come from env config (`import.meta.env` in Vite), parsed like any trust boundary.

## Dependencies

- Platform first: modern JS/TS, `Intl`, `URL`, `fetch`, `AbortController`, `structuredClone` before reaching for a package.
- Add a dependency for genuinely hard problems (validation → zod, server state → TanStack Query); write it yourself when it's a small piece of code you'd fully own. Install what you introduce.

## Async

- Parallelize independent awaits with `Promise.all`; a sequential `await` chain implies real data dependency.
- Thread an `AbortSignal` through cancellable or long-running operations.

## Scope and compatibility

- Backwards compatibility is opt-in: ship only the new path and delete what it replaces, unless something is known to depend on the old one — ask instead of assuming.
- Reuse before writing: search for an existing utility to use or extend before adding a near-duplicate.
- Keep related code close: a helper used by one file lives in that file.

## Before finishing

Re-read the full diff against the rules above and fix any drift — adherence decays over a long session, and the diff is where it shows.
