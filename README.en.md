# Desktop Thread Worker

[Overview with Traditional Chinese summary](README.md) | English

A lightweight Planner / Worker skill for solo developers using Codex Desktop.

No extra packages, CLI runner, background service, or separate API key. The skill uses native Codex thread tools and your existing Codex access. Normal account usage still applies.

## Why this exists

This skill was developed for one person maintaining one software project with a limited token budget. Repeated background explanations, manual message forwarding, and unnecessary review rounds can make agent collaboration costly. A large orchestration system can add more work than it removes.

The goal is a small, understandable workflow: keep a persistent Worker, send only task changes, and return enough evidence for review. The Planner must also follow KISS and YAGNI. It must identify a concrete unmet condition before requesting a repair and close the objective when the agreed conditions pass. Optional improvements must not become new acceptance requirements.

## How it works

You set the scope → Planner A dispatches → persistent Worker B implements and verifies → A reviews the evidence.

- Reuse one A/B pair for the same objective.
- Send only the task changes after the first handoff.
- Return the result once. Do not poll by default.
- Stop when the agreed conditions pass. Avoid speculative architecture and moving acceptance criteria.
- Default to five Worker responses per objective. Ask the user when the limit is reached.
- Enable goal mode only on explicit request. Remove the response limit, but retain progress checks, scope boundaries, and user-defined stopping conditions.

## Visible work and user control

Planner and Worker are separate Desktop conversations. The user retains control of the objective, scope, and phase decisions, and can:

- Open the Worker conversation and inspect its visible execution record.
- Give the Worker instructions directly.
- Choose a different Worker and transfer the work state.
- Use the Codex UI to change the Worker's model and reasoning effort, choosing from the options available to the account and selected model.

Model and reasoning settings are user-controlled. The skill preserves each thread's settings unless the user requests a change. No skill edit or separate API setup is needed to use the available UI controls. This does not imply that a setting change alters a turn already in progress.

This is a deliberate workflow choice, not a claim that every subagent interface hides its work or prevents intervention. Subagent visibility and controls vary by tool. This skill uses user-accessible Desktop threads and does not substitute temporary subagents for the Worker.

To replace a Worker, tell A which thread to use. Transfer the current work state, verify the new Worker's authorization and workspace, and preserve the same objective's response count. Resolve any active edits before the replacement starts. Replacing a Worker does not reset the response count or expand permission.

## Requirements

Codex Desktop must expose native thread tools such as `send_message_to_thread`, `read_thread`, and `list_threads`. Creating a Worker also requires `create_thread`; repository task creation requires `list_projects`. Initialization and diagnosis can use `wait_threads`.

Both threads must be able to read the skill. The skill does not add missing tools or override user permissions.

## Install

Download this repository. Copy `SKILL.md` and the `agents` directory into a folder named `desktop-thread-worker` under your Codex skills directory, normally `~/.codex/skills/desktop-thread-worker`.

The included configuration allows implicit invocation for relevant development tasks. This does not authorize creating a Worker; creation still requires an explicit user request.

Start a new task if the skill does not appear in the current task's skill list. No installer is required. Do not overwrite a customized copy without a backup.

## Use

In Planner A:

> Use $desktop-thread-worker. Create a Worker for this project. Scope: fix the reported issue. Done: reproduce the failure, implement the smallest fix, and pass the relevant checks. The Worker may receive scoped tasks and return one result per handoff.

Or select an existing Worker by its task link. A verifies its authorization and workspace before dispatch. B reads this skill on first use and after a notified update.

For optional goal mode:

> For this objective, continue without the five-response limit. Keep the agreed scope and completion conditions. Stop if progress stalls or a decision is needed.

## Example: fix an incomplete alert list

This example adapts a real development task. Project details are anonymized. The messages below illustrate the workflow; they are not a transcript or a verified completion report.

**1. Give Planner A a bounded task**

> Use $desktop-thread-worker with my existing Worker.
>
> Scope: fix an alert list that hides unresolved alerts after they become older than the query window.
>
> Done:
>
> - Older unresolved alerts remain visible.
> - Cleared alerts are excluded.
> - Acknowledged alerts follow the existing domain rules.
> - If the list has a display limit, report the total and whether results are truncated.
> - Keep historical queries unchanged.
>
> Preserve unrelated changes. Do not commit or push.

**2. A sends one handoff to B**

> Handoff: EXAMPLE-001
>
> Fix the unresolved-alert query within the agreed scope. Check the domain rules before changing state filters. Return the changed files, verification results, and remaining limitations.

**3. B implements and returns evidence**

B makes the scoped change, runs relevant checks, and reports what passed, what failed, and what remains unverified.

**4. A reviews the delivery**

A checks the actual changes and evidence against the agreed conditions. If a condition is unmet, A sends a focused repair request that identifies the gap. When all conditions pass, A closes the task.

The user can open B at any time to inspect progress or give instructions directly.

## Extension points

A/B is the default minimum configuration. The same thread messaging pattern can extend to C, D, or other independent Desktop tasks for research, implementation, or review. The user decides whether to add roles. Each added role needs a clear scope, a reporting target, and file ownership to prevent duplicate work or concurrent edits to the same files.

The current skill defines the A/B workflow. Multi-Worker coordination is an extension possibility, not a complete supported or validated mode. Start with A/B and add roles only when the task justifies their coordination and usage costs.

## Limits and status

This is an experimental instruction-based workflow, not a runtime scheduler. Response limits are not platform-enforced spending caps. Delivery, wake-up, and recovery depend on the available Desktop tools. Recent rule changes have passed skill-format validation, but have not all been exercised in end-to-end tasks.

Short handoffs, fewer duplicate reads, and fewer empty round trips are intended to reduce wasted context. No measured token-saving percentage is claimed.

Additional batches require explicit user approval. Keep the cumulative count and set the new limit to the current count plus the authorized additional responses. Worktree acceptance and integration into the destination checkout are separate completion checks.

## License

MIT. See LICENSE.
