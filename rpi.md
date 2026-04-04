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

## 檔案管理

### 存放位置

每個 feature 的文件存放在 `.rpi/{feature-name}/` 底下：
- `.rpi/{feature-name}/requirement.md` — 需求文件
- `.rpi/{feature-name}/plan.md` — Plan 文件

是否將 `.rpi/` 加入 `.gitignore` 由使用這個 skill 的使用者自行決定。

### 階段之間的 Context 交接

階段之間透過檔案交接。每個階段結束時，將產出寫入對應的檔案。下一個階段讀取檔案作為 input。Context window 中的採訪來回對話會自然被 compact 掉，但檔案內容是持久的。

## 跳回機制

使用者可以在任何階段要求跳回前面的階段（例如在 Implementation 階段回到 Requirement 階段）。因為每個階段的產出都已寫成檔案，跳回時以現有檔案為基礎重新進入該階段的流程。

---

## 階段 1: R - Requirement 確認需求

這個階段目標是確認需求，先不要管如何實作。

If a question can be answered by exploring the codebase, explore the codebase instead.

### 1. 先問使用者: 我們要打造什麼？

### 2. 採訪使用者

使用 grill-me skill 採訪使用者，確認需求。

採訪結束條件：Claude 跟使用者都可以決定結束。Claude 覺得所有面向都已確認時會建議結束，使用者也可以隨時喊停。

### 3. 產出或更新需求文件

將採訪結果寫入 `.rpi/{feature-name}/requirement.md`。

需求文件沒有固定格式，但必須涵蓋：
- 使用者提出的所有需求
- 採訪過程中的所有決策及原因

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

If a question can be answered by exploring the codebase, explore the codebase instead.

### 1. 進入 Claude code 的 plan mode

讀取 `.rpi/{feature-name}/requirement.md` 作為 input。

### 2. 採訪使用者

使用 grill-me skill 採訪使用者，確認實作細節。

採訪結束條件同階段 1：Claude 跟使用者都可以決定結束。

### 3. 產出或更新 Plan 文件

使用 Claude Code plan mode 產生 Plan 內容，寫入 `.rpi/{feature-name}/plan.md`。

Plan 文件的格式和內容就是 Claude Code plan mode 原本會產生的內容。

### 4. Claude code Review Plan 文件

#### 4-1 確認 Plan 符合目前的 code convention

檢查目前的 Plan 跟 codebase 內的 code convention 是一致的。

- 如果兩者一致: 直接進到 4-2
- 如果兩者不一致:
  - 如果你覺得 Plan 的作法比較好: 詢問使用者
    - 如果使用者同意 Plan 的作法: 繼續往下執行
    - 如果使用者不同意 Plan 的作法: 回到步驟 3 更新文件
  - 如果你覺得 convention 的作法比較好: 回到步驟 3 更新文件

#### 4-2 確認 Plan 符合 best practice

根據使用者目前裝的 skill 以及 codebase 的技術棧（語言、框架），找出所有可能適合用來 review 這個 plan 的 skills。

**列出候選 skills 問使用者要用哪些來 review**，使用者確認後，為選中的每個 skill 啟一個 agent 來 review 這個 plan。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 3。

#### 4-3 確認 Plan 沒有任何潛在問題

Claude code 自己 review Plan 文件，問自己：這個 Plan 有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 3。

### 5. 使用者 Review Plan 文件

Plan mode 會直接顯示內容給使用者看，讓使用者確認沒有問題就進入下一個階段。

如果使用者有提出問題或改善的地方，視情況回到步驟 2 或步驟 3。

## 階段 2.5: P -> I

確保 Plan 文件已寫入 `.rpi/{feature-name}/plan.md`，所有重要資訊都已持久化到檔案中，再進入下一階段。

## 階段 3: I - Implementation 實作

### 1. 實作

讀取 `.rpi/{feature-name}/plan.md` 作為 input。

按照 Plan 的步驟逐步實作。每完成一個步驟，執行相關測試（如果有的話）。如果測試失敗，先修正再繼續下一步。

不自動 commit，留給使用者自己決定。

### 2. Claude code Review 實作

#### 2-1 確認實作符合 best practice

根據使用者目前裝的 skill 以及 codebase 的技術棧，找出所有可能適合用來 review implementation 的 skills。

**列出候選 skills 問使用者要用哪些來 review**，使用者確認後，為選中的每個 skill 啟一個 agent 來 review implementation。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。

#### 2-2 確認實作沒有任何潛在問題

Claude code 自己 review implementation，問自己：目前的實作有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。

### 3. 完成

程式碼改動就是最終產出。不額外產出文件。
