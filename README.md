# Desktop Thread Worker

A lightweight Planner / Worker skill for solo developers using Codex Desktop.

No extra packages, CLI runner, background service, or separate API key. The skill uses native Codex thread tools and your existing Codex access. Normal account usage still applies.

## How it works

You set the scope → Planner A dispatches → persistent Worker B implements and verifies → A reviews the evidence.

- Reuse one A/B pair for the same objective.
- Send only the task changes after the first handoff.
- Return the result once. Do not poll by default.
- Stop when the agreed conditions pass. Avoid speculative architecture and moving acceptance criteria.
- Default to five Worker responses per objective. Ask the user when the limit is reached.
- Enable goal mode only on explicit request. Remove the response limit, but retain progress checks, scope boundaries, and user-defined stopping conditions.

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

## Limits and status

This is an experimental instruction-based workflow, not a runtime scheduler. Response limits are not platform-enforced spending caps. Delivery, wake-up, and recovery depend on the available Desktop tools. Recent rule changes have passed skill-format validation, but have not all been exercised in end-to-end tasks.

Short handoffs, fewer duplicate reads, and fewer empty round trips are intended to reduce wasted context. No measured token-saving percentage is claimed.

Additional batches require explicit user approval. Keep the cumulative count and set the new limit to the current count plus the authorized additional responses. Worktree acceptance and integration into the destination checkout are separate completion checks.

## 繁體中文摘要

給單人開發者的輕量 Codex Desktop A/B 協作 skill。你決定目標與範圍，A 規劃及驗收，固定 B 實作與驗證。

不需額外套件或 API key，但需要 Desktop 提供原生 thread 工具，並使用既有帳號用量。採用精簡交接、單次回傳、KISS／YAGNI 與明確停止條件。預設五次回覆上限；只有使用者明確啟用，才使用無固定次數上限的目標模式。

## License

MIT. See LICENSE.