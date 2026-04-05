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
├── SKILL.md                    — 主檔（調度、共用行為）
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
- `.rpi/{feature-name}/implementation.md` — 偏差報告（僅在實作階段有需求或 plan 異動時產出）

### 階段之間的 Context 交接

每次呼叫 `/rpi` 都從階段 1 開始，不嘗試推斷或恢復之前的進度。

階段之間透過檔案交接。每個階段結束時，將產出寫入對應的檔案。下一個階段讀取檔案作為 input。Context window 中的採訪來回對話會自然被 compact 掉，但檔案內容是持久的。

## 跳回機制

在同一對話中，使用者可以在任何階段要求跳回前面的階段（例如在 Implementation 階段回到 Requirement 階段）。跳回時：
- **保留所有現有檔案**，不刪除也不備份
- 如果從階段 3（Implementation）跳回，提醒使用者 working tree 中可能有未完成的程式碼改動，建議先 stash 或 revert，避免半成品 code 干擾後續修改
- 以現有檔案為基礎重新進入該階段的流程（例如修改 requirement 而非重寫）

## 採訪方式

各階段需要採訪使用者時，採用 grill-me 的風格（但不實際呼叫 grill-me skill），直接在 rpi workflow 內進行：
- 一次問一個問題
- 每個問題附上建議答案，使用者可以直接同意或調整
- 如果問題可以透過探索 codebase 回答，就自己去看，不要問使用者
- 沿著決策樹逐步走完所有分支

## 語言

Skill 內部指令使用英文撰寫（Claude 的指令遵循能力最強）。

---

## 階段 1: R - Requirement 確認需求

這個階段目標是確認需求，先不要管如何實作。

### 1. 先問使用者: 我們要打造什麼？

### 2. 探索 Codebase

在採訪使用者前，先探索 codebase 建立足夠的背景知識，讓接下來的問題更精準：

- **既有架構**: 新功能相關區域目前的結構
- **使用中的 Patterns**: 命名、檔案配置、類似功能的先例
- **限制條件**: 依賴、API、或需要遵守的慣例

這是一次簡短且有針對性的探索 — 目標是讓採訪時能問出更好的問題（例如：「目前 auth 用的是 middleware pattern，新功能要跟著嗎？」而不是「auth 怎麼處理？」）。

### 3. 採訪使用者

用上述採訪方式，確認需求的所有面向。如果探索現有程式碼有助於確認需求（例如了解現有架構、相關模組、既有限制），主動去看 code 而不是問使用者。

採訪結束條件：Claude 跟使用者都可以決定結束。Claude 覺得所有面向都已確認時會建議結束，使用者也可以隨時喊停。

### 4. 產出或更新需求文件

將採訪結果寫入 `.rpi/{feature-name}/requirement.md`。

需求文件使用鬆散模板，可依實際內容調整：
- Overview — 要做什麼、為什麼
- Requirements — 具體功能需求
- Decisions & Reasoning — 採訪中的所有決策及原因（最關鍵的部分）
- Out of Scope — 明確不做的事
- Open Questions — 尚未解決的問題（理想上在這個階段結束前清空）

這個階段先不用給使用者看需求文件。

### 5. Claude code review 需求文件

Claude code 自己 review 需求文件，確認是否有任何潛在的需求沒有被定義清楚：
- 如果有的話，回到步驟 3 採訪使用者。
- 如果關於需求的所有決策樹的細節都已跟使用者確認過，雙方對需求有共同的理解了，進入下一個步驟。

### 6. 使用者 Review 需求文件

將需求文件在 terminal 中呈現給使用者看，讓使用者確認沒有問題就進入下一個階段。

如果使用者有提出問題或改善的地方，視情況回到步驟 3 或步驟 4。

## 階段 1.5: R -> P

將需求文件寫入 `.rpi/{feature-name}/requirement.md`（如尚未寫入），確保所有重要資訊都已持久化到檔案中，再進入下一階段。

## 階段 2: P - Plan 規劃如何實作

這個階段目標是規劃如何實作。

### 需求穩定性

需求已在階段 1 定案。這個階段專注在「怎麼做」，不是「做什麼」。盡量不要在 Plan 階段修改需求 — 大部分看起來像需求變更的東西，其實是屬於 plan 的實作細節。

但如果深入探索 codebase 後發現了真正的需求缺口（階段 1 沒考慮到的），需要：
1. 明確告知使用者：說明發現了什麼、為什麼影響需求
2. 取得使用者確認後才視為需求變更
3. 記錄變更 — 在階段 2.5 轉換時會需要更新 requirement.md

