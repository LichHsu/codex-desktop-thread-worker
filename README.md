# Desktop Thread Worker

Overview with 繁體中文摘要 | [English](README.en.md)

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

## 使用範例：修正警報清單遺漏

本範例改寫自真實開發任務，專案細節已匿名化。以下訊息用於說明操作流程，並非對話逐字紀錄或已驗證的完成報告。

作者目前使用的模型配置（2026-09-10）：Planner A 使用 Astra，思考強度為 `low`；Worker B 使用 Luna，思考強度為 `xhigh`。這是作者的使用配置，並非 skill 的必要條件。

**1. 給 Planner A 明確範圍的任務**

> 使用 $desktop-thread-worker，沿用我現有的 Worker。
>
> 範圍：修正警報清單在警報發生時間超過查詢區間後，隱藏尚未解除警報的問題。
>
> 完成條件：
>
> - 較早發生但尚未解除的警報仍可見。
> - 已解除的警報不列入。
> - 已確認的警報依既有領域規則處理。
> - 若清單有顯示上限，回報總數及是否截斷。
> - 保持歷史查詢原有語意。
>
> 保留無關變更。不要 commit 或 push。

**2. A 向 B 發出一次交接**

> 交接代號：EXAMPLE-001
>
> 在約定範圍內修正未解除警報查詢。變更狀態篩選前，先確認領域規則。回報變更檔案、驗證結果及剩餘限制。

**3. B 實作並回傳證據**

B 完成範圍內的修改、執行相關檢查，並回報通過、失敗及尚未驗證的項目。

**4. A 驗收交付**

A 依約定條件查核實際變更與證據。若有未達標條件，A 發出指出具體缺口的修正要求；所有條件通過後即結案。

使用者隨時可以開啟 B，查看進度或直接下達指示。

## Extension points

A/B is the default minimum configuration. The same thread messaging pattern can extend to C, D, or other independent Desktop tasks for research, implementation, or review. The user decides whether to add roles. Each added role needs a clear scope, a reporting target, and file ownership to prevent duplicate work or concurrent edits to the same files.

The current skill defines the A/B workflow. Multi-Worker coordination is an extension possibility, not a complete supported or validated mode. Start with A/B and add roles only when the task justifies their coordination and usage costs.

## Limits and status

This is an experimental instruction-based workflow, not a runtime scheduler. Response limits are not platform-enforced spending caps. Delivery, wake-up, and recovery depend on the available Desktop tools. Recent rule changes have passed skill-format validation, but have not all been exercised in end-to-end tasks.

Short handoffs, fewer duplicate reads, and fewer empty round trips are intended to reduce wasted context. No measured token-saving percentage is claimed.

Additional batches require explicit user approval. Keep the cumulative count and set the new limit to the current count plus the authorized additional responses. Worktree acceptance and integration into the destination checkout are separate completion checks.

## 繁體中文摘要

給單人維護單一軟體、token 預算有限的開發者使用。你決定目標與範圍，A 規劃及驗收，固定 B 實作與驗證。透過精簡交接減少手動轉述與重複背景，並以 KISS／YAGNI 約束 Planner：退回須有具體未達標證據，達標即結案。

A 與 B 都是可直接開啟的 Desktop 對話。使用者能查看過程、直接介入、指定更換 Worker，也能透過 Codex UI 調整 Worker 的模型與思考強度；可選項目依帳號及模型而定，不需修改 skill 或另外設定 API。這是使用者的控制權，並非 Worker 自行切換，也不表示變更會套用到正在執行的回合。不同工具的 subagent 可見性與控制方式不同，本專案不宣稱它們一律不可見或不可控制。

不需額外套件或 API key，但須有原生 thread 工具，並使用既有帳號用量。預設五次回覆上限；使用者可明確授權追加次數或啟用目標模式。更換 Worker 須交接狀態、處理進行中的修改，並保留同一目標的計數與授權邊界。

A/B 是最小配置，訊息架構可按需延伸 C／D。新增角色須明定範圍、回報對象與檔案責任；目前 skill 定義的是 A/B，多 Worker 協作尚非完整支援或已驗證的模式。

## License

MIT. See LICENSE.
