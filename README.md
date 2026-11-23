# 📊 上市櫃公司法說會 LLM Wiki 知識庫

> 採用 Andrej Karpathy 的 **LLM Wiki** 模式，將非結構化的上市櫃公司法說會簡報（`raw/`），持續編譯並維護為跨公司橫向連動、跨季度縱向追蹤的 Markdown 知識庫（`wiki/`），並生成新手友善的互動式教學 HTML 簡報（`slides/`）。

## 🌐 線上成果展示
- **教學型 HTML 簡報（GitHub Pages）**：`https://<你的帳號>.github.io/<專案名稱>/slides/`
- **核心營運規範**：[`AGENTS.md`](./AGENTS.md)
- **知識庫總目錄**：[`index.md`](./index.md)
- **增量操作與檢核日誌**：[`log.md`](./log.md)

## 🏗️ 專案目錄架構
```text
/
├── AGENTS.md                  # [Schema] AI 營運與擴充最高指導規範
├── README.md                  # 專案說明文件
├── index.md                   # [Catalog] 知識庫總目錄（每次異動必同步更新）
├── log.md                     # [Timeline] 增量操作日誌（記錄每次 ingest/query/lint）
├── raw/                       # [Raw Sources] 原始法說會簡報 PDF（唯讀唯一事實來源）
│   └── 2026-Q3/               # 2026 Q3 五大 AI 供應鏈簡報（tsmc, mediatek, ase, honhai, delta）
├── wiki/                      # [Wiki Layer] LLM 編譯維護的 Markdown 知識庫
│   ├── companies/             # 個別公司長期追蹤檔（2330, 2454, 3711, 2317, 2308）
│   ├── comparisons/           # 跨公司橫向比較（2026 Q3 AI 供應鏈五強總覽）
│   └── concepts/              # 法說會核心財務與產業指標百科
└── slides/                    # [Outputs] 基於 Wiki 生成的簡潔學術白底風互動簡報
    └── index.html
```

## 🔄 核心設計特色
1. **不可變事實來源（Single Source of Truth）**：`raw/` 保留原始法說會 PDF，所有數據萃取與圖表轉表格皆經內部檢核迴圈驗證。
2. **單位統一與防呆機制**：自動識別並統一「新台幣十億元（台積電、台達電）」與「新台幣百萬元（聯發科、日月光、鴻海）」的數量級差異，杜絕跨公司比較時的錯置。
3. **可持續擴充性**：不綁定單一年度或固定公司清單。未來只需將新季度（如 `raw/2026-Q4/`）或新公司簡報放入 `raw/`，AI 即可依據 `AGENTS.md` 自動增量更新知識庫並重新生成教學簡報。