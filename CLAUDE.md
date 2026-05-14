# CLAUDE.md — Where is the Garbage Truck?

本專案採用 SDD（Spec-Driven Development）工作流程。

## 目錄結構

```
20260513_Where_is_garbage_truck/
├── current/          ← 進行中的 feature / fix
│   ├── feat-001/
│   ├── feat-002/
│   └── fix-001/
├── archived/         ← 已完成或作廢的 feature / fix
│   └── feat-001/
├── CLAUDE.md
├── README.md
├── LOG.md
├── index.html        ← 主程式
└── data.js           ← 垃圾車站點資料
```

## Feature 規範

每個 `feat-NNN/` 目錄包含三份文件，**依序撰寫**：

### SPEC.md — 外部規格
- 從使用者角度描述這個功能做什麼
- 包含：使用情境、UI 行為、輸入/輸出、邊界條件
- **不涉及實作細節**

### PLAN.md — 實作策略
- 技術層面的實作方式
- 包含：涉及的檔案、函式設計、資料結構、執行順序
- 以 SPEC.md 為依據，不得超出 SPEC 範圍

### TEST.md — 驗收測試
- 如何確認這個功能符合 SPEC
- 包含：測試情境（正常/邊界/例外）、手動測試步驟、預期結果
- 以 SPEC.md 為標準，PLAN.md 的實作細節不影響驗收標準

## Design 規範

每個 `design-NNN/` 目錄包含兩份文件：

### DESIGN.md — UI 設計內容
- 視覺呈現、排版、元件、配色、互動行為
- 可包含 ASCII mockup 或文字描述的版面結構
- 不涉及實作細節（CSS 命名、JS 邏輯等）

### ACCEPT.md — User Acceptance Test
- 從使用者角度驗證設計是否達成預期體驗
- 包含：視覺檢查項目、互動流程、裝置相容性
- 以 DESIGN.md 為標準

## Fix 規範

每個 `fix-NNN/` 目錄包含兩份文件：

### FIX.md — 問題描述
- 問題是什麼、如何發現的
- 重現步驟、影響範圍
- 根本原因（已知或假設）

### TEST.md — 驗證方式
- 修復後如何確認問題已解決
- 回歸測試：確認修復不影響其他功能

## 工作流程

```
撰寫 SPEC.md
  → 撰寫 TEST.md（驗收標準先於實作）
  → 撰寫 PLAN.md
  → 實作
  → 依 TEST.md 驗收
  → 移至 archived/
```

## 編號規則

- Feature：`feat-001`, `feat-002`, ...（三位數補零）
- Fix：`fix-001`, `fix-002`, ...（三位數補零）
- Design：`design-001`, `design-002`, ...（三位數補零）
- 編號不重用，完成或作廢的項目移至 `archived/` 並保留編號

## Git / GitHub

CCStudio 目錄下所有專案的 GitHub 操作一律使用金鑰 `~/.ssh/id_ecdsa`。

push 指令範例：
```bash
GIT_SSH_COMMAND="ssh -i ~/.ssh/id_ecdsa -o IdentitiesOnly=yes" git push
```
