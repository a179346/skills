# create-verify-skill

## 目標

建立一個 skill 叫做 `create-verify-skill` 協助使用者為他的專案建立專屬的 `verify` skill。

建立出來的 `verify` skill: Verify a code change does what it should by running the app, testing the changed functionality, and fixing failure automatically.

## 觸發方式

使用者手動下 `/create-verify-skill`

## create-verify-skill 的內容

## 採訪方式

各階段需要採訪使用者時，採用 grill-me 的風格（但不實際呼叫 grill-me skill）：
- 一次問一個問題
- 每個問題附上建議答案，使用者可以直接同意或調整
- 如果問題可以透過探索 codebase 回答，就自己去看，不要問使用者
- 沿著決策樹逐步走完所有分支

### 1. 啟用 claude code skill-creator skill

如果使用者沒裝 skill-creator 就引導使用者去裝 skill-creator plugin。

使用者如果不想裝，就跳過這個步驟。

### 2. 跟使用者確認要建立的 skill

#### 2-1 建立的 skill 名稱

#### 2-2 建立出來的 skill 要用來驗證哪個專案

預設問是不是 claude code 當下所在路徑。

但使用者要可以手動輸入專案在本機的路徑。

#### 2-3 確認 skill 建立後要放的路徑

預設問使用者是不是要放在這個資料夾底下: `/{target-project-root}/.claude/skills/{skill-name}/`

但使用者要可以手動指定 skill 要放的位置。

- `{target-project-root}` 跟 `{skill-name}` 都是 placeholder，實際上顯示給使用者時要替換掉

### 3. 了解要建立 skill 的專案

專注在了解:
- 有哪些 testing 可以執行
- 有哪些跟 testing 相關的 script
- 要如何啟動 app
- 要如何測試
  - api server 專案 (nodejs / express / ...) -> 了解如何測試 api / 有什麼驗證機制才能呼叫 API / ...
  - web server 專案 (react / next.js / ...) -> 了解如何啟動 server / 需不需要登入 / 怎麼在瀏覽器環境開啟 / 怎麼讓 claude code 控制瀏覽器 / ...
  - mobile 專案 (react-native / ...) -> 了解如何啟動 server / 需不需要登入 / 怎麼在模擬機環境開啟 / 怎麼讓 claude code 控制模擬機 / ...
- 其他

### 4. 再次採訪使用者

使用上面提到的採訪方式，訪問使用者。

目標是對如何測試有完整的了解。

### 5. 跟使用者確認 skill 的內容

根據剛剛討論的結果整理出重點，呈現給使用者看，讓使用者確認。

### 6. 產出 skill 到指定資料夾底下