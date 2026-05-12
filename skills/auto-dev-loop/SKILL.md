---
name: auto-dev-loop
description: Invoke skill `auto-dev` with PROGRESS.md-based resume logic.
  Use when continuing an autonomous dev loop across sessions.
---

# Auto Dev Loop

Invoke skill `auto-dev`。

讀取 docs/PROGRESS.md 判斷目前階段：

- 若不存在 → 執行 Phase 0-2（初始化 + 分解 + 寫 PROGRESS.md）
- 若存在且有 Pending units → 執行 Phase 3（按 DAG layer 處理下一批 units）
- 若所有 units 都 Completed 或 Failed → 執行 Phase 4（最終驗證）

每次迭代結束前必須更新 PROGRESS.md。
