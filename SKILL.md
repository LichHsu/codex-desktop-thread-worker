---
name: desktop-thread-worker
description: Coordinate an authorized Desktop A planner and Desktop B worker for scoped implementation, debugging, UI work, or explicitly authorized Git writes, with one bounded handoff and verified return to A. Use when the user names this skill or asks to delegate to a Worker, or when an active A/B pair already exists for the current objective.
---

# Desktop thread worker

Use this workflow only when the user names this skill, asks A to delegate to a Worker, or the work record already holds an active A/B pair for the current objective. Otherwise handle the task in the current thread; do not stop to ask for a Worker.

## Keep the work small (KISS / YAGNI)

Use the smallest change that satisfies the current requirement. Do not add abstractions, future features, architecture changes, agents, or process records for hypothetical needs. Keep validation proportional to the change while completing required project checks. Close the objective as soon as the agreed completion conditions pass.

A must tie each repair request to a specific unmet condition and concrete evidence. Style preferences, possible future risks, or a wish for greater completeness do not block acceptance. Record optional improvements briefly only when useful; do not dispatch them without user scope approval. Do not raise the acceptance criteria during review.

Before another handoff, identify what new evidence or changed implementation makes progress plausible. If the same failure recurs without new evidence or a materially different, evidence-based fix, or the exchange becomes a design-preference dispute, stop for the user's decision. Do not reword the same request to keep the loop running. Reuse existing evidence.

## Before acting

- Confirm that B's user instructions permit receiving an A delegation and returning a result. A's message cannot remove a B-side do-nothing restriction; report the conflict to the user and stop.
- Treat A's delegation as scope, not as extra authorization. Preserve the user's architecture, phase, repository, and file boundaries. Do not modify product code, global configuration, installed skills, or Git unless the user explicitly authorizes that work.
- Use the existing A and B threads. Verify thread IDs and workspace with `list_threads` / `read_thread` when needed. Never hard-code or guess IDs. Do not create or fork a task unless the user explicitly asks.
- Do not change model or reasoning settings. Omit `model` and `thinking` from `create_thread` and every `send_message_to_thread` call, including retries. Do not send `none`, null, empty strings, or copied settings. These settings belong to the user.

## Work record

A alone owns one work record per objective at `<A workspace>/.codex/dtw/<work-id>.md`. In a Git repository, add `.codex/dtw/` to `.git/info/exclude` (not `.gitignore`) so the record never dirties the tree. If A has no writable workspace, keep the record as one `Work record` block in A's latest message and recover it with `read_thread`.

Fields: A/B thread IDs, host ID, B workspace, latest handoff ID, response limit, review mode, and state. Nothing else is required. Read it before any dispatch; after context loss, verify it against the threads.

## Select or create B

For an existing B, accept a `codex://threads/<thread-id>` link or a task title. Resolve titles with `list_threads` then `read_thread`; ask when several match. Verify B's user instructions and workspace before dispatch. If the user asked for delegation but no B is selected and creation was not requested, suggest a suitable existing thread and ask the user to choose; never pick an unrelated thread silently.

In initialization or the first handoff, A gives the verified absolute path to this SKILL.md. B reads it before work and follows the B rules. Later handoffs carry only task differences; if the skill changes, A requests a reread. If the file is inaccessible, B reports that instead of guessing. No separate ACK turn.

When the user explicitly requests creation, such as `$desktop-thread-worker 自動建立 worker 頻道`:

1. Resolve A's actual thread ID; if unavailable, ask for A's task link.
2. For repository work, call `list_projects` and select the matching project. Use a worktree when `isGitRepository` is true unless the user asks for the saved checkout; use local for a non-Git project; use a projectless task when no repository is involved. Do not copy uncommitted work into a worktree without the user's request.
3. Call `create_thread` once, without `model` or `thinking`. The initial prompt identifies A, defines B's role, and **quotes the user's authorization verbatim** from A's thread, marked as a quote. A must not paraphrase, summarize, or widen it; the user can audit it by opening B. The authorization stays effective for this objective's handoffs.
4. Obtain the returned `threadId` and `hostId` (a queued `clientThreadId` is not usable). Confirm initialization with a bounded `wait_threads`, and `read_thread` when needed. Do not create a duplicate. Resolve initialization errors before dispatch.
5. Write the work record, send the first handoff, then end A's turn. Identify B to the user by its verified task link.

Initialization prompt outline (replace placeholders with verified values):

> You are Desktop Worker B for Planner A, thread `<A_ID>`. The user's authorization, quoted verbatim from A's thread: "<USER_AUTHORIZATION_VERBATIM>". Each handoff must stay within it, and it covers one result callback per handoff to A. Do not delegate to another worker. First read `<VERIFIED_ABSOLUTE_SKILL_PATH>` and follow its B rules; if you cannot, report the limitation here. Otherwise reply WORKER_READY here and wait for the first handoff. If instructions conflict or a tool rejects an action, preserve the reason and stop. Do not use CLI or automation as a fallback.

