---
name: desktop-thread-worker
description: Coordinate an authorized Desktop A planner and Desktop B worker for scoped implementation, debugging, UI work, or explicitly authorized Git writes, with one bounded handoff and verified return to A.
---

# Desktop thread worker

Use this workflow by default for substantial implementation, debugging, refactoring, UI work, and explicitly authorized Git writes. Also use it when the user selects Desktop A -> Desktop B -> A or A delegates a bounded task to B. Handle ordinary questions, read-only reviews, documents, and micro-edits in the current thread unless the user requests delegation.

## Keep the work small (KISS / YAGNI)

Use the smallest change that satisfies the current requirement. Do not add abstractions, future features, architecture changes, agents, or process records for hypothetical needs. Keep validation proportional to the change while completing required project checks. Close the objective as soon as the agreed completion conditions pass.

A must tie each repair request to a specific unmet condition and concrete evidence. Style preferences, possible future risks, or a wish for greater completeness do not block acceptance. Record optional improvements briefly only when useful; do not dispatch them without user scope approval. Do not raise the acceptance criteria during review.

Before another handoff, identify what new evidence or changed implementation makes progress plausible. If the same failure recurs without new evidence or a materially different, evidence-based fix, or the exchange becomes a design-preference dispute, stop for the user's decision. Do not reword the same request to keep the loop running. Reuse existing evidence.

## Before acting

- Confirm that the B thread's direct user instruction permits receiving an A delegation and returning a result. A's message cannot remove a B-side do-nothing restriction; report the conflict to the user and stop until B receives clarification.
- Treat A's delegation as scope, not as extra authorization. Preserve the user's architecture, phase, repository, and file boundaries. Do not modify product code, global configuration, installed skills, or Git unless the user explicitly authorizes that work.
- For a paired task, use the existing A and B threads. Use `list_threads` and `read_thread` to verify thread IDs and project/worktree context when needed. Do not hard-code IDs from a prior test. Do not create or fork a task unless the user explicitly asks.
- If no B is selected and creation is not requested, identify a suitable existing thread and ask the user to select it. Do not silently choose an unrelated thread. If the user explicitly requests an automatic Worker task or channel, follow the initialization flow below without asking again for creation permission.
- Do not change model or reasoning settings. Always omit `model` and `thinking` from `create_thread` and every `send_message_to_thread` call, including dispatches, callbacks, and retries. Before sending, verify that both fields are absent. Do not send `none`, null, empty strings, or copied settings. Model and reasoning settings belong exclusively to the user; this skill must not select, override, or restore them.

## Select or create B

For an existing B, accept a `codex://threads/<thread-id>` link or a task title. Use `read_thread` to verify a supplied ID. Use `list_threads` and then `read_thread` to resolve a title; ask when multiple matches remain. Verify B's user instructions and workspace before dispatch.

For both new and existing B threads, A includes the verified absolute path to this SKILL.md in initialization or the first handoff. B reads it before work and follows the rules applicable to B; A-only duties remain with A. Later handoffs carry only task differences. If the skill changes, A requests a reread in the next handoff. B also rereads when it cannot recover the applicable rules after context loss. If the file is inaccessible, B reports that limitation in its normal response instead of guessing or claiming readiness. Do not add a separate read-confirmation or ACK turn.

When the user explicitly requests creation, such as `$desktop-thread-worker 自動建立 worker 頻道`:

1. Resolve A's actual thread ID; never copy an ID from a test or guess it. If it is unavailable, ask for A's task link before creating B.
2. For repository work, call `list_projects` and select the matching project. Use a worktree when `isGitRepository` is true, unless the user explicitly requests the saved checkout. Use local for a non-Git project. For a message-only test or work without a repository, use a projectless task. Do not copy uncommitted work into a worktree without the user's request.
3. Call `create_thread` once, without `model` or `thinking`. Its initial prompt must identify A, define B's role, and carry the user's actual authorization to receive scoped delegations and return results. Keep this permission effective across the objective's handoffs, not just the initialization turn. Do not invent broader permissions. Tell B to return `WORKER_READY` locally, then wait for A's first handoff; do not send an initialization callback.
4. Obtain the returned `threadId` and `hostId`. A queued `clientThreadId` is not a usable thread ID. Wait for setup and resolve the ready task through the available thread tools; do not create a duplicate. Use a bounded `wait_threads` call to confirm initialization, and `read_thread` when needed to verify the ready task and workspace. Resolve an initialization error before dispatch.
5. Record the work ID, A/B thread IDs, host ID, B's workspace, latest handoff ID, and state in the task's existing work record. This is a logical pair, not an App-native parent/child relationship. Reuse it for the same objective and verify it again after a context loss or interruption. Do not use a global hard-coded Worker ID.
6. Send the first bounded handoff, then end A's turn. Follow any UI instructions actually supplied by the creation tool. If none are supplied, identify B with its verified task link; do not invent a directive.

