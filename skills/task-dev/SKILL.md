---
name: task-dev
description: TickTick-driven dev loop for small features.
  Reads tasks from a TickTick project, processes them
  one by one with plan/TDD/review/verify/commit,
  then marks completed in TickTick.
---

# Task Dev

從 TickTick 專案讀取待辦清單，逐項實作小功能。
每個 task 走完整的 Plan → TDD → Review → Verify → Commit 流程。

## When to Use

- 有一組小功能待辦在 TickTick 專案中
- 每個功能是獨立的（1-5 個檔案）
- 不需要 PRD 或 DAG 分解

**When NOT to use:**
- 大型功能、跨模組架構變更 → 用 `auto-dev` + PRD
- 單一小修改 → 直接做

## Input

TickTick project ID 或專案名稱。

若未指定，詢問使用者。可用 `ticktick projects list` 列出所有專案。

## Phase 0: Initialize

1. 執行 `ticktick tasks list <projectId>` 取得待辦清單
2. 過濾未完成的任務
3. **只處理有 `AI` 標籤的 task**，沒有 `AI` 標籤的跳過（使用 `--format json` 檢查 `tags` 欄位）
4. 若篩選後無任務，告知使用者「沒有標記 AI 標籤的待辦」並結束
5. 對有 description 的任務，執行 `ticktick tasks get <projectId> <taskId>` 讀取細節
6. 讀取 `docs/TASK_PROGRESS.md`（若存在，恢復進度）
7. 掃描程式碼，識別技術棧

### 任務排序

依以下優先順序處理：
1. Priority: high → medium → low → none
2. Due date: 近的先做
3. 清單順序

## Phase 1: Write TASK_PROGRESS.md

建立 `docs/TASK_PROGRESS.md` 記錄進度，這是跨 session 的記憶橋樑。

```markdown
# Task Progress

Generated: {日期}
Source: TickTick project "{專案名}" ({projectId})

## Tasks

### {taskId} — {title}
- **Priority**: high | medium | low | none
- **Tags**: ...
- **Due**: YYYY-MM-DD
- **Description**: (從 TickTick 讀取)
- **Steps**: (Phase 2 填入)
- **Status**: Pending
- **Commit**: (完成後填入 hash)

(repeat for each task)

## Summary

| Status | Count |
|--------|-------|
| Pending | N |
| In Progress | 0 |
| Completed | 0 |
| Skipped | 0 |
| Total | N |
```

## Phase 2: Execute (Per Task)

找到第一個 Pending 的 task，執行以下流程：

### Step 1: Plan

**REQUIRED:** Invoke `superpowers:writing-plans`

1. 在 TASK_PROGRESS.md 將該 task 標記為 `In Progress`
2. 將 TickTick task 的 title + description 作為需求輸入
3. 依 `superpowers:writing-plans` 流程產出實作計畫
4. 用 TodoWrite 將計畫拆為 3-8 個可驗證的步驟
5. 將步驟寫入 TASK_PROGRESS.md 的 Steps 欄位

### Step 2: TDD

**REQUIRED:** 使用 `superpowers:test-driven-development` 的原則

- RED: 先寫測試，確認失敗
- GREEN: 寫最小實作讓測試通過
- REFACTOR: 清理程式碼

### Step 3: Code Review

使用 code-reviewer agent：
- 修復所有 CRITICAL / HIGH issues
- MEDIUM issues 盡量修復

### Step 4: Verify

**REQUIRED:** Invoke `superpowers:verification-before-completion`

- 確認所有測試通過
- 確認無 lint / type 錯誤
- 確認功能符合 task 描述
- 若為 UI 相關 task（前端元件、頁面、樣式、RWD 等）：
  **REQUIRED:** Invoke `everything-claude-code:e2e`
  - 用 Playwright 開啟對應頁面並截圖
  - 用 Read tool 讀取截圖，視覺確認畫面正確
  - 若畫面不符預期 → 修復後重新截圖驗證

### Step 5: Write Dev Log

**REQUIRED:** 在 commit 前，於專案 `docs/dev-log/` 目錄建立開發紀錄。

檔名: `{YYYY-MM-DD}-{taskId}.md`

內容：
```markdown
# {task title}

- **Date**: {YYYY-MM-DD}
- **Task**: {taskId}
- **Commit**: (Step 6 填入)

## 變更摘要
- 簡述做了什麼（2-3 句）

## 修改的檔案
- `path/to/file.ts` — 變更說明

## 測試
- 測試結果（pass/fail 數量）

## 備註
（任何值得記錄的決策、取捨、或待追蹤事項）
```

此檔案隨 commit 一起提交。

### Step 6: Commit

依 git-workflow 規範提交：
- message 格式: `<type>: <task 描述>`
- body 包含 TickTick task ID
- 範例: `feat: 新增重量篩選 filter (ticktick: 69b94b62)`

### Step 7: Update Progress

1. 更新 TASK_PROGRESS.md — 標記為 Completed，記錄 commit hash
2. **不要**在 TickTick 標記完成 — 由使用者自行確認後手動完成
3. 更新 Summary 計數

### 然後

繼續處理下一個 Pending task，重複 Phase 2。

## Phase 3: Final Verification (當所有 task 完成)

1. 執行完整測試套件
2. 確認 build 通過
3. 回顧 TASK_PROGRESS.md，確認所有 Completed 的功能正常運作

## Failure Recovery

- 單個 task 失敗 2 次 → 標記 `Skipped`，在 TickTick task 加 comment 說明原因，跳到下一個
- Context 接近上限 → 更新 TASK_PROGRESS.md，結束 session，下次從未完成項繼續

## Commit 規則

- **禁止**累積多個 tasks 才一次 commit
- **禁止**跳過 review 或 verify 直接 commit
- 每個 commit 應該是可獨立運行的狀態（tests pass, no build errors）

## Red Flags

- 想跳過 TDD — 不行
- 想跳過 code review 或 verify — 不行
- TASK_PROGRESS.md 沒更新就進入下一個 task — 不行
- 想等多個 tasks 完成再一起 commit — 不行
