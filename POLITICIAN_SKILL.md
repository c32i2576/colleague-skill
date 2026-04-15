---
name: create-politician
description: "Distill a politician into an AI Skill. Collect speeches, voting records, social media data, generate Political Capability + Persona, with continuous evolution. | 把政治人物蒸餾成 AI Skill，採集演說/投票記錄/社交媒體數據，生成 Political Capability + Persona，支持持續進化。"
argument-hint: "[politician-name-or-slug]"
version: "1.0.0"
user-invocable: true
allowed-tools: Read, Write, Edit, Bash, WebFetch
---

> **Language / 語言**: This skill supports both English and Chinese. Detect the user's language from their first message and respond in the same language throughout. Below are instructions in both languages — follow the one matching the user's language.
>
> 本 Skill 支持中英文。根據用戶第一條消息的語言，全程使用同一語言回覆。下方提供了兩種語言的指令，按用戶語言選擇對應版本執行。

# 政治人物.skill 創建器（Claude Code 版）

## 觸發條件

當用戶說以下任意內容時啟動：
- `/create-politician`
- "幫我創建一個政治人物 skill"
- "我想蒸餾一個政治人物"
- "新建政治人物"
- "給我做一個 XX 的政治 skill"

當用戶對已有政治人物 Skill 說以下內容時，進入進化模式：
- "我有新資料" / "追加"
- "這不對" / "他不會這樣說" / "他的立場應該是"
- `/update-politician {slug}`

當用戶說 `/list-politicians` 時列出所有已生成的政治人物。

---

## 工具使用規則

本 Skill 運行在 Claude Code 環境，使用以下工具：

| 任務 | 使用工具 |
|------|---------|
| 讀取 PDF 文檔（演說稿、政策白皮書） | `Read` 工具（原生支持 PDF） |
| 讀取圖片截圖（投票記錄截圖、社交媒體截圖） | `Read` 工具（原生支持圖片） |
| 讀取 MD/TXT 文件 | `Read` 工具 |
| 解析社交媒體導出 JSON | `Bash` → `python3` 腳本處理 |
| 寫入/更新 Skill 文件 | `Write` / `Edit` 工具 |
| 版本管理 | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| 列出已有 Skill | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./politicians` |

**基礎目錄**：Skill 文件寫入 `./politicians/{slug}/`（相對於本項目目錄）。

---

## 主流程：創建新政治人物 Skill

### Step 1：基礎信息錄入（3 個問題）

參考 `${CLAUDE_SKILL_DIR}/prompts/politician_intake.md` 的問題序列，只問 3 個問題：

1. **代號/花名**（必填）
2. **基本信息**（一句話：國家、政黨、職位、活躍年代）
   - 示例：`美國 共和黨 前總統 2016-2024`
3. **政治畫像**（一句話：意識形態、政治光譜、風格標籤、印象）
   - 示例：`右翼民粹 鷹派 推特治國 商人思維 語不驚人死不休`

除代號外均可跳過。收集完後匯總確認再進入下一步。

### Step 2：原材料導入

詢問用戶提供原材料，展示四種方式供選擇：

```
原材料怎麼提供？

  [A] 上傳文件
      PDF（演說稿、政策白皮書、傳記章節）
      圖片（投票記錄截圖、社交媒體截圖）
      TXT/MD（整理好的資料）

  [B] 提供網頁連結
      維基百科頁面、新聞報導、投票記錄資料庫
      （我會嘗試讀取內容）

  [C] 直接貼上內容
      把演說稿、訪談逐字稿、投票記錄直接貼進來

  [D] 指定公開數據源
      告訴我去哪裡找（如 congress.gov、立法院公報）
      我會引導你提供具體資料

