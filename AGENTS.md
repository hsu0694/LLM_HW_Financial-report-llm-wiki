# AGENTS.md — 上市櫃公司法說會 LLM Wiki 營運規範

## 1. 專案願景與核心定位 (Core Vision)
本專案採用 Andrej Karpathy 的 **LLM Wiki** 模式，建置一個**可持續累積、跨公司橫向連動、跨季度縱向追蹤**的「台灣上市櫃公司法說會知識庫」。
- **核心原則**：原始法說會簡報（`raw/`）為不可變的唯一事實來源（Single Source of Truth）；AI Agent 負責將非結構化的法說會簡報與圖表，編譯並持續維護成高度互連的 Markdown 知識庫（`wiki/`）。
- **擴充性設計**：本架構不綁定單一年度、單一季度或固定公司清單。未來每新增一份法說會檔案，Agent 必須依照本規範增量更新現有公司檔案、新增季度摘要、刷新跨公司比較表，並可隨時基於最新知識庫生成教學型或分析型簡報。

---

## 2. 目錄結構規範 (Directory Architecture)

```text
/
├── AGENTS.md                  # [Schema] 本規範文件（最高指導原則）
├── README.md                  # [Docs] 專案說明與線上展示入口
├── index.md                   # [Catalog] 知識庫總目錄（依分類索引，每次異動必更新）
├── log.md                     # [Timeline] 增量操作日誌（Append-only，記錄每次 ingest/query/lint）
├── raw/                       # [Raw Sources] 原始法說會簡報 PDF（唯讀，嚴禁修改或刪除）
│   └── YYYY-QN/               # 依年份季度歸檔（如 2026-Q3/tsmc.pdf）
├── wiki/                      # [Wiki Layer] LLM 編譯與維護的 Markdown 知識庫
│   ├── companies/             # 個別公司長期追蹤檔（如 2330-tsmc.md、2454-mediatek.md）
│   ├── quarters/              # 單季法說會深度摘要（如 2026-q3-tsmc.md）
│   ├── comparisons/           # 跨公司/產業橫向比較（如 2026-q3-ai-supply-chain.md）
│   └── concepts/              # 法說會財務與產業名詞百科（如 gross-margin.md、yoy-qoq.md）
└── slides/                    # [Outputs] 基於 Wiki 生成的 HTML 教學簡報（供 GitHub Pages 展示）
    └── index.html             # 簡潔學術白底風互動簡報
```

---

## 3. 文件格式與連結規範 (Conventions & Frontmatter)

### 3.1 YAML Frontmatter（每篇 `wiki/` 檔案必備）
所有位於 `wiki/` 底下的 `.md` 檔案開頭必須包含標準 YAML 標頭，便於未來檢索與追蹤：
```yaml
---
title: "頁面標題"
category: company | quarter | comparison | concept
ticker: "股票代號（若適用，如 2330）"
updated_at: "YYYY-MM-DD"
sources: ["raw/2026-Q3/tsmc.pdf"]
tags: ["法說會", "2026Q3", "半導體"]
---
```

### 3.2 雙向連結與圖表轉譯規則
- **Wiki 內部連結**：提及任何已建立的公司、季度或財務名詞時，一律使用相對連結或 Wiki 連結格式（例如 `[毛利率](../concepts/gross-margin.md)` 或 `[[gross-margin]]`），避免產生孤島頁面（Orphan Pages）。
- **圖表轉結構化表格（Chart-to-Table Normalization）**：法說會簡報包含大量長條圖、圓餅圖與折線圖。Agent 在讀取圖表時，**嚴禁只寫模糊形容詞**，必須將圖表數據還原為標準 Markdown 表格，並明確標註**幣別與單位**（如：新台幣百萬元、新台幣十億元、美金十億元）。

---

## 4. 核心工作流與迴圈設計 (Core Workflows & Loops)

