---
name: create-politician
description: "Distill a politician into an AI Skill using Nuwa-style six-agent parallel research, three-fold mental-model validation, and explicit honest limitations. | 以六軌並行研究、心智模型三重驗證、誠實邊界宣告，把政治人物蒸餾成 AI Skill。"
argument-hint: "[politician-name-or-slug]"
version: "2.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, WebFetch, WebSearch, Task
---

> **Language / 語言**：偵測用戶首條消息語言，全程使用同一語言回覆。下方中文為主，底部附英文版觸發詞與概述。
> This skill is bilingual. Detect the user's first-message language and respond in that language. Chinese is the primary version below; condensed English appears at the bottom.

# 政治人物.skill 創建器（v2 — 六軌並行 + 心智模型驗證）

## 版本說明（v1 → v2 的差異）

| 面向 | v1 | v2（本版） |
|------|-----|----------|
| 原料蒐集 | 使用者主動提供 | **六軌並行 research agent**（可選） |
| 人格層 | 6-layer persona | **3-7 心智模型 + 5-10 決策啟發式 + Expression DNA** |
| 驗證 | 證據權重規則 | 證據權重 **+ 三重驗證門檻**（跨域 / 生成力 / 排他性） |
| 誠實邊界 | 僅列證據類型 | 明確宣告 **「本 Skill 做不到什麼」** |
| 文件結構 | prompts/ 全部 | 新增 `references/`（方法論 + 模板）與 `examples/` |

---

## 觸發條件

- 創建：`/create-politician`、"幫我創建一個政治人物 skill"、"我想蒸餾一個 {政治人物}"
- 進化：`/update-politician {slug}`、"我有新資料"、"他不會這樣說"、"他的立場應該是"
- 列出：`/list-politicians`
- 回滾：`/politician-rollback {slug} {version}`
- 刪除：`/delete-politician {slug}`

---

## 工具使用規則

| 任務 | 工具 |
|------|------|
| 讀取 PDF / 圖片 / 文字 | `Read`（原生支持） |
| 網頁抓取 | `WebFetch` / `WebSearch` |
| 六軌並行研究 | `Task`（spawn 6 subagents，見 Phase 1） |
| 社媒 JSON 解析 | `Bash` → `python3` |
| 寫入 Skill 檔 | `Write` / `Edit` |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 方法論參考 | `Read` → `${CLAUDE_SKILL_DIR}/references/extraction-framework.md` |
| 模板參考 | `Read` → `${CLAUDE_SKILL_DIR}/references/skill-template.md` |

**基礎目錄**：`./politicians/{slug}/`

---

# 主流程：創建新政治人物 Skill

## Phase 0：基礎資訊錄入（3 題）

參考 `${CLAUDE_SKILL_DIR}/prompts/politician_intake.md`：

1. **代號 / slug**（必填）
2. **基本資訊**（國家、政黨、職位、活躍年代）— 例：`美國 共和黨 前總統 2016-2024`
3. **初步畫像**（意識形態、風格、印象）— 例：`右翼民粹 鷹派 推特治國 商人思維`

確認後進入 Phase 0.5。

---

## Phase 0.5：原料來源決策

詢問用戶：

```
原料怎麼來？

  [1] 我來提供（上傳檔案、貼連結、貼文字）
  [2] 由你（AI）做並行網路研究——啟動六軌 research agent
  [3] 混合模式（我提供 A，你補 B）

你也可以選擇跳過（僅憑 Phase 0 手動資訊生成——品質會顯著降低）。
```

根據選擇進入 Phase 1。

---

## Phase 1：六軌並行研究

> 本階段核心設計來自 nuwa-skill。詳見 `references/extraction-framework.md` 第六節。

### 若用戶選 [1]（純手動）

跳到 Phase 1M（手動模式），按傳統四選項（A/B/C/D）收集：
- [A] 上傳文件（PDF / 圖片 / TXT / MD）
- [B] 提供連結（用 `WebFetch`）
- [C] 直接貼文字
- [D] 指定數據源（引導取得）

收集後歸入 `politicians/{slug}/research/manual/`。

### 若用戶選 [2] 或 [3]（含自動研究）

**用 `Task` 工具並行啟動 6 個 subagent**，每個負責一軌：