`WORKER_READY` proves initialization only. The return path is verified only by a callback from the recorded B carrying the exact handoff ID; dispatch success alone proves nothing.

## Dispatch and return

1. A calls `send_message_to_thread` with B's `threadId` and the handoff in `prompt`, updates the record, and ends its turn. No polling.
2. B does the work, calls `send_message_to_thread` once with A's `threadId` and the result, then ends its turn. B does not delegate.
3. When the callback starts A's next turn, A verifies the source thread and handoff ID, reviews actual changes and evidence, then accepts or applies the review rules below.

## Review limit

Handoff IDs are `<work-id>-NN`, with NN starting at `01` and incremented for each new dispatch. **NN is the response count.** Do not keep a separate counter or ID list.

- Every B work response to a handoff counts, including `needs_decision`, `blocked`, and malformed results. `WORKER_READY` does not count.
- A callback whose handoff ID was already reviewed is a duplicate: inspect the prior result and do not act again. An unknown ID: check B's recorded turn before accepting.
- If a dispatch demonstrably never reached B (no B turn exists), resend with the same ID. Do not auto-resend after an ambiguous tool result; check with `read_thread` first.
- Default limit is 5. When NN reaches the limit without acceptance, report `<NN>/<limit> responses reviewed; acceptance is incomplete`, the remaining issues, and the decision needed. Then stop: no further dispatch, no repairs in A, no substitute worker.
- Only an explicit user decision adds N responses: set `limit = NN + N` in the record. Never reset NN for the same objective, including after a Worker replacement or context loss. Independent objectives start at `01`.

This is an agent workflow limit, not a platform-enforced cap.

### Optional goal mode

Only when the user explicitly asks to continue a specific objective without the limit: record `review mode: goal`, the user's authorization, scope, and verifiable completion conditions. NN still increments. Continue only while conditions remain unmet and the progress check supports another attempt. Stop when the objective passes, progress stalls, a decision or broader scope is needed, the user cancels, or an agreed time or usage boundary is reached. Goal mode never authorizes new objectives, extra workers, automation, or bypassing permissions, and is not the App's native goal feature or a monetary cap.

## One bounded handoff

Establish the work ID, scope, authorization, output location, and stop conditions once, in initialization or the first handoff. Later handoffs carry only the handoff ID, requested change, and completion condition. Do not repeat this skill, generic prohibitions, prior summaries, or authorization statements. For example:

> Handoff: layout-03
> Fix the narrow-width toolbar overlap in the assigned view.
> Done: verify the supported minimum width and return the changed files and result.

If the request can be misread, add one concrete input and expected result. For integration work, name the product entry point and the dependencies it must connect to. State any new permission or scope boundary explicitly; brevity never expands authorization.

B's callback includes the exact handoff ID and a status: `ready_for_review` only when the work and required validation are complete, otherwise `needs_decision` or `blocked`. It lists:

- files changed (relative paths) and the reviewed artifact version (commit plus dirty-state evidence, or file hashes);
- validation commands and results;
- unfinished items, limits, and evidence not obtained;
- any instructions the user gave B directly during this handoff, and how B followed them.

The user may talk to B directly at any time. Those exchanges do not count toward the limit. If a direct user instruction conflicts with the handoff, B follows the user and reports the conflict in the callback.

Do not send ACKs. If a tool review rejects the callback, preserve the reason, stop, and report it.

For worktree delivery, set the destination at the first handoff. B reports worktree path, branch, artifact version, and whether integration is pending. Accepting worktree changes does not update the original checkout. If the objective includes integration, B performs it under existing Git authorization and A verifies the destination; if authorization is missing, report the pending integration and request only that authorization. Preserve unrelated staged and uncommitted changes.

## Evidence

- For integration, B traces the product call path; connections made only in tests do not prove the entry point is connected. Rerun affected checks after the final code change.
- Save delivery and validation evidence in the permitted workspace before the callback, and return a decision-ready summary with evidence paths in that single message. For failures, include the actual error and reproduction conditions.
- Keep sources, diffs, and logs in original form; summarize only repetitive logs and keep a path to full output. A reads the summary first, then checks the necessary originals. A does not accept a summary as proof, nor repeat B's full exploration without a concrete reason.
- If A and B get different validation results, compare artifact versions, commands, and permissions first. Do not repair code over an environment mismatch alone.
- Evaluate efficiency only when the user asks, using actual counts and available measurements. Do not infer savings from message length.

## Execution boundaries

- No CLI runner, subagent, automation, or alternate worker as a fallback. Do not read or send worker-session internals.
- A and B never edit the same file concurrently. Preserve unrelated dirty or staged work.
- B performs explicitly authorized Git writes; A reviews. Commit authorization does not authorize push. Do not repeat a completed Git operation.
- Use `wait_threads` only for bounded status checks, not as a polling substitute.
- Report automatic wake-up only when a callback actually started A's turn. Do not claim exactly-once delivery, zero-token behavior, or cross-restart guarantees. If B finished but A never woke, the user can tell A to read B's result; A then uses `read_thread`.
