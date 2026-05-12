---
name: task-dev-loop
description: Invoke skill `task-dev` with TASK_PROGRESS.md-based resume logic.
  Use when continuing a TickTick-driven dev loop across sessions.
---

# Task Dev Loop

Invoke skill `task-dev`。

讀取 docs/TASK_PROGRESS.md 判斷目前階段：

- 若不存在 → 詢問 TickTick 專案，執行 Phase 0-1（初始化 + 寫 TASK_PROGRESS.md）
- 若存在且有 Pending tasks → 執行 Phase 2（處理下一個 task）
- 若所有 tasks 都 Completed 或 Skipped → 執行 Phase 3（最終驗證）

每次迭代結束前必須更新 TASK_PROGRESS.md。
完成 task 後必須同步更新 TickTick。