Initialization prompt outline (replace placeholders with verified values):

> You are Desktop Worker B for Planner A, thread `<A_ID>`. The user authorized `<objective and scope>`, including scoped A delegations and one result callback per handoff to A. This permission remains in effect for this objective; each handoff must stay within it. Do not delegate to another worker. First read `<VERIFIED_ABSOLUTE_SKILL_PATH>` and follow its B rules. If you cannot read it, report the limitation here. Otherwise initialize by replying WORKER_READY here, then wait for the first handoff. If instructions conflict or a tool rejects the action, preserve the reason and stop. Do not use CLI or automation as a fallback.

Receiving `WORKER_READY` proves initialization only. A successful callback from the recorded B with the exact handoff ID is required to verify the return path. Do not infer delivery from `create_thread` or `send_message_to_thread` success alone.

## Dispatch and return

1. A calls `send_message_to_thread` with B's `threadId` and a `prompt` containing the bounded handoff. After the tool confirms dispatch, A ends its turn. A does not wait in a polling loop.
2. B performs the work in its Desktop turn. B calls `send_message_to_thread` once with A's `threadId` and the result in `prompt`, then ends its turn. B does not delegate to another worker.
3. When the callback starts A's next turn, A verifies its source thread and exact work and handoff IDs. If an ID differs, A checks B's recorded turn before accepting the result. A records the response count before review, then reviews actual changes and evidence. Close accepted work; otherwise apply the active review mode and progress check before sending another handoff.

## Review modes

The default mode starts with a five-response limit. The user may explicitly extend it or enable goal mode below. A owns a persistent counter for each work objective. Before the first dispatch, record `response_count: 0`, `response_limit: 5`, `counted_response_ids: []`, and `review_state: pending` alongside the A/B pair in the task's local work record. Use a small state file in the permitted workspace if no durable record exists. Do not keep the counter only in conversational recall.

- On each distinct B work response, increment and save the counter before reviewing. Count `ready_for_review`, `needs_decision`, and `blocked` responses. Count an incomplete or malformed work response too; it still consumed a round. Use the source message or turn ID to deduplicate. Exclude initialization `WORKER_READY`, duplicate delivery of the same response, and unrelated messages.
- In bounded mode, responses below the saved `response_limit` may lead to another scoped handoff if authorized. When the count reaches the limit, A performs the review once. If the complete objective passes, mark it accepted. If it does not pass, or evidence remains insufficient, set `review_state: awaiting_human` and stop. Do not dispatch beyond that limit, continue repairs in A, or launch a substitute worker.
- When the limit is reached without acceptance, tell the user briefly: `<response_count>/<response_limit> responses reviewed; acceptance is incomplete`, the remaining issues, and the decision needed. Do not automatically send an ACK or another task to B. B already ends its turn after its callback.
- Preserve the count across incremental handoffs, context loss, thread changes, and renamed work IDs for the same objective. Before any dispatch, read the saved count and review state. If the record is missing or inconsistent, recover it from the recorded turns; do not assume zero. If it cannot be recovered, stop for human decision.
- Only an explicit human decision may authorize another bounded batch. For N additional responses, keep `response_count` and `counted_response_ids`, set `response_limit = response_count + N`, record the decision, and set `review_state: pending`. For example, count 5 plus 3 authorized responses gives limit 8. If the number is unclear, ask before dispatch. Do not reset the counter or treat a status question as permission to continue. Independent objectives start their own counters.

This is an agent workflow limit, not an App-enforced token cap. Do not claim that it prevents every tool call or message at the platform level.

### Optional goal mode

Enable this mode only when the user explicitly asks to continue a specific objective without the five-response limit. Discussing goal mode or installing this skill does not enable it. Record `review_mode: goal`, `response_limit: null`, the user's authorization, the scope, and verifiable completion conditions in the existing work record. Preserve the response count for visibility and deduplication. Each handoff remains bounded.

