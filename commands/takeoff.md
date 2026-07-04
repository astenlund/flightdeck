---
description: "Use after a brainstorm session has produced a signed-off spec and the user wants the feature driven autonomously from planning through shipping."
---

# takeoff

## Scope

Triggered by `/flightdeck:takeoff`, "take off", "take-off", or close conversational variants. Used after a brainstorm session has produced a spec the user has signed off on; takeoff is the autonomous portion of a feature's lifecycle: plan to implementation to ship to follow-up triage. Spec-hardening (`/flightdeck:revise-spec`) is the **pre-flight gate** below: it ensures a signed-off, phase-scoped, reviewed spec exists before planning begins, and if no signed-off spec exists at all it halts and directs the user to the brainstorming skill first.

Build a single flat TaskCreate queue before starting work, containing the numbered steps below with step 4 expanded into the `/flightdeck:land` procedure's individual steps; mark each step completed as its sub-skill graduates and pull through to completion. Accumulate user-decision-required items (`/flightdeck:revise-spec` and `/flightdeck:revise-plan` findings deferred for later, implementation-phase questions and out-of-scope findings, `/flightdeck:revise-code` tech-debt findings) into a single "follow-up items" list maintained in conversation memory; `/flightdeck:land`'s follow-up triage step presents this list. Per-step user gates inside sub-skills (follow-up triage, brainstorming sign-off, etc.) still apply: an auto-accept or autonomous mode that suppresses routine tool-permission prompts does not bypass workflow-level confirmation steps embedded in a skill or command.

## Pre-flight: spec gate

Before step 1, confirm the spec is plan-ready. This gate is where `/flightdeck:revise-spec` runs; keeping it ahead of the procedure puts the natural context-compaction boundary here, between spec-hardening and plan-writing: run it at the brainstorm tail while the design context is warm, then compact, and a fresh takeoff session starts directly at planning with the spec already hardened.

Check, in order:

1. **Signed-off spec exists** for the work this takeoff implements. If not, halt and direct the user to the brainstorming skill first.
2. **`/flightdeck:revise-spec` has graduated for the current phase's sections, and the user has reviewed it.** Scope matters: if the spec graduated in an earlier phase of the same feature, that does not cover this phase - a prior whole-file pass under-scrutinizes sections later phases build, so the run must be scoped to the sections the current phase implements.
   - If this session produced the spec and ran a phase-scoped `/flightdeck:revise-spec` + user review, proceed to step 1.
   - If unsure (e.g. a fresh or compacted session that cannot see the brainstorm), ask the user whether the spec still needs review before planning.
   - If it has not been done, run `/flightdeck:revise-spec` here (scoped to the current phase) to graduation, then the user-review gate, then proceed. Findings flow into the same follow-up list as the numbered steps.

## Triage during revise iterations

If a finding changes the implementation scope (what we're building, not how we describe it), block and ask: that's a brainstorm gap that needs resolving before code lands. Polish-shaped findings (clarity, naming, missing context) go to the follow-up list and the loop continues.

## Procedure

1. Write the implementation plan via the `superpowers:writing-plans` skill.
2. `/flightdeck:revise-plan` - run to graduation per the skill's own criteria.
3. Implement the plan using the `superpowers:subagent-driven-development` skill. One fresh subagent per plan task; dispatch in parallel where the dependency graph allows. Verify each batch's commits landed on the intended branch: a subagent that inherits a git-worktree skill may create a worktree and commit on a separate branch even when told to work on the current one, so check `git log` after each batch rather than assuming. Use subagents even when the plan contains complete verbatim code: dispatch keeps implementation churn (file contents, test output) out of the controller's context window, which is a primary benefit on its own, separate from the fresh-context review.
4. Run the `/flightdeck:land` procedure, carrying the accumulated follow-up items list (from pre-flight and steps 2-3) into its follow-up triage. `/flightdeck:land` owns the entire late-stage workflow (revise-code through triage); its steps are not duplicated here.
