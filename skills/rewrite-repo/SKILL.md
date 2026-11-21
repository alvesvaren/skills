---
name: rewrite-repo
description: "Re-architect an existing codebase as a whole: read how it grew, derive the architecture it should have today, present it as one spec with diagrams, get the user's approval, then rebuild or refactor toward it. Covers any codebase, with extra weight on backend plus frontend and the contract between them. Manual only. Invoke with /alve-skills:rewrite-repo <path or git URL>."
argument-hint: "<path or git URL>"
disable-model-invocation: true
---

# Rewrite repo

Target: $ARGUMENTS. If no target is given, ask for one.

Find the architecture the project would have if it were started today by an engineer who knew everything it turned out to need. Design the whole system at once, not one module at a time, because the pieces constrain each other. The contract shapes both sides, and the data model shapes the contract. The old code is evidence of what the system does and where it keeps changing. It does not decide the new structure.

The goal is better code. The spec exists so the user can judge the design and learn why it is better. Keep each explanation brief, and spend the words on the decisions that matter.

[typescript-best-practices](../typescript-best-practices/SKILL.md), [testing-best-practices](../testing-best-practices/SKILL.md), [backend-best-practices](../backend-best-practices/SKILL.md), and [react-best-practices](../react-best-practices/SKILL.md) apply in full to the new code. [pushback](../pushback/SKILL.md) applies to every recommendation, and [writing](../writing/SKILL.md) to the spec and everything you write to the user. Read the writing skill even though it is manual only.

## Vocabulary

Use these terms in the spec and in your reasoning:

- **Deep module:** a lot of behavior behind a small interface. A **shallow module** has an interface nearly as complex as its implementation.
- **Seam:** the place where a module's interface lives, where behavior can change without editing the callers.
- **Axis of change:** something the history shows gets added or replaced repeatedly, such as a new provider, report type, or page.
- **Deletion test:** imagine deleting a module and inlining it into its callers. If complexity vanishes, it was a pass-through. If the same complexity reappears across several callers, it earns its place.

## Orchestrate with subagents

Invoking this skill is the user's request to use subagents. Keep the main context and the main model for decisions, and hand everything else out:

- **Reading and summarizing** go to the smallest model the harness offers. Use a mid-size model for an area that is large or tangled.
- **Implementing a slice against a fixed contract** goes to a mid-size model.
- **Designing twice and the final review** stay on the main model, since they are decisions.
- Set the model on each subagent explicitly. For cheap work, skip subagents that inherit the conversation, since they usually run on the main model.
- Each new subagent starts cold. Give it a brief with its area, what to look for, and the shape of its report.
- Run independent subagents in parallel.

## 1. Read the repo in parallel

Spawn these subagents at once, and keep the main context for the design:

- **One read-only subagent per area,** such as backend, frontend, data and migrations, and infra and CI. It reports the area's behavior, with each item marked working, broken, half-built, or dead, and who consumes it. It also reports where the area hurts: code that is hard to test, tests that mock internal modules, shallow modules, and changes that spread across many files.
- **One history subagent.** It reports hotspots, files that change together, and the commits that added a new variant of something, with how many files each commit touched. The repeated variants are the axes of change.
- **One seam subagent,** when the repo has both a frontend and a backend. It reports how types cross the wire, where input is validated, how errors and auth reach the client, and how the client caches server state.

When you can, run the app and its tests yourself, since observed behavior beats read behavior.

## 2. Derive the architecture

### Remove before you add

Run the deletion test on every layer, wrapper, and abstraction the old code has. A project that grew organically carries pass-throughs that were added for symmetry or for a future that never came. Removing them is usually the largest simplification available.

### Make every new pattern cite evidence

A pattern earns its place with one of two kinds of evidence:

- **An axis of change.** At least three variants exist, or the history shows new ones keep arriving. Make adding a variant cheap, ideally one new file plus one registration.
- **A real seam.** Something varies across it today. One adapter is a hypothetical seam, and two adapters, such as production and a test double at an I/O edge, make a real one.

