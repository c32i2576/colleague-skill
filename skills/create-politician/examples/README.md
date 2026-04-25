# Examples — 預先蒸餾的政治人物 Skill

本目錄收錄使用 v2 流程產出的政治人物 Skill 範例，供：
1. 新用戶快速上手：直接 copy 到 `politicians/{slug}/` 使用
2. 模板參考：看完整的 SKILL.md 實際長什麼樣
3. 三重驗證對照：看「心智模型」實際怎麼寫

---

## 目錄結構（每個範例）

```
examples/{slug}/
├── SKILL.md          # 完整產出（可直接運行）
├── political.md      # PART A/B/D 片段
├── persona.md        # PART C
├── limitations.md    # 誠實邊界
├── meta.json
└── research/         # 六軌原料（摘要版）
```

---

## 目前範例

（尚未填充 — 使用 `/create-politician {name}` 以 v2 流程生成首個範例後，移至此處。）

建議優先生成的對比組合：
- 一位**意識形態強烈**的（如：陳水扁、柯文哲）— 心智模型應清晰
- 一位**務實派**的（如：朱立倫、賴清德）— 邊界與張力應明顯
- 一位**語言風格鮮明**的（如：韓國瑜、川普）— Expression DNA 應易量化

---

## 如何使用範例

```bash
# 複製到專案的 politicians/ 目錄
cp -r examples/{slug} politicians/{slug}

# 在 Claude Code 載入
/{slug}
```

## 貢獻範例

若你產出了高品質 Skill，歡迎 PR：
1. 跑完 v2 完整流程（包含 Phase 4 三項驗證）
2. 清理 research/ 為可公開版本
3. 確認 limitations.md 明確宣告邊界
4. 附一個 `HOW_IT_WAS_BUILT.md` 說明採用哪些軌道、是否有特殊處理
