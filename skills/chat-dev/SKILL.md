---
name: chat-dev
description: Interactive dev loop for small features.
  Discusses the task with the user, clarifies requirements,
  then executes with plan/TDD/review/verify/commit.
---

# Chat Dev

透過對話釐清需求，然後逐項實作小功能。
每個 task 走完整的 Plan → TDD → Review → Verify → Commit 流程。

## When to Use

- 使用者有一個小功能想做，但還沒有明確的 task 清單
- 需要先討論、釐清需求再動手
- 每個功能是獨立的（1-5 個檔案）
- 不需要 PRD 或 DAG 分解

**When NOT to use:**
- 大型功能、跨模組架構變更 → 用 `auto-dev` + PRD
- 已有 TickTick task 清單 → 用 `task-dev`
- 單一小修改 → 直接做

## Input

使用者口頭描述的功能需求。

## Phase 0: Clarify Requirements

**REQUIRED:** 與使用者對話釐清需求，不要急著動手。

1. 聽取使用者描述的功能需求
2. 提出釐清問題，確認以下事項：
   - **目標**: 這個功能要解決什麼問題？
   - **範圍**: 影響哪些頁面/模組？
   - **行為**: 預期的使用者操作流程？
   - **邊界**: 有什麼不需要做的？
   - **驗收標準**: 怎樣算做完？
3. 將釐清後的需求整理為 1-N 個獨立的 task
4. 列出 task 清單，請使用者確認後再開始

### 釐清原則

- 問題要具體，不要問「還有什麼要注意的？」這種空泛問題
- 如果需求已經很明確，不要為了問而問，直接進入下一步
- 一次最多問 3-5 個問題，避免疲勞轟炸
- 可以提出建議方案讓使用者選擇，而不是全部開放式提問

## Phase 1: Write TASK_PROGRESS.md

建立 `docs/TASK_PROGRESS.md` 記錄進度，這是跨 session 的記憶橋樑。

```markdown
# Task Progress

Generated: {日期}
Source: Chat discussion with user

## Tasks

### CHAT-001 — {title}
- **Priority**: high | medium | low | none
- **Description**: (從對話中整理)
- **Acceptance Criteria**: (從對話中整理)
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
2. 將 task 的 title + description + acceptance criteria 作為需求輸入
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

檔名: `{YYYY-MM-DD}-CHAT-{NNN}.md`

內容：
```markdown
# {task title}

- **Date**: {YYYY-MM-DD}
- **Task**: CHAT-{NNN}
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
- body 包含 task ID
- 範例: `feat: 新增重量篩選 filter (chat-dev: CHAT-001)`

### Step 7: Update Progress

1. 更新 TASK_PROGRESS.md — 標記為 Completed，記錄 commit hash
2. 更新 Summary 計數

### 然後

繼續處理下一個 Pending task，重複 Phase 2。

## Phase 3: Final Verification (當所有 task 完成)

1. 執行完整測試套件
2. 確認 build 通過
3. 回顧 TASK_PROGRESS.md，確認所有 Completed 的功能正常運作

## Failure Recovery

- 單個 task 失敗 2 次 → 標記 `Skipped`，記錄原因，跳到下一個
- Context 接近上限 → 更新 TASK_PROGRESS.md，結束 session，下次從未完成項繼續

## Commit 規則

- **禁止**累積多個 tasks 才一次 commit
- **禁止**跳過 review 或 verify 直接 commit
- 每個 commit 應該是可獨立運行的狀態（tests pass, no build errors）

## Red Flags

- 想跳過需求釐清直接開幹 — 不行
- 想跳過 TDD — 不行
- 想跳過 code review 或 verify — 不行
- TASK_PROGRESS.md 沒更新就進入下一個 task — 不行
- 想等多個 tasks 完成再一起 commit — 不行