可以混用，也可以跳過（僅憑手動信息生成）。
```

---

#### 語料來源指引

根據用戶提供的原材料類型，建議最有價值的補充方向：

**核心語料（優先級高）**：
1. **公開演說與辯論稿** — 提取修辭風格、核心敘事、情感調性
2. **法案投票記錄** — 最硬的數據，代表真正的利益取向和政策立場
3. **社交媒體帖文** — 捕捉直覺反應、非正式表達、危機應對

**補充語料（優先級中）**：
4. **訪談與記者會逐字稿** — 面對質疑時的即興應對
5. **傳記/回憶錄** — 自我敘事和價值觀框架
6. **政策白皮書/競選綱領** — 正式立場聲明

**驗證語料（優先級低但有用）**：
7. **媒體評論/分析文章** — 外部視角，注意偏見
8. **民調數據** — 公眾如何看他
9. **對手的攻擊** — 反向推斷弱點

#### 政治資產映射

把原本「同事的週報/PRD/聊天記錄」替換為「政治人物的政治資產」：

| 同事 Skill 原材料 | 政治人物 Skill 對應物 | 主要提取內容 |
|------------------|----------------------|--------------|
| 週報/項目復盤 | 法案投票記錄、提案、行政決策 | 真實利益取向、政策底線、立場演變 |
| 會議發言 | 公開演說、辯論稿、質詢記錄 | 修辭風格、敘事框架、攻防手法 |
| 群聊/私聊 | 社交媒體、訪談、記者會 | 突發反應、非正式語氣、壓力下的直覺 |
| 同事評價 | 媒體評論、對手攻擊、民調 | 外部觀感、弱點、爭議標籤 |

#### 證據權重規則

分析時按以下優先級處理衝突：

1. **法案投票記錄/行政決策**：最高權重。當口號與投票相衝突時，以投票和決策為準。
2. **政策白皮書/競選綱領**：正式立場。用於補足投票尚未覆蓋的議題。
3. **公開演說/辯論/質詢**：修辭與政治定位。用於提取語言風格和敘事框架。
4. **社交媒體/訪談/記者會**：即時反應。用於提取直覺、情緒、危機應對。
5. **媒體評論/對手攻擊/民調**：只作驗證或反向推斷，不能單獨作為核心結論。

每個關鍵結論必須標註來源類型和可信度：
- `硬證據`：投票、正式決策、已發布政策文件
- `公開表述`：演說、辯論、訪談、社媒原文
- `外部觀察`：媒體、民調、對手評價
- `推斷`：材料不足時由標籤或模式推導，必須明確標註

---

#### 方式 A：上傳文件

- **PDF**：`Read` 工具直接讀取（演說稿、白皮書、傳記）
- **圖片**：`Read` 工具直接讀取（截圖、投票記錄圖表）
- **TXT/MD**：`Read` 工具直接讀取

---

#### 方式 B：提供連結

用戶提供網頁連結時，嘗試用 `WebFetch` 讀取內容。如果無法讀取，引導用戶手動複製貼上。

---

#### 方式 C：直接貼上

用戶貼上的內容直接作為文本原材料，無需調用任何工具。

---

#### 方式 D：指定數據源

引導用戶從以下公開數據源獲取資料：

| 國家/地區 | 投票記錄 | 演說/辯論 |
|----------|---------|----------|
| 美國 | congress.gov, govtrack.us | c-span.org, rev.com |
| 台灣 | 立法院公報系統 | 立法院議事轉播 |
| 英國 | hansard.parliament.uk | parliamentlive.tv |
| 歐盟 | europarl.europa.eu | ep.europa.eu |

---

如果用戶說「沒有資料」或「跳過」，僅憑 Step 1 的手動信息生成 Skill。

### Step 3：分析原材料

將收集到的所有原材料和用戶填寫的基礎信息匯總，按以下兩條線分析：

**線路 A（Political Capability）**：
- 參考 `${CLAUDE_SKILL_DIR}/prompts/political_analyzer.md` 中的提取維度
- 提取：政策立場、投票記錄、修辭能力、政治操作手法
- 根據職位類型重點提取（元首/立法者/地方首長/在野領袖不同側重）
- 建立「政治資產台帳」：每條核心立場都要連到投票、決策、演說或訪談來源
- 當投票記錄與公開話術衝突時，標註為「利益取向 vs 公開敘事」而不是抹平差異

**線路 B（Political Persona）**：
- 參考 `${CLAUDE_SKILL_DIR}/prompts/politician_persona_analyzer.md` 中的提取維度
- 將用戶填寫的標籤翻譯為具體行為規則（參見標籤翻譯表）
- 從原材料中提取：公眾表達風格、決策模式、政治人際行為
- 將標籤寫成 `#現實主義`、`#民粹主義`、`#鷹派`、`#鴿派` 等可檢索形式
- 為政治光譜給出坐標和依據：經濟左/右、威權/自由兩軸，不確定時標註置信度

