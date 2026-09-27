# governed-glossary

在一個專案裡，用一份由你決定的**規範用語表**管理用詞。agent 回覆時會標出每一個規範用語，讓你一眼分得出哪些詞是你定案的、哪些只是 agent 的建議。

這是一份 [Agent Skills](https://agentskills.io/specification) 格式的 skill：一個資料夾，裡面只有一個 `SKILL.md`，沒有腳本、hook 或伺服器。

## 為什麼需要它

和 agent 長時間討論一個專案，用詞很容易漂移：

- 同一個概念換了好幾個名字。
- 同一個詞指不同的東西，例如「確認」有時指你同意了，有時指查過事實。
- 撞到日常用詞，例如「簡報」會被讀成投影片。
- 中英文輪流出現，例如 release note 和發布說明。

結果是你讀不懂 agent 在說什麼，或誤以為某個名稱已經定下來了。

## 它怎麼幫你

- **每次都標記。** 回覆裡的規範用語寫成行內程式碼並加上半形括號，狀態用括號內開頭的小符號區分：

  | 狀態 | 寫法 | 意思 |
  |---|---|---|
  | 定案 | `[草稿]` | 你已決定 |
  | 暫定 | `[˜草稿]` | 你同意先暫時沿用 |
  | 建議 | `[ˀ草稿]` | agent 建議，你還沒定案 |
  | 停用 | `[ˣ暫存稿]` | 舊稱，不再使用 |

  你自己隨口用過的詞，也只算建議，要你明確定案才算數。
- **用自然的話管理。** 說「A 改成 B」「這幾個都定案」「停用 C」即可。agent 每次都立刻寫進檔案，提醒你新名稱會不會撞到日常用詞、和既有的詞重疊，或讓已寫好的文件出現舊名稱；只提醒，不擋你。
- **批次定案。** agent 不會每出現一個新詞就停下來問，而是等建議詞累積到一定數量（預設 10 個），或一個主題告一段落時，整理成一張表讓你一次決定。
- **檢查殘留的舊稱。** 改名之後，agent 會實際搜尋文件裡還在用舊名稱的地方，列出檔案與行號；每一處由你決定要不要改，不會自動改掉引述的原話。
- **在專案之間共用。** 可以把規範用語表匯出、匯入，或用符號連結掛載、參照到其他專案。
- **對話變長也有效。** 每次變動都寫進檔案，context 被壓縮後，agent 會重新讀取。

寫進檔案的文件，預設只用正式名稱、不加標記，因為文件的讀者不一定認得這套標記。

## 規範用語表放在哪裡

放在專案根目錄的 `.governed-glossary/`：

```text
my-project/
└── .governed-glossary/
    ├── project.jsonl          # 本專案自己的規範用語表，所有變動都寫進這裡
    └── other-project.jsonl    # 其他規範用語表，一律只讀
```

格式是 JSONL，一行一個詞。你不需要直接讀這個檔案，想看的時候請 agent 整理成表格即可。建議把 `.governed-glossary/` 放進版本控制，修改歷史就交給 git。

## 安裝

資料夾名稱必須是 `governed-glossary`，與 `SKILL.md` 裡的 `name` 相同。

### Claude Code

所有專案都能用（使用者層級）：

```bash
git clone https://github.com/chencyr/governed-glossary ~/.claude/skills/governed-glossary
```

只在某個專案用（專案層級），在專案根目錄執行：

```bash
git clone https://github.com/chencyr/governed-glossary .claude/skills/governed-glossary
```

### 其他支援 Agent Skills 的工具

把 `governed-glossary` 資料夾放到該工具讀取 skill 的目錄，位置請見各工具的文件。

## 使用方式

在 session 開始時輸入：

```text
/governed-glossary
```

專案還沒有 `.governed-glossary/` 時，agent 會先問你要不要建立。要使用別處的規範用語表，就把路徑當參數帶進去，agent 會問你要匯入（複製進來，歸本專案管）還是參照（連過去，跟著來源更新）：

```text
/governed-glossary ../other-project/.governed-glossary/project.jsonl
```

### 讓專案預設啟用

每次都手動呼叫很容易忘。第一次建立 `.governed-glossary/` 時，agent 會建議你在專案的 `AGENTS.md` 加一句話，例如：

```text
本專案使用規範用語，開始工作前先使用 governed-glossary skill。
```

之後在這個專案開 session，agent 就會自己使用這個 skill。你同意後它才會寫入。

注意：專案裡已經有 `CLAUDE.md` 時，Claude Code 不會讀 `AGENTS.md`。這時請在 `CLAUDE.md` 加一行 `@AGENTS.md` 匯入，或直接把那句話寫進 `CLAUDE.md`。

這個 skill 允許 agent 自行呼叫，專案指示檔裡的那句話才會有作用。它的觸發描述只允許三種情況：專案有 `.governed-glossary/`、專案指示檔要求，或你提到規範用語。

實測時發現，遇到和用詞有關的任務，例如統一用詞、整理專有名詞、翻譯，即使專案沒有 `.governed-glossary/`，agent 幾乎每次都會先載入這個 skill。這時它會判斷不適用，照一般方式完成工作，不會加標記，也不會要你建立規範用語表；代價是這次 session 多占一些 context。和用詞無關的任務，例如寫程式，實測沒有載入。

## 適用與不適用

**適用：**在一個專案裡長期討論或撰寫文件，希望用詞由你決定、不再漂移。

**不適用：**查一個詞的意思、翻譯，或只是要寫一份詞彙表文件。

## 語言

目前只有繁體中文版。

## 授權

[MIT](LICENSE)