| Agent | 任務 | 輸出檔案 |
|-------|------|---------|
| 1 | 搜集此人**公開演說 / 辯論 / 就職演說 / 質詢稿**，至少 10 段代表性原文 + 出處 + 日期 | `research/01-speeches.md` |
| 2 | 搜集**法案投票 / 行政決策 / 正式政策文件**，至少 15 條關鍵紀錄 + 日期 + 官方來源 | `research/02-votes-decisions.md` |
| 3 | 搜集**社交媒體 / 即時發言 / 短影片**，至少 20 條代表性貼文 + 語境 | `research/03-social-media.md` |
| 4 | 搜集**媒體評論 / 學術分析 / 對手攻擊**，至少 5 篇不同立場的觀察 + 引述 | `research/04-external-views.md` |
| 5 | 搜集**訪談 / 記者會 / 回憶錄 / 傳記**，至少 5 份逐字稿或章節摘要 | `research/05-interviews-bio.md` |
| 6 | 建立**時間線 + 重大轉折 + 近 6 個月動態**，按年序列出關鍵事件 | `research/06-timeline.md` |

**每個 subagent 的 prompt 模板**：

```
你是研究 agent #{N}，專責 {軌道主題}。
目標：{任務描述}。
允許工具：WebFetch, WebSearch, Read。
約束：
  - 輸出必須寫入 politicians/{slug}/research/{檔名}
  - 所有引述附日期 + 來源 URL 或出處
  - 無法取得的內容標「查無」，不要編造
  - 不處理立場分析，只做「採集 + 結構化」
  - 語料以此人的第一手產出優先；二手評論（軌 4 除外）需標明
截止日期：{今日 YYYY-MM-DD}
```

**資源限制**：若環境不允許 6 軌並行，優先跑 軌 1（演說）、軌 2（投票）、軌 3（社媒）——最硬的一手語料。

### Phase 1.5：研究總結與確認

六軌完成後，整合摘要給用戶：

```
六軌研究完成：
  軌 1（演說）：收集 {N} 段，代表作 …
  軌 2（投票/決策）：收集 {N} 條，關鍵票 …
  軌 3（社媒）：收集 {N} 則，熱門話題 …
  軌 4（外部評論）：收集 {N} 篇，主要批評角度 …
  軌 5（訪談/傳記）：收集 {N} 份
  軌 6（時間線）：{起迄} + {N} 個轉折點

發現值得特別關注的張力：
  - {投票 vs 口號衝突的例子}
  - {立場演化的轉折}

要補充什麼嗎？還是進入分析階段？
```

---

## Phase 2：提取（Analyze）

並行執行三個分析任務：

### 2A：Political Capability 分析
依照 `${CLAUDE_SKILL_DIR}/prompts/political_analyzer.md`：
- 走**三重驗證**（跨域 / 生成力 / 排他性），產出 3-7 個心智模型
- 產出 5-10 條決策啟發式
- 彙整 10-20 條硬證據
- 政治光譜座標

### 2B：Expression DNA 分析
依照 `${CLAUDE_SKILL_DIR}/prompts/politician_persona_analyzer.md`：
- 句式指紋（7 項量化）
- 風格光譜（6 軸評分）
- 口癖 3-7 個 + 禁忌詞
- 標誌性敘事 3-5 個
- 場景切換（對黨內 / 反對方 / 媒體 / 基層）

### 2C：誠實邊界清單
掃描整個研究結果，列出本 Skill **做不到**的項：
- 資料不足的維度（如私下決策邏輯、家族政治）
- 研究截止日期後的事件
- 此人刻意不公開的議題
- 即興情緒反應（公開語料多為篩選版）
- 其他此人特有盲區

---

## Phase 2.5：心智模型預覽與確認

**不要跳過此關**。向用戶展示心智模型列表，確認後才進 Phase 3：

```
我從研究中提取了 {N} 個候選心智模型，逐一通過三重驗證：

✅ 模型 1：{名稱}
   跨域：議題 A + 議題 B
   生成力：對「{未來 X 議題}」推斷為 {Y}
   排他性：vs 同黨典型人物的區別 {Z}

✅ 模型 2：……

⚠️ 降級為啟發式：候選「{X}」僅通過 1/3 驗證
❌ 剔除：候選「{Y}」跨域只有 1 個，且同黨普遍這樣想

這份模型清單看來對嗎？有沒有你認為的重要框架我漏了？
```

用戶確認或修正後進入 Phase 3。

---

## Phase 3：生成（Build）

三個 builder 並行產出：