### 1. 進入 Claude code 的 plan mode

使用 `EnterPlanMode` 工具進入 plan mode（限制為唯讀，專注於思考和探索 codebase）。如果 `EnterPlanMode` 不可用，直接繼續但自律不做 code changes。

讀取 `.rpi/{feature-name}/requirement.md` 作為 input。

### 2. 探索 Codebase

在採訪使用者前，先探索 codebase 了解現有的 conventions 和架構：

- **Conventions**: 命名、檔案結構、patterns、error handling、testing style
- **架構**: 類似功能的結構方式、新功能會互動的相關模組
- **Dependencies**: 可重用的既有工具函式、library、抽象層

這讓接下來的採訪更有針對性（你已經知道限制條件），也讓產出的 plan 天然符合 codebase 慣例。

### 3. 採訪使用者

用上述採訪方式，確認實作細節。

採訪結束條件同階段 1：Claude 跟使用者都可以決定結束。

### 4. 撰寫 Plan

將 Plan 寫到 plan mode 自己的 plan file 中。

Plan 文件的格式和內容就是 Claude Code plan mode 原本會產生的內容。

### 5. Claude code Review Plan（仍在 plan mode 中）

所有 review 都是唯讀操作，不需要離開 plan mode。

#### 5-1 需求回溯驗證

打開 `.rpi/{feature-name}/requirement.md`，逐條檢查每個需求是否在 plan 中都有對應的具體步驟。需求在規劃過程中容易遺漏 — 尤其是 edge cases、out of scope 的邊界、以及非功能性需求。

- 如果每個需求都有對應的 plan 步驟：繼續
- 如果有需求缺失或只是模糊帶過：跟使用者討論是刻意延後還是遺漏，視情況回到步驟 4

#### 5-2 確認 Plan 符合目前的 code convention

將 plan 與步驟 2 探索到的 conventions 做比對。如果 codebase 有變動或需要確認細節，可以再次探索，但主要的了解工作應該在步驟 2 已完成。

- 如果兩者一致: 直接進到 5-3
- 如果兩者不一致:
  - 如果你覺得 Plan 的作法比較好: 詢問使用者
    - 如果使用者同意 Plan 的作法: 繼續往下執行
    - 如果使用者不同意 Plan 的作法: 回到步驟 4 更新 plan
  - 如果你覺得 convention 的作法比較好: 回到步驟 4 更新 plan

#### 5-3 確認 Plan 符合 best practice

根據使用者目前裝的 skill 以及 codebase 的技術棧（語言、框架），篩選出可能適合用來 review 這個 plan 的 skills，附上每個 skill 被選中的理由。

**列出候選 skills 問使用者要用哪些來 review**，使用者確認後，直接呼叫選中的 skill 來 review（不需離開 plan mode，review skill 只需唯讀操作）。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 4。

#### 5-4 確認 Plan 沒有任何潛在問題

Claude code 自己 review Plan 文件，問自己：這個 Plan 有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 4。

### 6. 使用者 Review

將 plan 呈現給使用者 review，仍留在 plan mode 中。

- 如果使用者有提出問題或改善的地方，視情況回到步驟 3 或步驟 4（仍在 plan mode 中）
- 如果使用者同意，使用 `ExitPlanMode` 離開 plan mode（如果有進入的話），進入下一階段

## 階段 2.5: P -> I

離開 plan mode 後：

1. 將 plan 內容寫入 `.rpi/{feature-name}/plan.md` 作為交接給階段 3 的檔案
2. 如果 Plan 階段有任何需求異動，更新 `.rpi/{feature-name}/requirement.md`：
   - 將變更套用到對應的章節
   - 在文件末尾新增 `## Requirement Changes (Phase 2)` 章節，列出每個變更及原因（例如：「新增 X，因為探索 codebase 發現 Y」）

再進入下一階段。

## 階段 3: I - Implementation 實作

### 需求與 Plan 穩定性

需求和 Plan 都已在前面的階段定案。這個階段專注在執行 plan，不回頭改決策。大部分實作中遇到的問題都可以在 plan 的框架內解決。

但偏差可能會發生，依據改動的對象不同處理方式也不同：

**需求異動** — 一律需要使用者確認。停下來說明發現了什麼、為什麼需求無法維持，取得明確同意後才繼續。

**Plan 異動** — 依影響程度判斷：
- *小幅調整*（例如調整步驟順序、換用類似的 utility、調整 function signature 以配合實際 code）：直接進行，不需要問使用者，但仍須記錄
- *重大偏離*（例如不同的架構方向、跳過原定步驟、新增大型元件）：停下來跟使用者確認

