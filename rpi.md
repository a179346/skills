# RPI workflow

## 目標

打造一個在 claude code 可以重複使用的 workflow: `rpi`。

他會分成三個階段，協助我一步一步從確認需求到規劃到實作:
- r: requirement 確定需求
- p: plan 規劃如何實作
- i: implementation 實作

## 實作形式

以 Claude Code Skill 的形式實作。使用者進入 claude code 後輸入 `/rpi {feature-name}` 就可以開始這個 workflow。

`{feature-name}` 只允許簡短標籤（如 `auth-login`），用作資料夾名稱。

### Skill 結構

採用 progressive disclosure 架構，SKILL.md 作為 orchestrator，各階段細節放在 `references/` 下：

```
skills/rpi/
├── SKILL.md                    — 主檔（調度、狀態管理、共用行為）
└── references/
    ├── phase-r.md              — 階段 1: Requirement
    ├── phase-p.md              — 階段 2: Plan
    └── phase-i.md              — 階段 3: Implementation
```

根據當前階段只載入對應的 reference file，避免一次塞太多指令進 context。

## 檔案管理

### 存放位置

每個 feature 的文件存放在 `.rpi/{feature-name}/` 底下：
- `.rpi/{feature-name}/requirement.md` — 需求文件
- `.rpi/{feature-name}/plan.md` — Plan 文件

是否將 `.rpi/` 加入 `.gitignore` 由使用這個 skill 的使用者自行決定。

### 階段之間的 Context 交接

每次呼叫 `/rpi` 都從階段 1 開始，不嘗試推斷或恢復之前的進度。

階段之間透過檔案交接。每個階段結束時，將產出寫入對應的檔案。下一個階段讀取檔案作為 input。Context window 中的採訪來回對話會自然被 compact 掉，但檔案內容是持久的。

## 跳回機制

使用者可以在任何階段要求跳回前面的階段（例如在 Implementation 階段回到 Requirement 階段）。跳回時：
- **保留所有現有檔案**，不刪除也不備份
- 以現有檔案為基礎重新進入該階段的流程（例如修改 requirement 而非重寫）

## 採訪方式

各階段需要採訪使用者時，採用 grill-me 的風格（但不實際呼叫 grill-me skill），直接在 rpi workflow 內進行：
- 一次問一個問題
- 每個問題附上建議答案，使用者可以直接同意或調整
- 如果問題可以透過探索 codebase 回答，就自己去看，不要問使用者
- 沿著決策樹逐步走完所有分支

## 語言

Skill 內部指令使用英文撰寫（Claude 的指令遵循能力最強）。對使用者的回應則自動配合使用者的語言。

---

## 階段 1: R - Requirement 確認需求

這個階段目標是確認需求，先不要管如何實作。

### 1. 先問使用者: 我們要打造什麼？

### 2. 採訪使用者

用上述採訪方式，確認需求的所有面向。

採訪結束條件：Claude 跟使用者都可以決定結束。Claude 覺得所有面向都已確認時會建議結束，使用者也可以隨時喊停。

### 3. 產出或更新需求文件

將採訪結果寫入 `.rpi/{feature-name}/requirement.md`。

需求文件使用鬆散模板，可依實際內容調整：
- Overview — 要做什麼、為什麼
- Requirements — 具體功能需求
- Decisions & Reasoning — 採訪中的所有決策及原因（最關鍵的部分）
- Out of Scope — 明確不做的事
- Open Questions — 尚未解決的問題（理想上在這個階段結束前清空）

這個階段先不用給使用者看需求文件。

### 4. Claude code review 需求文件

Claude code 自己 review 需求文件，確認是否有任何潛在的需求沒有被定義清楚：
- 如果有的話，回到步驟 2 採訪使用者。
- 如果關於需求的所有決策樹的細節都已跟使用者確認過，雙方對需求有共同的理解了，進入下一個步驟。

### 5. 使用者 Review 需求文件

將需求文件在 terminal 中呈現給使用者看，讓使用者確認沒有問題就進入下一個階段。

如果使用者有提出問題或改善的地方，視情況回到步驟 2 或步驟 3。

## 階段 1.5: R -> P

將需求文件寫入 `.rpi/{feature-name}/requirement.md`（如尚未寫入），確保所有重要資訊都已持久化到檔案中，再進入下一階段。

## 階段 2: P - Plan 規劃如何實作

這個階段目標是規劃如何實作。

### 1. 進入 Claude code 的 plan mode

使用 `EnterPlanMode` 工具進入 plan mode（限制為唯讀，專注於思考和探索 codebase）。

讀取 `.rpi/{feature-name}/requirement.md` 作為 input。

### 2. 採訪使用者

用上述採訪方式，確認實作細節。

採訪結束條件同階段 1：Claude 跟使用者都可以決定結束。

### 3. 撰寫 Plan

將 Plan 寫到 plan mode 自己的 plan file 中。

Plan 文件的格式和內容就是 Claude Code plan mode 原本會產生的內容。

### 4. Claude code Review Plan（仍在 plan mode 中）

所有 review 都是唯讀操作，不需要離開 plan mode。

#### 4-1 確認 Plan 符合目前的 code convention

檢查目前的 Plan 跟 codebase 內的 code convention 是一致的。

- 如果兩者一致: 直接進到 4-2
- 如果兩者不一致:
  - 如果你覺得 Plan 的作法比較好: 詢問使用者
    - 如果使用者同意 Plan 的作法: 繼續往下執行
    - 如果使用者不同意 Plan 的作法: 回到步驟 3 更新 plan
  - 如果你覺得 convention 的作法比較好: 回到步驟 3 更新 plan

#### 4-2 確認 Plan 符合 best practice

根據使用者目前裝的 skill 以及 codebase 的技術棧（語言、框架），篩選出可能適合用來 review 這個 plan 的 skills，附上每個 skill 被選中的理由。

**列出候選 skills 問使用者要用哪些來 review**，使用者確認後，為選中的每個 skill 啟一個 agent 來 review 這個 plan。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 3。

#### 4-3 確認 Plan 沒有任何潛在問題

Claude code 自己 review Plan 文件，問自己：這個 Plan 有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 3。

### 5. 使用者 Review 並離開 Plan Mode

透過 `ExitPlanMode` 將 plan 呈現給使用者 review。

- 如果使用者有提出問題或改善的地方，視情況回到步驟 2 或步驟 3
- 如果使用者同意，離開 plan mode，進入下一階段

## 階段 2.5: P -> I

離開 plan mode 後，將 plan 內容寫入 `.rpi/{feature-name}/plan.md` 作為交接給階段 3 的檔案，再進入下一階段。

## 階段 3: I - Implementation 實作

### 1. 實作

讀取 `.rpi/{feature-name}/plan.md` 作為 input。

按照 Plan 的步驟逐步實作。在有意義的 checkpoint（一個函式完成、一個模組接好）執行相關測試，而非每個微小改動都跑。如果測試失敗，先修正再繼續下一步。

不自動 commit，留給使用者自己決定。

### 2. Claude code Review 實作

#### 2-1 確認實作符合 best practice

根據使用者目前裝的 skill 以及 codebase 的技術棧，篩選出可能適合用來 review implementation 的 skills，附上理由。

**列出候選 skills 問使用者要用哪些來 review**，使用者確認後，為選中的每個 skill 啟一個 agent 來 review implementation。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。

#### 2-2 確認實作沒有任何潛在問題

Claude code 自己 review implementation，問自己：目前的實作有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。

### 3. 完成

程式碼改動就是最終產出。不額外產出文件。
