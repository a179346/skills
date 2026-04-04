# RPI workflow

## 目標

打造一個在 claude code 可以重複使用的 workflow: `rpi`。

他會分成三個階段，協助我一步一步從確認需求到規劃到實作:
- r: requirement 確定需求
- p: plan 規劃如何實作
- i: implementation 實作

我只要進入 claude code 後輸入 `/rpi {要開發的feature名稱}` 就可以開始這個 workflow。

你幫我挑選適合的方式： plugin / skill / command / agent / script / 其他 。

在不同階段要想一個方法，可以把 context 的雜訊移除，留下所有重要資訊給下一個階段。

## 階段 1: R - Requirement 確認需求

這個階段目標是確認需求，先不要管如何實作。

If a question can be answered by exploring the codebase, explore the codebase instead.

### 1. 先問使用者: 我們要打造什麼？

### 2. 採訪使用者

使用 grill-me skill 採訪使用者，確認需求。

### 3. 產出或更新需求文件

這個階段先不用給使用者看需求文件。

### 4. Claude code review 需求文件

Claude code 自己 review 需求文件，確認是否有任何潛在的需求沒有被定義清楚：
- 如果有的話，回到步驟 2 採訪使用者。
- 如果關於需求的所有決策樹的細節都已跟使用者確認過，雙方對需求有共同的理解了，進入下一個步驟。

### 5. 使用者 Review 需求文件

把需求文件呈現給使用者看，讓使用者確認沒有問題就進入下一個階段。

如果使用者有提出問題或改善的地方，視情況回到步驟 2 或步驟 3。

## 階段 1.5: R -> P

這個階段要把需求文件交接給 Plan 階段，並且把 context 中的雜訊移除掉。

## 階段 2: P - Plan 規劃如何實作

這個階段目標是規劃如何實作。

If a question can be answered by exploring the codebase, explore the codebase instead.

### 1. 進入 Claude code 的 plan mode

### 2. 採訪使用者

使用 grill-me skill 採訪使用者，確認實作細節。

### 3. 產出或更新 Plan 文件

這個階段先不用給使用者看 Plan 文件。

### 4. Claude code Review Plan 文件

#### 4-1 確認 Plan 符合 best practice

根據使用者目前裝的 skill 以及 codebase 找出所有適合的 skills，並為他們每個啟一個 agent 來 review 這個 plan。

舉例: 如果這個 codebase 是用 react 寫的，且使用者有裝 vercel-react-best-practices 這個 skill，就啟動一個 agent 用 vercel-react-best-practices skill 來 review plan。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 3。

#### 4-2 確認 Plan 沒有任何潛在問題

Claude code 自己 review Plan 文件，問自己：這個 Plan 有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 3。

### 5. 使用者 Review Plan 文件

把 Plan 文件呈現給使用者看，讓使用者確認沒有問題就進入下一個階段。

如果使用者有提出問題或改善的地方，視情況回到步驟 2 或步驟 3。

## 階段 2.5: P -> I

這個階段要把 Plan 文件交接給 Implementation 階段，並且把 context 中的雜訊移除掉。

## 階段 3: I - Implementation 實作

### 1. 實作

### 2. Claude code Review 實作

#### 2-1 確認實作符合 best practice

根據使用者目前裝的 skill 以及 codebase 找出所有適合的 skills，並為他們每個啟一個 agent 來 review implementation。

舉例: 如果這個 codebase 是用 react 寫的，且使用者有裝 vercel-react-best-practices 這個 skill，就啟動一個 agent 用 vercel-react-best-practices skill 來 review implementation。

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。

#### 2-2 確認實作沒有任何潛在問題

Claude code 自己 review implementation，問自己：目前的實作有沒有什麼潛在問題

如果 review 時有發現可以調整的地方時，視情況問使用者問題，然後回到步驟 1。