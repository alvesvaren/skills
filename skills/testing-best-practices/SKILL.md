---
name: testing-best-practices
description: Testing conventions — Vitest integration tests as the regression net, red-green TDD for bug fixes and algorithms, unit tests where logic earns them, Playwright for critical E2E workflows, edge-level mocking. Use when writing or changing tests, fixing a reported bug, or deciding what a change needs tested.
---

# Testing best practices

A test exists to pin behavior that could plausibly regress, or to drive a fix red-green. Assert observable behavior: a test proving `1 + 1 === 2`, or that an error message contains its own words, pins nothing — delete it rather than write it.

Vitest is the default runner.

## Layers

- **Integration tests (the bulk):** exercise a feature through its public surface — component + hooks + real data flow, network mocked at the edge. This is the regression net that catches changed behavior.
- **Unit tests:** for logic-heavy code — parsing, date math, algorithms, tricky branching. Pin behavior where the logic earns it.
- **E2E (Playwright):** critical workflows only — the flows that must never break. A thin layer, not a mirror of the integration suite.

## Red-green loop

Bug fixes and algorithmic work go red-green:

1. **Red:** reproduce the bug (or express the target behavior) as a failing test. A bug test must fail for the reported reason before any fix is written.
2. **Green:** implement until it passes.
3. Keep the test — it is now the regression guard.

Feature work with meaningful logic gets the same loop: write the goal as tests first, then implement toward them. Trivial plumbing (wiring a prop, rendering static content) ships without tests.

## Mocking

- Mock the edge: network (MSW or fetch-level), time, randomness, external services.
- Own modules run real. Mock an internal module only when isolating it is genuinely expensive (heavy auth context, native bindings) — treat every internal mock as a cost to justify.

## Assertions

- Assert outcomes users or callers observe: rendered text and roles, returned values, persisted state, calls that crossed the edge.
- Assert specific values: one `toEqual` with the real expected object beats a chain of `toBeDefined`/`toContain` shape checks.
