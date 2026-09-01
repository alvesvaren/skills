---
name: react-best-practices
description: React/UI conventions for Vite + React 19 + TanStack Query + Tailwind v4 — Suspense and error boundaries for async UI, near-zero useEffect, small named components, flex-first layout, CVA variants. Use when building or refactoring React components, hooks, forms, queries, or Tailwind styling; typescript-best-practices still applies to all .tsx code.
---

# React best practices

Conventions for React UI code. Every `.ts`/`.tsx` module also follows [typescript-best-practices](../typescript-best-practices/SKILL.md) (types, Zod, adapters, control flow, comments, dependencies).

**Stack defaults:** React 19, Vite, TanStack Query, Tailwind v4. OpenAPI-generated client code (often `src/client/`) is an unstable external shape — adapt it (see Adapters in typescript-best-practices).

## Data and effects

- Server/async state lives in TanStack Query. Reach for Zustand (or similar) only when local UI state is large or cross-cutting.
- Renders are pure: UI derives from props, state, and query results.
- `useEffect` is a last resort: reach first for query `select`/callbacks, event handlers, derived values, or URL/search params. A genuinely needed effect lives in a dedicated `useThing()` hook; an inline effect is reserved for a tiny one-off sync that can't live anywhere else.

## Errors and loading

- Async/render-tree loading and failure: Suspense + error boundaries (or route-level equivalents), e.g. with suspense queries.
- Event handlers, timers, and mutation paths bypass boundaries — handle those with try/catch, mutation `onError`, or toasts.

## Structure

- Small, focused components. Extract when it names a concept or enables reuse; keep JSX inline when splitting would hurt locality.
- Multiple components per file only when tightly coupled; split a very long file by domain.
- Compute view-ready data in named variables above the `return`; keep `{...}` expressions in JSX trivial.

## DOM and performance

- Imperative DOM/browser APIs live in custom hooks, never sprinkled through component bodies.
- `useMemo`/`React.memo`/`useCallback` only when the cost is measured or obvious — React Compiler is shrinking the need further.
- Several independent `useState`s are fine. Coupled updates, stored derived state, or state that belongs in the URL/query cache signal extraction into a hook, store, or derived value.

## Tailwind and CSS

- Flex first; grid when the layout is truly 2D (explicit rows/columns, overlapping areas).
- Theme tokens over arbitrary values; an arbitrary value is for a clearly one-off case. `min-*` + `max-*` together is a fine intentional clamp.
- Variant-heavy presentational components use CVA — add `class-variance-authority` as a direct dependency when introducing it.
- Format dates and numbers with `Intl` APIs (`Intl.DateTimeFormat`, `Intl.NumberFormat`), not hand-rolled strings.