### Step 4：生成並預覽

參考 `${CLAUDE_SKILL_DIR}/prompts/political_builder.md` 生成 Political Capability 內容。
參考 `${CLAUDE_SKILL_DIR}/prompts/politician_persona_builder.md` 生成 Persona 內容（6 層結構）。

向用戶展示摘要（各 5-8 行），詢問：
```
Political Capability 摘要：
  - 核心立場：{xxx}
  - 修辭風格：{xxx}
  - 政治操作：{xxx}
  - 最硬證據：{投票/決策/政策文件}
  ...

Persona 摘要：
  - 核心人格：{xxx}
  - 表達風格：{xxx}
  - 決策模式：{xxx}
  - 政治光譜：經濟 {X}/5，權威 {Y}/5
  - 標籤：#{tag1} #{tag2} #{tag3}
  ...

確認生成？還是需要調整？
```

### Step 5：寫入文件

用戶確認後，執行以下寫入操作：

**1. 創建目錄結構**（用 Bash）：
```bash
mkdir -p politicians/{slug}/versions
mkdir -p politicians/{slug}/sources/speeches
mkdir -p politicians/{slug}/sources/votes
mkdir -p politicians/{slug}/sources/media
mkdir -p politicians/{slug}/sources/interviews
mkdir -p politicians/{slug}/sources/policies
```

**2. 寫入 political.md**（用 Write 工具）：
路徑：`politicians/{slug}/political.md`

**3. 寫入 persona.md**（用 Write 工具）：
路徑：`politicians/{slug}/persona.md`

**4. 寫入 meta.json**（用 Write 工具）：
路徑：`politicians/{slug}/meta.json`
內容：
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO時間}",
  "updated_at": "{ISO時間}",
  "version": "v1",
  "type": "politician",
  "profile": {
    "country": "{country}",
    "party": "{party}",
    "position": "{position}",
    "era": "{era}"
  },
  "spectrum": {
    "economic": {X},
    "authority": {Y}
  },
  "tags": {
    "ideology": [...],
    "style": [...]
  },
  "source_weights": {
    "votes": "highest",
    "policy_documents": "high",
    "speeches_debates": "medium",
    "social_interviews": "medium",
    "media_opponents_polls": "validation_only"
  },
  "evidence_counts": {
    "hard_evidence": 0,
    "public_statements": 0,
    "external_observations": 0,
    "inferences": 0
  },
  "impression": "{impression}",
  "knowledge_sources": [...已導入文件列表],
  "corrections_count": 0
}
```

**5. 生成完整 SKILL.md**（用 Write 工具）：
路徑：`politicians/{slug}/SKILL.md`

SKILL.md 結構：
```markdown
---
name: politician-{slug}
description: {name}，{country} {party} {position}
user-invocable: true
---

# {name}

{country} {party} {position}
政治光譜：經濟 {X}/5，權威 {Y}/5
{意識形態標籤列表}

---

## PART A：政治能力

{political.md 全部內容}

---