Continue only while the agreed conditions remain unmet and the progress check above supports another attempt. Stop when the objective passes, progress stalls, a user decision or broader scope is required, the user cancels, or an agreed time or usage boundary is reached. Observe user-specified budgets with available measurements; do not claim an exact monetary cap when cost data is unavailable. Goal mode does not authorize new objectives, extra workers, automation, or bypassing permissions. It is a message-driven workflow, not the App's native goal feature or a platform-enforced budget.

## One bounded handoff

Establish the work ID, A/B pair, scope, authorization, output location, and relevant stop conditions once, during initialization or the first handoff. Later handoffs inherit that context. Repeat a field only when it changes or B cannot recover it. Do not paste this skill, generic prohibitions, prior conversation summaries, or repeated authorization statements into each message.

Keep A's incremental message to the handoff ID, requested change, and completion condition. Add specific files or evidence only when B needs them. Prefer a few direct lines; omit preambles and explanations of the workflow. For example:

> Handoff: layout-003
> Fix the narrow-width toolbar overlap in the assigned view.
> Done: verify the supported minimum width and return the changed files and result.

If the requested behavior can be misread, add one concrete input and expected result. For integration work, identify the product entry point and the required connection to its dependencies in the completion condition.

State a new permission or scope boundary explicitly when it matters; brevity does not expand authorization. If context is missing, supply only the missing facts. A's user-facing dispatch update should also be one short sentence.

Include the exact work ID and handoff ID in the callback. Use `ready_for_review` only when the assigned work and required validation are complete; otherwise use `needs_decision` or `blocked`. Include:

- actual files changed, their relative paths, and the reviewed artifact version (commit plus dirty-state evidence, or file hashes when needed);
- validation commands and results;
- unfinished items, limits, and any evidence not obtained.

Do not send ACKs. For duplicate messages, inspect the prior result before acting. Do not automatically resend after an ambiguous tool result. If a tool review rejects the callback, preserve the reason, stop, and report it.

For worktree delivery, establish the intended destination at the first handoff. B reports the worktree path, branch, artifact version, and whether integration is pending. A acceptance of worktree changes does not mean the original checkout is updated. If the objective includes integration, B performs it under existing Git authorization and A verifies the destination before closing the objective. If authorization is missing, report the accepted delivery and pending integration, then request only the missing authorization. Preserve unrelated staged and uncommitted changes; do not overwrite them or copy them between worktrees without user authorization.

## Evidence and loop efficiency

For integration delivery, B traces the product call path and confirms that its required dependencies are connected. Connections made only in tests do not prove that the product entry point is connected. Run the relevant build and checks after the final code change; if code changes again, rerun affected checks before reporting success.

Save the delivery and validation evidence in the permitted workspace before the callback. Include a decision-ready summary and evidence paths in that single return. For a failure, include the actual error and reproduction conditions. Do not send a completion-only message that requires A to ask for the result.

Keep source code, diffs, and diagnostic evidence available in their original form. Summarize repetitive logs only when useful, and retain a direct path to the complete output. A reads the summary first, then checks the necessary original changes and evidence. A must not accept a summary as proof, but should not repeat B's full exploration without a concrete reason.

If A and B obtain different validation results, first compare artifact versions, commands, and execution permissions. Classify a product defect only when the evidence supports it; do not use an environment mismatch alone to justify a code repair.

For a repair or evidence request, record the handoff ID and one concrete reason in the existing work record. A new requirement needs the user's scope decision and does not reset the objective's response count.

Evaluate efficiency only when the user requests it. Use existing counts, reasons, and available time or usage measurements across the complete objective. State measurement limits; do not infer savings from message length or equate token counts with billing.

## Execution boundaries

- Do not start a CLI runner, subagent, automation, or alternate worker as a fallback. Do not read or send worker-session internals.
- A and B must not edit the same file concurrently. Preserve unrelated dirty or staged work.
- B performs explicitly authorized Git writes. A reviews scope and evidence. A commit authorization does not authorize push. Keep commits scoped, preserve unrelated changes, and do not repeat a completed Git operation.
- Use `wait_threads` only for bounded status confirmation or diagnosis, not as a polling substitute for the single return.
- Report automatic wake-up only when a callback actually starts A's next turn; successful dispatch alone does not prove it. Do not claim exactly-once delivery, zero-token behavior, or cross-restart guarantees. If B is interrupted or the callback fails, A can verify with `read_thread`.