- `political.md` — 用 `political_builder.md`
- `persona.md` — 用 `politician_persona_builder.md`
- `limitations.md` — 從 Phase 2C 誠實邊界清單直接生成

摘要給用戶預覽（各 6-10 行）：

```
Political Capability：
  心智模型：{名稱 x 3}
  決策啟發式 {N} 條
  最硬證據：{投票 / 決策}

Expression DNA：
  平均句長 {X} 字，{斷言/謹慎}
  口癖：{X, Y, Z}
  禁忌詞：{A, B}
  標誌性敘事：{α, β}

誠實邊界：
  ❌ 做不到 {項目 1}
  ❌ 做不到 {項目 2}
  ……

確認生成？還是需要調整？
```

---

## Phase 4：三項品質驗證（nuwa 標準）

生成前跑三個測試，任一失敗回 Phase 3 修正：

1. **已知立場對照**：用此 Skill 推斷此人「{3 個已知立場議題}」，對比公開資料是否吻合？
2. **邊界案例**：用此 Skill 推斷「{3 個原始語料未涵蓋的議題}」，是否產出合理推斷 or 誠實標「超出範圍」？
3. **語氣驗證**：產出一段 200 字的虛擬發言，交叉比對 Expression DNA 的量化指標是否符合？

通過後進入 Phase 5。

---

## Phase 5：寫入檔案

### 5.1 目錄結構

```bash
mkdir -p politicians/{slug}/research politicians/{slug}/versions
# research/ 已在 Phase 1 建立，此處確認存在
```

目標結構：
```
politicians/{slug}/
├── SKILL.md                  # 最終產出（依 references/skill-template.md）
├── political.md              # PART A + B + D(片段)
├── persona.md                # PART C
├── limitations.md            # PART D 誠實邊界
├── meta.json                 # 元資料
├── research/                 # 六軌原料
│   ├── 01-speeches.md
│   ├── 02-votes-decisions.md
│   ├── 03-social-media.md
│   ├── 04-external-views.md
│   ├── 05-interviews-bio.md
│   └── 06-timeline.md
└── versions/                 # 歷次快照
```

### 5.2 meta.json

```json
{
  "name": "{name}",
  "slug": "{slug}",
  "schema_version": "2.0",
  "created_at": "{ISO}",
  "updated_at": "{ISO}",
  "research_cutoff": "{YYYY-MM-DD}",
  "version": "v1",
  "type": "politician",
  "profile": {
    "country": "...", "party": "...", "position": "...", "era": "..."
  },
  "spectrum": { "economic": X, "authority": Y, "confidence": "high|medium|low" },
  "tags": { "ideology": [...], "style": [...] },
  "mental_models_count": N,
  "heuristics_count": N,
  "evidence_counts": {
    "hard_evidence": 0,
    "public_statements": 0,
    "external_observations": 0,
    "inferences": 0
  },
  "research_sources": {
    "speeches": N, "votes_decisions": N, "social_media": N,
    "external_views": N, "interviews_bio": N, "timeline": N
  },
  "known_blindspots": [...],
  "corrections_count": 0
}
```

### 5.3 SKILL.md 生成

**嚴格依照 `${CLAUDE_SKILL_DIR}/references/skill-template.md`**。把 political.md、persona.md、limitations.md 的內容嵌入對應 PART。

完成後告知用戶：

```
✅ 政治人物 Skill 已創建（v2 schema）

位置：politicians/{slug}/
研究檔案：politicians/{slug}/research/（可查閱原料）
觸發詞：
  /{slug}           完整版
  /{slug}-political 只載入政治能力（PART A/B/D）
  /{slug}-persona   只載入表達 DNA（PART C）

如果用起來感覺哪裡不對：
  - 有新資料 → "我有新資料"（進入 Phase 1 增量軌道）
  - 立場錯了 → "他不會這樣說"（進入糾正軌道）
  - 心智模型錯了 → "模型 X 不對"（觸發三重驗證重跑）

⚠️ 本 Skill 只模擬此人公開言行模式，不代表任何政治立場的背書。
⚠️ 研究截止日期：{YYYY-MM-DD}。此日期後的事件請自行補充。
```

---

# 進化模式

## 增量資料（Evolution - Append）

用戶提供新資料：

