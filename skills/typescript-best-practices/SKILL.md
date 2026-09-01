---
name: typescript-best-practices
description: TypeScript/JavaScript module conventions — inferred DRY types, no any, Zod at trust boundaries, adapters only for unstable dependencies, early-return control flow, dependency discipline, async patterns. Use when writing or refactoring any .ts/.tsx logic, types, validation, utilities, adapters, or config; for React UI work also apply react-best-practices.
---

# TypeScript best practices

Rules for TypeScript/JavaScript modules (utilities, adapters, shared types, business logic). React UI behavior lives in [react-best-practices](../react-best-practices/SKILL.md); testing in [testing-best-practices](../testing-best-practices/SKILL.md).

## Types and validation

- Infer types from values: `satisfies`, `ReturnType`, generics. Write an explicit type only at a boundary consumers depend on.
- `any` is forbidden. At boundaries take `unknown` and narrow with a schema or type guard.
- **Trust boundaries** — forms, `JSON.parse`, fetch responses, env vars, any data you don't control — are parsed with **Zod**. Internal shapes use discriminated unions or type guards: Zod at the edge, not in every layer.

## Adapters

Wrap a dependency behind a contract + adapter only when its shape is unstable (generated OpenAPI types, churning third-party APIs): define the type consumers need, map external → domain in one place, and depend on the contract. A pinned, stable library is used directly — fewer layers beats ceremony. The contract may be an inferred type; write an explicit `interface` only when inference stops being the single source of truth.

## Control flow

Eliminate branches; handling them is the fallback. Before writing a conditional, ask whether the code can be shaped so the case cannot occur:

- **Parse, don't validate**: normalize messy input into one canonical shape at the top of the function, so the body handles exactly one case instead of re-checking variants throughout.
- Choose types and data structures where invalid states are unrepresentable (discriminated unions over boolean flags, non-empty parsing over `.length` checks downstream).
- Special cases that survive become guard clauses that return early; after the guards, the happy path reads top-to-bottom, unnested, with no defensive re-checks of conditions already excluded.
- Each remaining branch is a cost. An `else`/`else if` chain signals a missing normalization, lookup map, or helper; keep an `else` only when it genuinely reads clearer.
- Prefer declarative constructs (map/filter, object lookups) over imperative bookkeeping — fewer branches, and logic bugs have fewer places to hide.

## Comments

- Comments are **timeless**: they describe the code as it stands and hold only what the code cannot say — intent, tradeoffs, non-obvious invariants.
- Change narration ("changed because the user asked...", "previously this did X", dated notes) belongs in the commit message, never in the code. Committed reasoning reads as law to the next agent, who will trust it over their own analysis long after it is stale.
- JSDoc public APIs, hooks, and utilities whose behavior the types don't convey. One or two lines.
- A spot that seems to need a comment is often code that needs a rethink: refactor first, comment what remains.

## Constants and config

Named constants for every value with meaning in business logic; deployment-specific values come from env config (`import.meta.env` in Vite), parsed like any trust boundary.

## Dependencies

- Platform first: modern JS/TS, `Intl`, `URL`, `fetch`, `AbortController`, `structuredClone` before reaching for a package.
- Add a dependency for genuinely hard problems (validation → zod, server state → TanStack Query); write it yourself when it's a small piece of code you'd fully own.
- Every imported package is a **direct** dependency in `package.json` — install it when introducing the pattern, never rely on transitive availability.

## Async

- Parallelize independent awaits with `Promise.all`; a sequential `await` chain implies real data dependency.
- Every promise is awaited or explicitly handled — no floating promises.
- Thread an `AbortSignal` through cancellable or long-running operations.

## Scope and compatibility

- Backwards compatibility is opt-in: ship only the new path unless something is known to depend on the old one — ask instead of assuming.
- Reuse before writing: search for an existing utility to use or extend before adding a near-duplicate.
- Keep the surface small: delete code your change makes dead; shorter code, fewer bugs.
- Keep related code close: a helper used by one file lives in that file.