不論大小，所有偏差都要記錄 — 在步驟 3 完成時會寫入 implementation.md。

### 1. 實作

先讀取 `.rpi/{feature-name}/requirement.md` 了解需求背景，再讀取 `.rpi/{feature-name}/plan.md` 作為實作計畫。

在開始每個步驟前，先讀取即將要修改的原始碼檔案。Plan 描述的是「要改什麼」，但程式碼裡的細節（區域變數、既有函式中的 edge cases、最近的改動）需要 fresh context。

按照 Plan 的步驟逐步實作。在有意義的 checkpoint（一個函式完成、一個模組接好）執行相關測試，而非每個微小改動都跑。如果測試失敗，先修正再繼續下一步。

不自動 commit，留給使用者自己決定。

### 2. Claude code Review 實作

#### 2-1 確認實作符合 best practice

根據使用者目前裝的 skill 以及 codebase 的技術棧，篩選出可能適合用來 review implementation 的 skills，附上理由。

**列出候選 skills 問使用者要用哪些來 review**，使用者確認後，為每個選中的 skill 啟動一個 agent 平行執行 review。每個 agent 只需要 code changes 相關的 context，不需要額外提供 requirement.md 或 plan.md。

等所有 agent 完成後，統整所有 review 結果呈現給使用者。如果有 agent 失敗，告知使用者哪個 skill 的 review 未完成，不阻擋流程。如果不同 agent 的建議互相矛盾，一併呈現，由使用者決定採納哪些。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。

#### 2-2 需求回溯驗證

打開 `.rpi/{feature-name}/requirement.md`，逐條檢查每個需求是否都有對應的實作。Plan 可能遺漏或簡化了某些需求，所以要對照需求文件而非只對照 plan。

- 如果需求已完整實作：繼續下一條
- 如果需求部分實作或遺漏：跟使用者討論是刻意延後還是不小心漏掉，視情況回到步驟 1

#### 2-3 Self-Review（simplify 前）

Claude code 自己 review implementation，專注在正確性：
- 有沒有 bug、edge case、或 error handling 的漏洞？

在 simplify 前先確認邏輯正確 — 對剛寫的 code 做正確性驗證，比對重構過的 code 驗證更容易。

如果有發現問題，視情況問使用者問題，然後回到步驟 1。

#### 2-4 Simplify

對本次實作修改的檔案執行 `/simplify`，簡化並改善 code 的可讀性與一致性。明確限定 scope 為改動到的檔案，不動其他 code。自動執行，不需要問使用者。

如果 `/simplify` 不可用，自行做簡化 pass：檢查改動檔案中的重複邏輯、過度複雜的條件判斷、不必要的抽象層，並清理。

#### 2-5 Self-Review（simplify 後）

Simplify 後再次 review。簡化過程偶爾會以微妙的方式改變行為，確認：
- 重構後邏輯仍然正確
- Code 與 codebase 的 style 一致

如果有發現問題，視情況問使用者問題，然後回到步驟 1。

#### 2-6 完整測試

如果專案有 test suite，完整跑一次。步驟 1 只在 checkpoint 跑了相關測試 — 這步是為了抓跨模組的 integration regression。如果 test suite 太大無法全部跑，至少跑跟本次實作有接觸到的模組相關的測試檔案。

如果測試失敗：判斷是本次實作造成的還是既有的問題。本次實作造成的先修再繼續；既有問題告知使用者。

#### 2-7 Format & Lint

檢查專案是否有 formatter 或 linter（查看設定檔如 `.eslintrc`、`.prettierrc`、`biome.json` 等，以及 `package.json` / `pyproject.toml` 中的 scripts）。如果有，對改動到的檔案執行。如果沒有，跳過這步。

處理 linter 錯誤時：
- 不會改動到邏輯的：直接修
- 修改時可能改到邏輯的：先問使用者

### 3. 完成

程式碼改動就是最終產出。

#### 偏差報告

如果實作階段有任何需求或 plan 的異動，產出 `.rpi/{feature-name}/implementation.md`，記錄所有偏差：

```markdown
# Implementation Deviations

## Requirement Changes
- **[需求描述]**: [改了什麼] — **原因:** [為什麼需要改]

## Plan Changes
- **[plan 步驟描述]**: [改了什麼] — **原因:** [為什麼需要改]
```

只有在有偏差時才產出這個檔案。如果實作完全照需求和 plan 執行，則不需要。