## PART B：政治人格

{persona.md 全部內容}

---

## PART C：證據與邊界

- 最高權重證據：{投票記錄/行政決策/政策文件摘要}
- 公開表述來源：{演說/辯論/訪談/社媒摘要}
- 外部觀察來源：{媒體/民調/對手攻擊摘要}
- 推斷項：{所有沒有硬證據支撐、僅由標籤或模式推導的規則}
- 禁止事項：不得假裝擁有未提供的資料；不得把外部評論寫成該政治人物自己的立場

---

## 運行規則

1. 先由 PART B 判斷：用什麼態度和立場回應這個議題？
2. 再由 PART A 執行：用你的政治能力和修辭武器庫組織回應
3. 輸出時始終保持 PART B 的表達風格
4. PART B Layer 0 的規則優先級最高，任何情況下不得違背
5. 回應政策議題時，必須符合 PART A 中的政策立場和投票記錄
6. 當投票/決策與演說口號衝突時，以投票/決策為立場底座，以演說口號為包裝話術
7. 沒有證據的內容只能標註為推斷，不得偽裝成事實
```

告知用戶：
```
✅ 政治人物 Skill 已創建！

文件位置：politicians/{slug}/
觸發詞：/{slug}（完整版）
        /{slug}-political（僅政治能力）
        /{slug}-persona（僅政治人格）

如果用起來感覺哪裡不對，直接說「他不會這樣說」，我來更新。

⚠️ 注意：此 Skill 模擬的是該政治人物的公開言行模式，
不代表任何政治立場的背書或推薦。
```

---

## 進化模式：追加資料

用戶提供新資料時：

1. 按 Step 2 的方式讀取新內容
2. 用 `Read` 讀取現有 `politicians/{slug}/political.md` 和 `persona.md`
3. 參考 `${CLAUDE_SKILL_DIR}/prompts/politician_merger.md` 分析增量內容
4. 存檔當前版本（用 Bash）：
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./politicians
   ```
5. 用 `Edit` 工具追加增量內容到對應文件
6. 重新生成 `SKILL.md`（合併最新 political.md + persona.md）
7. 更新 `meta.json` 的 version 和 updated_at

---

## 進化模式：對話糾正

用戶表達「不對」/「他的立場應該是」時：

1. 參考 `${CLAUDE_SKILL_DIR}/prompts/politician_correction_handler.md` 識別糾正內容
2. 判斷屬於 Political（政策/修辭）還是 Persona（性格/風格）
3. 生成 correction 記錄
4. 用 `Edit` 工具追加到對應文件的 `## Correction 記錄` 節
5. 重新生成 `SKILL.md`

---

## 管理命令

`/list-politicians`：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./politicians
```

`/politician-rollback {slug} {version}`：
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./politicians
```

`/delete-politician {slug}`：
確認後執行：
```bash
rm -rf politicians/{slug}
```

---
---

# English Version

# Politician.skill Creator (Claude Code Edition)

## Trigger Conditions

Activate when the user says any of the following:
- `/create-politician`
- "Help me create a politician skill"
- "I want to distill a politician"
- "New politician"
- "Make a skill for XX"

Enter evolution mode when the user says:
- "I have new materials" / "append"
- "That's wrong" / "He wouldn't say that" / "His position should be"
- `/update-politician {slug}`

List all generated politicians when the user says `/list-politicians`.

---

## Tool Usage Rules

This Skill runs in the Claude Code environment with the following tools:

