# skillify

## 目標

建立 `skillify` skill。

skillify skill: 從當前 session 的對話歷史中提煉出 repeatable process，產出一個完整的 Claude Code skill。

使用者在對話中經歷了一個值得複用的流程後，呼叫 `/skillify` 來把它 skill 化。

## 實作形式

以 Claude Code Skill 的形式實作。使用者輸入 `/skillify {skill-name}` 就可以開始。

`{skill-name}` 只允許簡短的 kebab-case 標籤（如 `debug-memory-leak`、`review-api-endpoint`），用作 skill 的目錄名稱。如果使用者沒提供，就問他要叫什麼。如果提供的不是 kebab-case，自動轉換後跟使用者確認（例如 `"my cool skill"` → `my-cool-skill`）。

### Skill 結構

```
skills/skillify/
├── SKILL.md
└── references/
    └── writing-guide.md    — skill-creator 不可用時的 fallback 指引
```

### 產出結構

skillify 產出的 skill 遵循跟 `rpi` 一樣的結構慣例：

- `{output-path}/{skill-name}/SKILL.md` — 主檔，包含 description frontmatter、整體流程調度、共用行為
- `{output-path}/{skill-name}/references/` — 各階段或模組的詳細指令（如果流程複雜有多個階段）

如果提煉出來的流程很簡單（單一線性流程），可能只需要一個 `SKILL.md`，不一定需要 references/。skillify 應該根據流程的複雜度自行判斷是否需要拆分。

### 產出路徑

skillify 是通用 skill，可在任何 repo 中使用。產出的 skill 建議預設路徑為 `{current-project-root}/.claude/skills/`（`{current-project-root}` 是 placeholder，實際執行時替換為當前專案的根目錄路徑），但在採訪階段詢問使用者確認或更改。

## 檔案管理

不需要中間產物。skillify 的流程較短，整個過程在一次對話中完成，context 被 compact 的機率低。最終產出的 skill 檔案本身就是持久化的。

## 語言

Skill 內部指令使用英文撰寫（Claude 的指令遵循能力最強）。產出的 skill 檔案也預設用英文撰寫（因為是給 Claude 讀的指令）。

---

## 階段 1: 提煉（Extract）

回顧當前 session 的對話歷史，識別出 **primary successful workflow** — 真正達成使用者目標的步驟序列。

過濾掉：
- 失敗的嘗試和回退
- 這次特有的 debugging/troubleshooting
- 一次性的探索命令

如果對話中有多個不同的 workflow，問使用者要 skillify 哪一個。

### 既有 Skill

如果指定的 output path 已經存在同名 skill，先讀取它並呈現給使用者。問使用者：要根據這次對話修訂現有 skill，還是從頭開始？

### 產出

在 terminal 中向使用者展示摘要（不寫檔案）：

- **識別出的 repeatable process** — 這次 session 做了什麼，哪些步驟是可複用的
- **建議的 skill 觸發條件** — 什麼情況下應該用這個 skill
- **初步的步驟拆解** — 大概分幾步、每步做什麼

這個摘要是採訪的起點。

### Edge Case: Context 不足

如果 context 被大量 compact 或流程不明顯，誠實告知使用者，然後請使用者口頭描述流程。提煉階段變成使用者主導描述 + Claude 整理。不要拒絕執行 — 使用者腦中可能有明確的流程想 skill 化。

## 階段 2: 載入 skill-writing 知識

### 優先：呼叫 `/skill-creator`

檢查 available skills list 中是否有 `/skill-creator`。如果有，呼叫它 — 但只載入 skill-writing 知識，**不啟動 eval/iterate/test-case 流程**。明確告知 skill-creator 你是來寫一個基於已提煉流程的 skill。

### Fallback：`references/writing-guide.md`

如果使用者沒有安裝 skill-creator，讀取 skillify 自帶的 `references/writing-guide.md` 作為 skill-writing 指引。

放在採訪前是因為 skill-writing 知識在兩個地方有用：
- **採訪時**：知道一個好的 skill 需要定義什麼，才能問出對的問題
- **撰寫時**：知道 SKILL.md 的 best practice

## 階段 3: 採訪（Refine）

用 grill-me 的風格（但不實際呼叫 grill-me skill）跟使用者確認細節：

- 一次問一個問題
- 每個問題附上建議答案，使用者可以直接同意或調整
- 如果問題可以透過探索 codebase 回答，就自己去看，不要問使用者
- 沿著決策樹逐步走完所有分支

### 採訪面向（依複雜度調整深度）

**所有流程都要涵蓋：**

1. **泛化 vs 特化** — 這次 session 的哪些部分是通用的、哪些是這次特有的？
2. **觸發條件** — 什麼時候應該用這個 skill？使用者會怎麼呼叫它？
3. **步驟調整** — 提煉出的步驟要不要調整順序、合併、或拆分？
4. **產出物** — skill 執行完會留下什麼 artifacts？
5. **產出路徑** — skill 檔案要放在哪裡？建議預設為 `{current-project-root}/.claude/skills/`

**複雜流程額外涵蓋：**

6. **參數** — skill 需要什麼輸入？
7. **結束條件** — 每個步驟什麼時候算完成？整個 skill 什麼時候算結束？

低爭議的項目可以合併成一個確認：例如「我建議產出路徑和產出物的預設是 [X, Y]，OK？」

採訪結束條件：Claude 跟使用者都可以決定結束。Claude 覺得所有面向都已確認時會建議結束，使用者也可以隨時喊停。

## 階段 4: 撰寫（Write）

結合 skill-writing 知識，產出完整的 skill 檔案：

- `{output-path}/{skill-name}/SKILL.md` — 主檔
- `{output-path}/{skill-name}/references/*.md` — 如果流程複雜需要拆分

根據流程的複雜度自行判斷是否需要 references/。

## 階段 5: Review

### 1. Self-Review

Claude 自己檢查產出的 skill：

- **完整性** — 步驟是否完整、有沒有遺漏採訪中確認的內容
- **清晰度** — 指令是否清晰明確，未來的 Claude 能正確執行
- **泛化程度** — 沒有殘留這次 session 的特定細節（hardcoded 路徑、專案名等）
- **Frontmatter** — name 和 description 正確，description 具體到可以可靠觸發
- **結構** — progressive disclosure 是否適合這個複雜度，references 的使用是否合理

如果有問題，自行修正或回到階段 3 跟使用者確認。

### 2. 使用者 Review

把產出的 skill 內容呈現給使用者確認。

- 如果使用者有問題或改善建議，視情況回到階段 3 或階段 4
- 如果使用者同意，進入階段 6

## 階段 6: 驗證（Verify）

確認 skill 檔案已正確寫入指定位置。檢查：

- `{output-path}/{skill-name}/SKILL.md` 存在且內容完整
- 如果有 references/，所有 reference 檔案都存在且內容完整
- SKILL.md 與 reference 檔案之間的交叉引用都有效

如果有檔案缺失或內容不完整，修正後再次驗證。全部確認無誤後，流程結束。
