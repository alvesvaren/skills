---
name: rewrite-repo
description: "Clean-room rewrite of a proof of concept or legacy codebase: inventory what it does, agree on scope and breaking changes with the user, then rebuild from that inventory under the conventions in this plugin. Manual only. Invoke with /alve-skills:rewrite-repo <path or git URL>."
argument-hint: "<path or git URL>"
disable-model-invocation: true
---

# Rewrite repo

Target: $ARGUMENTS. If no target is given, ask for one.

Rebuild the target as if you started it today, knowing everything it turned out to need. The result is the smallest codebase that does what the agreed scope says, written to the conventions in this plugin.

A clean-room rewrite starts from a description of behavior, not from the old code's structure. The old code is evidence of what the system does. Copy a piece of it when it already meets the sibling skills, and rewrite the pieces that do not.

## Apply every sibling skill

- [typescript-best-practices](../typescript-best-practices/SKILL.md), [testing-best-practices](../testing-best-practices/SKILL.md), and, where they fit, [backend-best-practices](../backend-best-practices/SKILL.md) and [react-best-practices](../react-best-practices/SKILL.md) apply in full to the new code.
- [pushback](../pushback/SKILL.md) governs every scope proposal and breaking change you raise.
- [writing](../writing/SKILL.md) governs the README, commit messages, and everything you write to the user. Read it even though it is manual only.
- Keep the target's own conventions when they encode a real constraint, such as a deploy target or a required runtime. Ask about the rest.

## Size everything to the behavior

- Add a layer, abstraction, or dependency only when an inventory item needs it. Symmetry and "might need it later" are not needs.
- Add no feature the old repo lacked, unless the user approves it during scoping.
- Leave out config options, flags, and extension points that nothing uses.

## 1. Inventory what it does

Read the old repo for behavior, not structure. Run the app and its tests when you can, since observed behavior beats read behavior. Read the git log for why things were added. Record:

- **Surfaces.** Routes, pages, CLI commands, exported APIs, jobs, webhooks.
- **Data.** Schemas, persisted shapes, env config, external services.
- **Logic.** Rules, calculations, permissions, state transitions. Note the edge cases the old code handles and the ones it gets wrong.
- **Consumers.** Who calls each surface. A surface with outside consumers is a compatibility contract.
- **Status.** Mark each item as working, broken, half-built, or dead.

Keep the inventory and plan outside the committed tree, such as in a scratch file or the PR description. They describe a change, and change narration does not belong in the code.

## 2. Settle scope before any code

Ask with AskUserQuestion. Put your recommendation first in each question and skip what the repo already answers.

- **Location.** Recommend a new directory beside the old repo, so the old one stays runnable as a reference. Alternatives are a new branch or a new repo.
- **Stack.** Keep the current stack, or move to the plugin defaults. Recommend the defaults unless a constraint holds the current stack in place.
- **Features.** For each inventory item, propose keep, cut, or defer. Default to keeping what works and is used, and to cutting what is dead. Ask about everything half-built.
- **Compatibility.** Name the surfaces that must keep their exact shape. Default to none unless a consumer is known.
- **Data.** Ask whether existing data must migrate into the new schema.
- **Done.** Agree on what must work before the rewrite can replace the old repo.

## Propose a rescope out loud

A rescope is any change to what the rewrite delivers. Raise each one as its own question, labeled "Rescope", with the reason and the effect. Propose a rescope when you find:

- A feature that is dead, half-built, or costs far more than its use justifies.
- A bug that consumers may depend on, so fixing it is also a break.
- A scope too large for one pass. Propose phases, with phase one a working vertical slice.
- A missing piece the rewrite needs to be correct, such as input validation, migrations, or auth.

Rescoping continues after the plan. When the build reveals something that changes scope, stop and ask. Never drop or add a feature silently.

## 3. Recommend breaking changes and show their cost

A rewrite is the cheapest moment to break things, so recommend a break wherever the new design is better. A renamed or reshaped API that only reads better is a valid break.

List each break with what changes, why, who it hits, and how they migrate. Label the why as defect, convention, or taste, as in pushback. Taste is a valid reason, but the user decides only after seeing the consequences.

Present the plan before writing code. It holds the inventory with each keep or cut decision, the list of breaks, and a sketch of modules and key dependencies. Wait for approval.

## 4. Build from the inventory

1. Set up tooling first: package manager, strict `tsconfig`, lint, format, test runner, and CI if the old repo had any. Install dependencies at their latest stable versions.
2. Build in vertical slices, the most valuable item first.
3. For each logic item, write the test from the inventory first, then implement until it passes.
4. Copy old code that already meets the sibling skills. Fit it into the new structure instead of letting it decide that structure.
5. Reproduce a known bug only when the user chose to keep it for compatibility.

## 5. Test what can regress

- Give each logic item in the inventory a test. Plumbing gets none.
- Skip porting the old tests one-to-one. Keep the cases that pin behavior and drop the ones that pin structure.
- When the old app runs, feed real inputs to both versions of each kept surface and compare the outputs. Every difference is either a listed break or a bug in the new code.

## 6. Document what the code cannot say

- Write a README with what the project does and how to run, configure, test, and deploy it. Leave out anything the code or the file tree already shows.
- Write comments only for intent, tradeoffs, and non-obvious invariants.
- When a surface with consumers broke, write a `MIGRATION.md` with the before, the after, and the steps for each break.

## Before finishing

1. Walk the inventory. Every item is implemented, cut with the user's approval, or changed as a listed break.
2. Run the typecheck, lint, tests, and the app itself.
3. Re-read the whole new tree against the sibling skills, not only the last diff. Every file is new, so every file can drift.
4. Report to the user what was cut, what broke, and what is left for a later phase.