| Task | Tool |
|------|------|
| Read PDF documents (speeches, policy papers) | `Read` tool (native PDF support) |
| Read image screenshots (voting records, social media) | `Read` tool (native image support) |
| Read MD/TXT files | `Read` tool |
| Parse social media JSON exports | `Bash` → `python3` script |
| Write/update Skill files | `Write` / `Edit` tool |
| Version management | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py` |
| List existing Skills | `Bash` → `python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./politicians` |

**Base directory**: Skill files are written to `./politicians/{slug}/` (relative to the project directory).

---

## Main Flow: Create a New Politician Skill

### Step 1: Basic Info Collection (3 questions)

Refer to `${CLAUDE_SKILL_DIR}/prompts/politician_intake.md` for the question sequence. Only ask 3 questions:

1. **Alias / Codename** (required)
2. **Basic info** (one sentence: country, party, position, active era)
   - Example: `USA Republican former-president 2016-2024`
3. **Political profile** (one sentence: ideology, political spectrum, style tags, impression)
   - Example: `right-wing populist hawk Twitter-governance businessman shock-value`

Everything except the alias can be skipped. Summarize and confirm before moving to the next step.

### Step 2: Source Material Import

Ask the user how they'd like to provide materials:

```
How would you like to provide source materials?

  [A] Upload Files
      PDF (speeches, policy papers, biography excerpts)
      Images (voting record screenshots, social media screenshots)
      TXT/MD (organized materials)

  [B] Provide Web Links
      Wikipedia pages, news articles, voting record databases
      (I'll try to fetch the content)

  [C] Paste Text
      Paste speeches, interview transcripts, voting records directly

  [D] Specify Public Data Sources
      Tell me where to look (e.g., congress.gov, Hansard)
      I'll guide you to provide specific materials

Can mix and match, or skip entirely (generate from manual info only).
```

---

#### Source Material Guide

Based on the type of materials the user provides, suggest the most valuable supplements:

**Core Sources (high priority)**:
1. **Public speeches & debate transcripts** — extract rhetorical style, core narratives, emotional tone
2. **Legislative voting records** — hardest data, reveals true interest alignment and policy positions
3. **Social media posts** — capture instinctive reactions, informal expression, crisis response

**Supplementary Sources (medium priority)**:
4. **Interview & press conference transcripts** — impromptu responses under pressure
5. **Biographies / memoirs** — self-narrative and value frameworks
6. **Policy papers / campaign platforms** — formal position statements

**Validation Sources (low priority but useful)**:
7. **Media commentary / analysis** — external perspectives, note potential bias
8. **Polling data** — public perception
9. **Opponent attacks** — reverse-engineer weaknesses

#### Political Asset Mapping

Replace the original "colleague weekly reports / PRDs / chat logs" with "politician political assets":

| Colleague Skill source | Politician Skill equivalent | Primary extraction target |
|------------------------|-----------------------------|---------------------------|
| Weekly reports / retrospectives | Legislative votes, bills, executive decisions | Real interest alignment, policy red lines, position evolution |
| Meeting comments | Public speeches, debate transcripts, hearings | Rhetorical style, narrative frames, attack/defense tactics |
| Chats / DMs | Social media, interviews, press conferences | Instinctive reactions, informal voice, pressure response |
| Peer feedback | Media commentary, opponent attacks, polling | External perception, weaknesses, contested labels |

#### Evidence Weighting Rules

Resolve conflicts using this priority order:

1. **Legislative votes / executive decisions**: highest weight. When slogans conflict with votes, use votes and decisions as the baseline.
2. **Policy papers / campaign platforms**: formal positions. Use them to cover issues not represented in votes.
3. **Public speeches / debates / hearings**: rhetoric and positioning. Use them for language style and narrative frames.
4. **Social media / interviews / press conferences**: immediate reactions. Use them for instinct, emotion, and crisis behavior.
5. **Media commentary / opponent attacks / polling**: validation or reverse inference only; never use as sole support for a core conclusion.

Every key conclusion must include source type and confidence:
- `hard_evidence`: votes, formal decisions, published policy documents
- `public_statement`: speeches, debates, interviews, social posts
- `external_observation`: media, polling, opponent claims
- `inference`: derived from tags or patterns when evidence is thin; must be marked explicitly

---

#### Option A: Upload Files

- **PDF**: `Read` tool directly (speeches, white papers, biographies)
- **Images**: `Read` tool directly (screenshots, voting record charts)
- **TXT/MD**: `Read` tool directly

---

#### Option B: Web Links

When the user provides web links, attempt to fetch content with `WebFetch`. If unable to read, guide user to manually copy-paste.

---

#### Option C: Paste Text

User-pasted content is used directly as text material. No tools needed.

---

#### Option D: Specify Data Sources

Guide the user to obtain materials from public data sources:

| Country/Region | Voting Records | Speeches/Debates |
|---------------|---------------|-----------------|
| USA | congress.gov, govtrack.us | c-span.org, rev.com |
| Taiwan | Legislative Yuan Gazette System | Legislative Yuan Live Stream |
| UK | hansard.parliament.uk | parliamentlive.tv |
| EU | europarl.europa.eu | ep.europa.eu |

---

If the user says "no materials" or "skip", generate Skill from Step 1 manual info only.

### Step 3: Analyze Source Material

Combine all collected materials and user-provided info, analyze along two tracks:

**Track A (Political Capability)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/political_analyzer.md` for extraction dimensions
- Extract: policy positions, voting records, rhetorical capability, political operations
- Emphasize different aspects by position type (head of state/legislator/local leader/opposition)
- Build a political asset ledger: every core position should link to a vote, decision, speech, or interview source
- When voting records and public rhetoric conflict, mark it as "interest alignment vs public narrative" instead of smoothing over the contradiction

**Track B (Political Persona)**:
- Refer to `${CLAUDE_SKILL_DIR}/prompts/politician_persona_analyzer.md` for extraction dimensions
- Translate user-provided tags into concrete behavior rules (see tag translation table)
- Extract from materials: public expression style, decision patterns, political interpersonal behavior
- Write tags in searchable form, such as `#realist`, `#populist`, `#hawk`, `#dove`
- Provide spectrum coordinates and evidence: economic left/right and authority/liberty axes; mark confidence when uncertain

### Step 4: Generate and Preview

Use `${CLAUDE_SKILL_DIR}/prompts/political_builder.md` to generate Political Capability content.
Use `${CLAUDE_SKILL_DIR}/prompts/politician_persona_builder.md` to generate Persona content (6-layer structure).

Show the user a summary (5-8 lines each), ask:
```
Political Capability Summary:
  - Core positions: {xxx}
  - Rhetorical style: {xxx}
  - Political operations: {xxx}
  - Hardest evidence: {votes/decisions/policy documents}
  ...

Persona Summary:
  - Core personality: {xxx}
  - Expression style: {xxx}
  - Decision pattern: {xxx}
  - Political spectrum: Economic {X}/5, Authority {Y}/5
  - Tags: #{tag1} #{tag2} #{tag3}
  ...

Confirm generation? Or need adjustments?
```

### Step 5: Write Files

After user confirmation, execute the following:

**1. Create directory structure** (Bash):
```bash
mkdir -p politicians/{slug}/versions
mkdir -p politicians/{slug}/sources/speeches
mkdir -p politicians/{slug}/sources/votes
mkdir -p politicians/{slug}/sources/media
mkdir -p politicians/{slug}/sources/interviews
mkdir -p politicians/{slug}/sources/policies
```

**2. Write political.md** (Write tool):
Path: `politicians/{slug}/political.md`

**3. Write persona.md** (Write tool):
Path: `politicians/{slug}/persona.md`

**4. Write meta.json** (Write tool):
Path: `politicians/{slug}/meta.json`
Content:
```json
{
  "name": "{name}",
  "slug": "{slug}",
  "created_at": "{ISO_timestamp}",
  "updated_at": "{ISO_timestamp}",
  "version": "v1",
  "type": "politician",
  "profile": {
    "country": "{country}",
    "party": "{party}",
    "position": "{position}",
    "era": "{era}"
  },
  "spectrum": {
    "economic": {X},
    "authority": {Y}
  },
  "tags": {
    "ideology": [...],
    "style": [...]
  },
  "source_weights": {
    "votes": "highest",
    "policy_documents": "high",
    "speeches_debates": "medium",
    "social_interviews": "medium",
    "media_opponents_polls": "validation_only"
  },
  "evidence_counts": {
    "hard_evidence": 0,
    "public_statements": 0,
    "external_observations": 0,
    "inferences": 0
  },
  "impression": "{impression}",
  "knowledge_sources": [...imported file list],
  "corrections_count": 0
}
```

**5. Generate full SKILL.md** (Write tool):
Path: `politicians/{slug}/SKILL.md`

SKILL.md structure:
```markdown
---
name: politician-{slug}
description: {name}, {country} {party} {position}
user-invocable: true
---

# {name}

{country} {party} {position}
Political Spectrum: Economic {X}/5, Authority {Y}/5
{ideology tag list}

---

## PART A: Political Capability

{full political.md content}

---

## PART B: Political Persona

{full persona.md content}

---

## PART C: Evidence and Boundaries

- Highest-weight evidence: {voting records / executive decisions / policy document summary}
- Public statement sources: {speeches / debates / interviews / social media summary}
- External observation sources: {media / polling / opponent attack summary}
- Inferred items: {all rules inferred only from tags or patterns without hard evidence}
- Prohibitions: do not pretend to have unseen materials; do not present external commentary as the politician's own position

---

## Execution Rules

1. PART B decides first: what attitude and stance to take on this issue?
2. PART A executes: use your political capability and rhetorical arsenal to craft a response
3. Always maintain PART B's expression style in output
4. PART B Layer 0 rules have the highest priority and must never be violated
5. When responding to policy issues, must align with PART A's policy positions and voting record
6. When votes/decisions conflict with slogans, use votes/decisions as the stance baseline and slogans as rhetorical packaging
7. Content without evidence may only be marked as inference; never present it as fact
```

Inform user:
```
✅ Politician Skill created!

Location: politicians/{slug}/
Commands: /{slug} (full version)
          /{slug}-political (political capability only)
          /{slug}-persona (persona only)

If something feels off, just say "he wouldn't say that" and I'll update it.

⚠️ Note: This Skill simulates the public behavioral patterns of this political figure.
It does not constitute an endorsement or recommendation of any political position.
```

---

## Evolution Mode: Append Materials

When user provides new materials:

1. Read new content using Step 2 methods
2. `Read` existing `politicians/{slug}/political.md` and `persona.md`
3. Refer to `${CLAUDE_SKILL_DIR}/prompts/politician_merger.md` for incremental analysis
4. Archive current version (Bash):
   ```bash
   python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action backup --slug {slug} --base-dir ./politicians
   ```
5. Use `Edit` tool to append incremental content to relevant files
6. Regenerate `SKILL.md` (merge latest political.md + persona.md)
7. Update `meta.json` version and updated_at

---

## Evolution Mode: Conversation Correction

When user expresses "that's wrong" / "his position should be":

1. Refer to `${CLAUDE_SKILL_DIR}/prompts/politician_correction_handler.md` to identify correction content
2. Determine if it belongs to Political (policy/rhetoric) or Persona (personality/style)
3. Generate correction record
4. Use `Edit` tool to append to the `## Correction Log` section of the relevant file
5. Regenerate `SKILL.md`

---

## Management Commands

`/list-politicians`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/skill_writer.py --action list --base-dir ./politicians
```

`/politician-rollback {slug} {version}`:
```bash
python3 ${CLAUDE_SKILL_DIR}/tools/version_manager.py --action rollback --slug {slug} --version {version} --base-dir ./politicians
```

`/delete-politician {slug}`:
After confirmation:
```bash
rm -rf politicians/{slug}
```
