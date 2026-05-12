---
name: prd-decompose
description: Use when a PRD needs to be broken down into
  work units with dependency DAG, complexity tiers, and
  file-overlap analysis before implementation begins.
---

# PRD Decompose

將 PRD 拆解為可獨立實作、可獨立測試的工作單元，產出 DAG + PROGRESS.md。

## Input

PRD 文件路徑（預設 `docs/product/prd.md`）。

## Step 1: 掃描現狀

1. 讀取 PRD 全文
2. 列出所有 User Stories 及其 Acceptance Criteria
3. 掃描現有程式碼（`frontend/src/`, `backend/app/`）
4. 比對每個 Story 的 AC，判斷哪些已實作、哪些未實作
5. 只對「未實作」的 Stories 進行分解

## Step 2: Story → Work Units

每個未實作的 Story 拆為 1~3 個 work units。拆分原則：

### 何時拆分

- Story 同時需要前端和後端改動 → 拆為 `backend-xxx` + `frontend-xxx`
- Story 的後端涉及新 DB schema + 新 API → 拆為 `schema-xxx` + `api-xxx`
- Story 的前端涉及新頁面 + 新元件 → 保持一個 unit（除非頁面間無關聯）

### 何時不拆

- Story 只改前端或只改後端 → 一個 unit
- 拆完後 unit 之間有大量共用檔案 → 不拆，保持一個 unit

### Unit 欄位（必填）

每個 unit 必須包含以下欄位：

```markdown
- **id**: kebab-case (e.g., `auth-backend-api`)
- **story**: 對應的 Story 編號 (e.g., Story 6)
- **name**: 人類可讀名稱
- **scope**: 具體要做什麼（2-3 句話）
- **deps**: 依賴的其他 unit id（沒有則填 none）
- **acceptance**: 從 Story AC 提取，只列本 unit 負責的部分
- **tier**: 1 | 2 | 3（見下方定義）
- **touches**: 預估會新增或修改的檔案路徑
```

## Step 3: 定義 Complexity Tier

根據以下規則判斷每個 unit 的 tier：

| Tier | 條件 | 範例 |
|------|------|------|
| **1** | 單一檔案或同模組內 2-3 個檔案，邏輯簡單，無外部依賴 | 新增一個 utility function、加一個靜態頁面 |
| **2** | 跨 3-5 個檔案，涉及前後端串接或新 API endpoint | 新增 API + 前端呼叫、新增元件 + service layer |
| **3** | 涉及新架構概念（DB schema、auth、第三方 OAuth）、安全敏感 | 會員系統、OAuth 整合、圖片上傳 |

## Step 4: 建立 DAG

### 依賴規則

依賴 **只** 標記在以下情況：
- Unit B 的程式碼直接 import Unit A 產出的模組
- Unit B 的 API 需要 Unit A 建立的 DB table
- Unit B 的前端元件需要 Unit A 的 API endpoint

**不算依賴的情況：**
- 只是邏輯上「先做 A 比較合理」→ 不標
- A 和 B 改同一個檔案但改不同部分 → 標為 file overlap，不是 dependency

### 分層

```
Layer 0: 所有 deps 為 none 的 units
Layer 1: 所有 deps 只指向 Layer 0 的 units
Layer 2: 所有 deps 只指向 Layer 0 或 Layer 1 的 units
...
```

### File Overlap 分析

對每個 layer 內的 units，比對 `touches` 欄位：
- 有重疊 → 標記，Phase 3 必須序列執行
- 無重疊 → 可平行執行

## Step 5: 產出 PROGRESS.md

將結果寫入 `docs/PROGRESS.md`，格式如下：

```markdown
# Development Progress

Generated: {日期}
PRD: docs/product/prd.md

## DAG

Layer 0: unit-a, unit-b [parallel]
Layer 1: unit-c (deps: unit-a) [sequential - overlaps with unit-d]
Layer 1: unit-d (deps: unit-b) [sequential - overlaps with unit-c]
Layer 2: unit-e (deps: unit-c, unit-d)

## Units

### unit-a
- **Story**: Story N
- **Name**: ...
- **Scope**: ...
- **Deps**: none
- **Tier**: 1
- **Touches**: `backend/app/models/xxx.py`, `backend/tests/test_xxx.py`
- **Acceptance**:
  - [ ] AC 1
  - [ ] AC 2
- **Status**: Pending

(repeat for each unit)

## Summary

| Status | Count |
|--------|-------|
| Pending | N |
| In Progress | 0 |
| Completed | 0 |
| Failed | 0 |
| Total | N |
```

## 禁止事項

- 不要把一個 Story 拆成超過 3 個 units
- 不要建立只有測試的 unit（測試跟實作放一起）
- 不要猜測 PRD 沒寫的需求
- 不要跳過 file overlap 分析
- 不要在這個階段寫任何程式碼