Merge near-duplicate code only when the copies change together in the history, since that shows they are the same knowledge. Code that looks alike but encodes two rules stays separate, since merging it makes one edit silently change both rules.

### Design for the test surface

The interface is the test surface. Shape each module so a test reaches it through the same interface its callers use, with only the edge mocked. When a test needs to reach past the interface, the module has the wrong shape. When an isolated test is the only reason to extract code, leave the code in place, since the bugs hide in how it is called.

### Treat frontend and backend as one system

When the repo has both sides, plan them together:

- Share code between the two only where it crosses the wire: schemas, inferred types, error codes. Keep UI and server code out of shared packages.
- Use a monorepo when one team ships both sides together. Keep separate repos when they deploy and version independently.

### Decide the cross-cutting pieces

Decide each cross-cutting piece explicitly, with its evidence. A design system means shared primitives with CVA variants and theme tokens. Set one up when the same UI elements repeat across many screens or drift visibly apart. Otherwise keep Tailwind and extract components as repetition shows up.

### Design the biggest decisions twice

For the one or two decisions that shape the rest, such as the module layout or the contract, spawn two subagents in parallel. Give each the same evidence and a different constraint, and compare their proposals in the spec. Settle smaller decisions directly.

## 3. Present the spec

The spec covers the whole system, so the user sees how the decisions fit together. Publish it as an Artifact when the tool is available, otherwise as an HTML file outside the repo. Split it into sections or pages as needed. It holds:

- **Overview.** A diagram of the target architecture: modules, dependency direction, the frontend-backend contract, and the axes of change with where a new variant goes.
- **Decisions.** One entry per significant change: the evidence, the change, why it is better, and its cost. Mark each Strong, Worth exploring, or Speculative. Add a before and after diagram where the structure is easier to see than to read, and keep each diagram to the one decision it explains.
- **Removed and rejected.** Abstractions the deletion test removed, and patterns you considered and rejected, with the reason. This list shows where you held back.
- **Scope.** Behavior grouped by area, marked keep, cut, or defer. Default to keeping what works and is used, and to cutting what is dead or half-built.
- **Breaking changes.** What changes, why, who it hits, and how they migrate.
- **Approach and phases.** Recommend refactoring in place when most behavior works and the target is reachable by a sequence of moves. Recommend a rewrite beside the old code when the stack changes or most behavior is broken or half-built. Order the phases by what each change unlocks, with phase one a working vertical slice.

Then ask the user what the repo cannot answer, with the harness's question tool or in chat. The usual questions cover cuts of features that may have hidden users, breaks that hit consumers, data migration, and the approach. Update the spec with the answers and wait for approval before writing code. After approval, never drop or add scope silently. When the build shows the spec is wrong, stop and ask.

## 4. Build toward the spec

1. Pin current behavior first. Write integration tests against the old code for every kept surface. They are the acceptance criteria for the new code, and a listed break changes its test on purpose.
2. Build the first slice alone, through backend, contract, and frontend together, until the contract is fixed. A slice that crosses the seam first exposes contract problems before other slices depend on the contract.
3. Once the contract is fixed, build independent slices in parallel subagents, each in its own worktree. Before that point, parallel slices would each guess a different contract.
4. Copy old code that already meets the sibling skills, and fit it into the new structure.
5. When refactoring in place, make one move per commit and keep the tests green between moves.

Add an architecture section to the README with the module map and, for each axis of change, where to add a new variant. When a surface with consumers broke, write a `MIGRATION.md` with the steps for each break.

## Before finishing

1. Walk the spec. Every scope item is built, cut with approval, or changed as a listed break.
2. Spawn a reviewer subagent with fresh context. It reads the whole new tree against the sibling skills and the spec's decisions, and reports drift and any abstraction the spec does not justify. Fix what it finds.
3. Report what was cut, what broke, and what is left for a later phase.