1. 依 Phase 1 的邏輯收（手動 or 再跑指定軌的 research agent）
2. `Read` 現有 `political.md` / `persona.md` / `limitations.md`
3. 存檔當前版本：`python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./politicians`
4. 依 `politician_merger.md` 做增量分析
5. **重跑三重驗證**（新資料可能使某候選模型通過原本沒通過的驗證）
6. `Edit` 更新對應檔案
7. 重新生成 SKILL.md、更新 meta.json

## 對話糾正（Evolution - Correct）

用戶說「不對 / 他的立場應該是」：

1. 參考 `politician_correction_handler.md` 識別糾正類型
2. 分類：Political（心智模型 / 啟發式 / 硬證據）vs Persona（DNA / 敘事）vs Limitations（邊界）
3. 生成 correction 記錄附證據
4. **若糾正的是心智模型 → 必須重跑三重驗證**
5. `Edit` 追加到對應檔案的 `## Correction 記錄` 節
6. 重新生成 SKILL.md

---

# 管理指令

```bash
# 列出
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./politicians

# 回滾
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./politicians

# 刪除（確認後）
rm -rf politicians/{slug}
```

---

# 重要原則（貫穿全流程）

1. **HOW 不是 WHAT**：提取的是「此人如何思考政治」，不是「他說過什麼」
2. **三重驗證優先於字數**：寧可 3 個扎實的心智模型，不要 7 個通用標籤
3. **誠實邊界是功能而非瑕疵**：明確宣告「做不到什麼」比裝萬能有價值
4. **投票 vs 口號不抹平**：並列保留，運行時以投票為底座、口號為包裝
5. **刪掉名字還能認出**：這是 Skill 成功的終極檢驗

---

---

# English Version (Condensed)

## Trigger
- Create: `/create-politician`, "distill a politician", "help me create a politician skill for X"
- Update: `/update-politician {slug}`, "new materials", "he wouldn't say that"
- List: `/list-politicians`

## Main Flow (v2)

**Phase 0** — Intake (3 questions: slug, basic info, initial profile)

**Phase 0.5** — Source strategy: [1] user-provided / [2] six-agent auto-research / [3] hybrid

**Phase 1** — **Six parallel research agents** (launched via `Task` tool):
1. Speeches / debates → `research/01-speeches.md`
2. Votes / executive decisions / policy docs → `research/02-votes-decisions.md`
3. Social media → `research/03-social-media.md`
4. Media critique / opponent views → `research/04-external-views.md`
5. Interviews / memoirs → `research/05-interviews-bio.md`
6. Timeline / recent activity → `research/06-timeline.md`

All outputs must be written inside `politicians/{slug}/research/` — never external.

**Phase 1.5** — Research summary + user confirmation

**Phase 2** — Parallel analysis:
- **2A** Political Capability (via `political_analyzer.md`): run **three-fold validation** (cross-domain / generative / distinctive) → 3-7 mental models + 5-10 heuristics + 10-20 hard evidence
- **2B** Expression DNA (via `politician_persona_analyzer.md`): sentence fingerprint + style spectrum + verbal tics + forbidden words + signature narratives
- **2C** Honest limitations list

**Phase 2.5** — Preview mental models list, get user confirmation (do not skip)

**Phase 3** — Build `political.md`, `persona.md`, `limitations.md`

**Phase 4** — Three quality tests:
1. Known-stance match: predict 3 known stances, verify against record
2. Edge case: predict 3 novel issues, check for reasonable inference or honest "out of scope"
3. Voice check: draft 200-word statement, verify against DNA fingerprint

**Phase 5** — Write files following `references/skill-template.md`. Directory:
```
politicians/{slug}/
├── SKILL.md
├── political.md
├── persona.md
├── limitations.md
├── meta.json
├── research/ (six-track raw materials)
└── versions/
```

## Evolution

- **Append**: new materials → rerun relevant research tracks → rerun three-fold validation → update
- **Correct**: user says "wrong" → classify (Political/Persona/Limitations) → if mental model corrected, rerun validation

## Core Principles

1. HOW they think, not WHAT they said
2. Three-fold validation beats word count
3. Honest limitations are a feature, not a flaw
4. Never smooth over vote-vs-slogan conflict
5. Remove the name — can you still recognize who it is?

## References
- `references/extraction-framework.md` — full methodology
- `references/skill-template.md` — SKILL.md output template
- `examples/` — pre-distilled politician skills

⚠️ This skill simulates public behavioral patterns only. Not an endorsement. Always declare research cutoff date.