### 4.1 Ingest Loop（新法說會資料匯入迴圈）
當使用者提供新的法說會簡報（放入 `raw/`）時，Agent **必須先與使用者討論重點，確認後再執行寫入**，完整步驟如下：
1. **Extract（資訊與圖表萃取）**：讀取 PDF 文字與圖表，提取「核心財務三表數據、產品/平台營收佔比、資本支出、下季/全年業績展望、重大事件」。
2. **Unit & Math Check（內部檢核迴圈 — 關鍵防錯節點）**：
   - *檢查點 A（單位統一）*：確認原始簡報單位是「新台幣百萬元（如聯發科、日月光、鴻海）」或「新台幣十億元（如台積電、台達電）」，換算或並列時不可錯置數量級。
   - *檢查點 B（加總與計算驗證）*：驗證產品組合圓餅圖佔比加總是否約為 100%、驗證「營業毛利 ÷ 營業收入 = 毛利率」是否與簡報一致。**若發現誤差，退回重新讀取原頁圖表直到吻合。**
3. **Discuss（先討論、後生成）**：向使用者列出本份法說會的 3～5 個核心亮點與異常訊號，確認無誤後才寫入檔案。
4. **Compile & Upsert（增量編譯寫入）**：
   - 新增 `wiki/quarters/<year>-<quarter>-<company>.md`。
   - **增量更新** `wiki/companies/<ticker>-<company>.md`（保留歷史季度表格，將最新一季追加至最上方並更新長期趨勢分析）。
   - 若涉及新財務指標或產業術語，建立或補充 `wiki/concepts/<term>.md`。
   - 更新 `wiki/comparisons/` 下的跨公司橫向對比表。
5. **Catalog & Log（索引與日誌同步）**：更新 `index.md` 目錄摘要，並在 `log.md` 末尾追加一筆 `## [YYYY-MM-DD] ingest | <公司名稱> <季度> 法說會` 紀錄。

### 4.2 Query & Slide Generation Loop（教學 HTML 簡報生成迴圈）
當使用者要求針對特定主題或最新知識庫生成教學簡報時，**嚴禁直接讀取 `raw/` 原始 PDF 憑印象發揮**，必須完全以 `wiki/` 為唯一知識來源：
1. **Plan & Retrieve（大綱規劃與檢索）**：先讀取 `index.md` 鎖定相關概念頁與公司頁，規劃「新手友善」的教學脈絡（概念定義 $\rightarrow$ 指標判讀方法 $\rightarrow$ 真實企業案例對比 $\rightarrow$ 實戰觀察重點）。
2. **Draft Content（撰寫簡報腳本）**：將教學觀念與 `wiki/` 中的真實數據轉化為結構化投影片內容（每頁聚焦一個核心觀念，避免文字牆）。
3. **Fact & Pedagogical Check（雙重品質檢查迴圈）**：
   - *事實一致性檢查*：逐一比對簡報上的每一個財務數字與 `wiki/` 頁面是否 100% 一致，杜絕幻覺。
   - *教學可讀性檢查*：檢查是否使用白話解釋專業術語（如 YoY、QoQ、毛利率、營業利益率），確保未接觸過法說會的讀者也能看懂。**若不通過，退回重寫該頁投影片。**
4. **Render HTML（視覺渲染與前端防呆規範）**：
   - 輸出單一自包含（Self-contained）的 `slides/index.html`，採用**簡潔學術白底風**（高對比深灰文字、清晰數據表格、顯示頁碼與進度條）。
   - **前端防呆編碼準則（Shift-Left UI Safeguards）**：由於 LLM 無法在生成階段直接啟動瀏覽器執行測試，輸出 HTML/JS 時必須在程式碼生成當下直接遵守以下防呆規範以規避跨環境相容性問題：
     1. 禁止使用行內事件屬性（如 `onclick="..."`），一律在 `DOMContentLoaded` 內透過標準 `addEventListener` 綁定按鈕、章節選單、鍵盤左右方向鍵與行動裝置觸控滑動事件。
     2. 每張投影片須配置獨立語意化 `id`（如 `#slide-1`），確保具備基礎錨點結構與無障礙相容性。
     3. 所有 HTML 標籤（含 `<script>` 與 `</html>`）必須完整閉合，嚴禁截斷。
   - 生成完畢後，將操作紀錄寫入 `log.md`。

### 4.3 Lint Loop（知識庫健康檢查迴圈）
每次累計新增 3～5 份資料後，或生成重要簡報前，執行一次全庫體檢：
- 檢查不同公司頁面間的**單位標示**是否清晰（避免十億與百萬混淆）。
- 檢查是否存在沒有連入連結的**孤島頁面（Orphan Pages）**。
- 檢查跨公司比較表是否已納入最新匯入的所有公司。
- 將修復結果記錄至 `log.md`。