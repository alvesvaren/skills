---
name: testing-best-practices
description: "Testing conventions: Vitest integration tests as the regression suite, red-green TDD for bug fixes and algorithms, unit tests where logic earns them, Playwright for critical E2E workflows, edge-level mocking. Use when writing or changing tests, fixing a reported bug, or deciding what a change needs tested."
---

# Testing best practices

A test exists to pin behavior that could plausibly regress, or to drive a fix red-green. Assert observable behavior. A test that proves `1 + 1 === 2`, or that an error message contains its own words, pins nothing. Delete it rather than write it.

Vitest is the default runner.

## Put most tests at the integration layer

- Make integration tests the bulk. Exercise a feature through its public surface: component, hooks, and real data flow, with the network mocked at the edge. This suite catches changed behavior.
- Write unit tests for logic-heavy code: parsing, date math, algorithms, tricky branching. Pin behavior where the logic earns it.
- Reserve Playwright E2E tests for critical workflows: the flows that must never break. Keep this layer thin. Do not mirror the integration suite.

## Work red-green

Bug fixes and algorithmic work go red-green:

1. Red: reproduce the bug, or express the target behavior, as a failing test. A bug test must fail for the reported reason before you write any fix.
2. Green: implement until the test passes.
3. Keep the test. It now guards against regression.

Feature work with meaningful logic gets the same loop: write the goal as tests first, then implement toward them. Trivial plumbing, such as wiring a prop or rendering static content, ships without tests.

## Mock the edge only

- Mock the edge: network (MSW or fetch-level), time, randomness, external services.
- Run your own modules real. Mock an internal module only when isolating it is genuinely expensive, such as a heavy auth context or native bindings. Treat every internal mock as a cost to justify.

## Assert observable outcomes

- Assert what users or callers observe: rendered text and roles, returned values, persisted state, calls that crossed the edge.
- Assert specific values. One `toEqual` with the real expected object beats a chain of `toBeDefined` and `toContain` shape checks.

## Before finishing

Re-read the tests you touched against the rules above. Confirm that every bug fix has a test that failed for the reported reason first. Fix any drift.
