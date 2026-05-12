---
name: auto-dev
description: PRD-driven autonomous dev loop.
  Use when a PRD exists and full implementation
  should proceed with zero human intervention.
---

# Auto Dev

PRD 驅動的全自主開發調度器。
本 skill 只負責排程與銜接，規則由被引用的 skill 定義。

## When to Use

- 有明確的 PRD / RFC 文件需要實作
- 需要全自主、零人類介入的開發流程

**When NOT to use:**
- 單檔小修改 — 直接做
- 沒有需求文件 — 先寫 PRD

## Pipeline

Phase 0 → 1 → 2 → 3 → 4，Phase 3 per-unit loop。

## Phase 0: Initialize

1. 讀取 PRD 文件
2. 讀取 `docs/PROGRESS.md`（若存在，恢復進度）
3. 掃描程式碼，識別已完成部分與技術棧

## Phase 1: Decompose

**REQUIRED:** Invoke `prd-decompose`（分解規則 + DAG + file overlap 分析）
**REQUIRED:** Invoke `everything-claude-code:ralphinho-rfc-pipeline`（merge queue + recovery 規則）

產出 work units + DAG layers + complexity tiers + PROGRESS.md。

## Phase 2: Write PROGRESS.md

記錄 DAG 結構與每個 unit 狀態（Pending / In Progress / Completed / Failed）。
這是跨迭代的記憶橋樑。

## Phase 3: Execute (Per Unit, by DAG layer)

**REQUIRED:** Invoke `superpowers:subagent-driven-development`
- Subagents **REQUIRED:** use `superpowers:test-driven-development`

同 layer 內無檔案重疊的 units 可平行 dispatch。

Merge 依 `ralphinho-rfc-pipeline` 的 merge queue rules。
Evicted unit 最多重試 3 次。

### Per-Unit Commit Loop

每個 unit 完成後，**必須**依序執行以下步驟才能進入下一個 unit：

```
Unit 實作完成（TDD green）
  ↓
① Code Review — 使用 code-reviewer agent
  - 修復所有 CRITICAL / HIGH issues
  - MEDIUM issues 盡量修復
  ↓
② Verify — Invoke `superpowers:verification-before-completion`
  - 確認測試通過
  - 確認無 lint / type 錯誤
  - 確認功能符合該 unit 的需求描述
  - 若為 UI 相關 unit（前端元件、頁面、樣式、RWD 等）：
    Invoke `everything-claude-code:e2e` — 用 Playwright 截圖，視覺確認畫面正確
    若畫面不符預期 → 修復後重新截圖驗證
  ↓
③ Write Dev Log — 在專案 `docs/dev-log/` 目錄建立開發紀錄
  - 檔名: `{YYYY-MM-DD}-{unit-id}.md`
  - 內容: 變更摘要、修改的檔案、測試結果、截圖（若有）、備註
  - 此檔案隨 commit 一起提交
  ↓
④ Commit — 依 git-workflow 規範提交
  - message 格式: `<type>: <unit 描述>`
  - body 包含 unit ID 與 DAG layer 資訊
  - 範例: `feat: 實作商品 CRUD API (unit-3, layer-2)`
  ↓
⑤ 更新 PROGRESS.md — 標記該 unit 為 Completed，記錄 commit hash
```

### Per-Layer Checkpoint

當一個 DAG layer 的所有 units 都完成後：

1. 執行**跨 unit 整合測試**（確認同 layer 的 units 之間無衝突）
2. 若整合測試失敗 → 修復後追加 `fix:` commit
3. 在 PROGRESS.md 記錄 layer 完成狀態

### Commit 規則

- **禁止**累積多個 units 才一次 commit
- **禁止**跳過 review 或 verify 直接 commit
- 每個 commit 應該是**可獨立運行**的狀態（tests pass, no build errors）
- 若 unit 過大，可拆成多個子 commit（如 `test:` + `feat:`），但每個子 commit 都必須通過 verify

## Phase 4: Final Verification

**REQUIRED:** Invoke `superpowers:verification-before-completion`

逐條比對 PRD 需求。

## Failure Recovery

- 連續 2 次迭代無進展 → 凍結，縮小 scope
- 同一 unit 失敗 3 次 → 標記 Failed，跳過
- Context 接近上限 → 寫入 PROGRESS.md，結束本 session

## Red Flags

- 想跳過 invoke 某個 skill — 不行，invoke 它
- PROGRESS.md 沒更新就進入下一個 unit — 不行
- 想等多個 units 完成再一起 commit — 不行，每個 unit 完成就 commit
- 想跳過 code review 或 verify 直接 commit — 不行，三步都必須走完
