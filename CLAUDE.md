# 一天一AI — 文章摘要專案

數位時代「一天一AI」專欄的文章索引，以互動式 HTML 捲動式單頁文件呈現。

---

## 檔案結構

```
D:\one_day_one_ai\
├── CLAUDE.md                  # 本文件
├── onedayoneai.md              # 原始資料（主要資料來源）
└── onedayoneai.html            # 互動式捲動頁面（從 MD 同步）
```

---

## 更新指令

當使用者說「請更新」、「更新文章」或類似指令時，依序執行以下流程：

### 第一步：抓取最新文章

用 WebFetch 從第 1 頁起依序抓取, 每頁提取所有文章的標題, 發布時間, 連結 URL, 直到該頁無文章或回傳 404 為止(2026-09-18 時共 6 頁):

- `https://www.bnext.com.tw/tags/%E4%B8%80%E5%A4%A9%E4%B8%80AI`
- `https://www.bnext.com.tw/tags/%E4%B8%80%E5%A4%A9%E4%B8%80AI?page=2`
- ...

### 第二步: 比對新舊, 識別新增文章

將抓到的文章 URL 與 `onedayoneai.md` 主題分類清單中的連結比對. 主題分類清單是唯一的完整文章清單, 每篇文章至少屬於一個分類.
URL 中的 article ID(如 `/article/91888/`)為唯一識別碼, 不在清單中的即為新文章, 依網站排列順序記錄標題, 發布時間, 連結備用.

### 第三步: 更新 MD 檔案

更新 `onedayoneai.md`:

1. 檔頭: 更新整理日期(今日), 文章總數
2. 更新摘要區塊: 更新總數, 新增數, 更新日期
3. 更新紀錄: 只保留最近 2 次更新
   - 刪除原本的「前次新增」區塊
   - 原本的「本次新增」標題改為「前次新增」
   - 在最上方新增「<今日日期> 本次新增 (N 篇)」區塊, 表格為 `| 標題 | 發布時間 | 分類 | 連結 |`, 分類欄以 ` / ` 分隔多個分類
   - 若本次無新文章, 不新增區塊, 也不輪替
4. 主題分類: 新文章以 `- [標題](URL)` 格式插入對應分類清單頂端; 若出現明顯新主題, 可新增分類

### 第四步: 更新 HTML 檔案

更新 `onedayoneai.html`, 與 MD 保持同步:

1. `<header>`: 更新整理日期與文章總數
2. 更新摘要 `<section>`: 更新四個 `.field` 卡片(總數, 新增數, 分類數, 日期)
3. 更新紀錄 `<section>`: 與 MD 相同的輪替方式, 刪除前次區塊, 原本次區塊的 `<h3>` 改為前次並移除該表格所有 `class="is-new"`, 再於 `.section-note` 之後插入新的本次區塊, 列加上 `class="is-new"`
4. 主題分類 `<section>`: 新文章插入對應 `.cat-panel` 內 `<ul>` 的頂端(`<li>` 必須有 `<a href="..." target="_blank">` 連結), 並同步更新對應 `.cat-tab` 上的 `<span class="count">` 篇數

### 第五步：Commit 與 Push (Auto Mode)

內容更新完成後, 若使用者是以 "Auto Mode" 下達 "請更新" 指令(例如同時對 bnext-ai-news 與 one_day_one_ai 兩個專案執行), 則不需再次詢問確認, 直接執行:

1. `git add onedayoneai.md onedayoneai.html`
2. `git commit -m "更新文章: <新增篇數> 篇 (<今日日期>)"`
3. `git push origin main`

若非 Auto Mode(例如使用者僅在本專案單獨對話中說 "請更新"), 則依一般流程, 在 commit/push 前先向使用者確認.

---

## HTML 設計規範

### 整體架構

單一可捲動頁面，不分投影片，結構固定為：

```html
<body>
  <button id="themeToggle">淺色模式</button>
  <header>...封面資訊...</header>
  <main>
    <section>更新摘要</section>
    <section>更新紀錄</section>
    <section>主題分類</section>
  </main>
</body>
```

`header` 置中呈現標題、副標題與來源／日期／篇數資訊；`main` 內每個 `section` 依序往下捲動，不使用任何分頁、投影片或全螢幕邏輯。

### 更新紀錄格式

`<section>` 內含一個 `<h2>更新紀錄</h2>`, 一段 `.section-note` 說明, 以及最多 2 個日期區塊(本次在上, 前次在下). 表格為 4 欄, 無編號欄:

```html
<section>
  <h2>更新紀錄</h2>
  <p class="section-note">保留最近 2 次更新新增的文章, 發布時間為抓取當下的相對時間.</p>
  <h3>2026-09-18 本次新增 (3 篇)</h3>
  <div class="table-wrap">
    <table>
      <thead>
        <tr><th>標題</th><th>發布時間</th><th>分類</th><th>連結</th></tr>
      </thead>
      <tbody>
        <tr class="is-new"><td>文章標題</td><td>發布時間</td><td>分類A / 分類B</td><td><a href="..." target="_blank">閱讀</a></td></tr>
      </tbody>
    </table>
  </div>
  <h3>2026-09-14 前次新增 (2 篇)</h3>
  <div class="table-wrap">...</div>
</section>
```

分類欄使用 `.cat-tab` 上的分類短名稱.

### 色彩系統（CSS 變數）

採用 Anthropic / Claude 品牌配色，以 CSS 變數管理：

```css
/* Dark mode（預設） */
--bg:           #1C1917;   /* 暖棕黑背景 */
--bg-card:      #292524;   /* 卡片背景 */
--bg-hover:     #302D2A;
--bg-thead:     #242120;
--border:       #3D3833;
--border-row:   #2F2C29;
--text-primary: #E8E3DF;
--text-secondary:#A9A09A;
--text-muted:   #6B6560;
--accent:       #D97757;   /* Claude 銅橘色 */
--accent-hover: #E88A65;
--accent-dim:   rgba(217,119,87,0.18);

/* Light mode（body.light） */
--bg:           #FAF9F7;   /* 暖米白 */
--bg-card:      #F0EDE8;
--accent:       #C86A40;   /* 銅橘深色版（光背景對比用） */
```

禁止使用純黑（`#000`）、純紅（`#ff0000`）或冷調灰藍色系。

### 主題切換

右上角固定 `#themeToggle` 按鈕，純文字（不用 emoji），文字在「淺色模式」／「深色模式」間切換，狀態存於 `localStorage`，預設為 dark mode。

### 更新摘要卡片格式

四個 `.field` 卡片放在 `.api-fields` grid 內：

```html
<div class="api-fields">
  <div class="field">
    <span class="name">102</span>
    <div class="desc">篇文章</div>
  </div>
</div>
```

### 主題分類 nav 格式

主題分類採分類標籤 nav 加篩選：上方一排 `.cat-tab` 按鈕（`.cat-tabs` 內），下方對應 `.cat-panel` 面板（`.cat-panels` 內），同一時間只顯示一個作用中面板，透過 JS 依 `data-target` 與面板 `id` 對應切換：

```html
<div class="cat-tabs" role="tablist">
  <button type="button" class="cat-tab is-active" data-target="cat-prompt" role="tab" aria-selected="true">分類名稱<span class="count">19</span></button>
  <button type="button" class="cat-tab" data-target="cat-tools" role="tab" aria-selected="false">分類名稱<span class="count">30</span></button>
</div>
<div class="cat-panels">
  <div class="cat-panel is-active" id="cat-prompt" role="tabpanel">
    <ul>
      <li><a href="https://www.bnext.com.tw/article/..." target="_blank">文章標題</a></li>
    </ul>
  </div>
  <div class="cat-panel" id="cat-tools" role="tabpanel">
    <ul>...</ul>
  </div>
</div>
```

新增文章到既有分類時，於對應 `.cat-panel` 的 `<ul>` 內插入 `<li>`，並將該分類 `.cat-tab` 的 `<span class="count">` 數字加一；新增分類時需同時新增一個 `.cat-tab` 按鈕與對應 `.cat-panel`，`data-target` 與 `id` 需一致。

### 新文章標示

只有「本次新增」表格的列加 `class="is-new"`, 會顯示左側銅橘色邊框. 輪替成「前次新增」時必須移除:

```html
<tr class="is-new"><td>文章標題</td><td>發布時間</td><td>分類</td><td>...</td></tr>
```

---

## 主題分類（8 大類）

新文章依內容歸入對應分類，一篇文章可跨多個分類：

| 分類 | 關鍵判斷 |
|------|---------|
| 提示詞技巧 | 教如何下指令、改善 prompt、與 AI 溝通技巧 |
| 工具應用 | 介紹特定工具（NotebookLM、Claude、Gemini、Canva 等）的具體操作 |
| 職場效率 | 工作流程自動化、職場任務加速、商務應用 |
| 翻譯與語言 | 中英翻譯、口說、去 AI 味、文藻翻譯系主任系列 |
| 學習與個人成長 | 學習方法、腦力訓練、自我提升 |
| 醫療與健康 | 醫師、健檢、用藥、健康數據相關 |
| 生活應用 | 家庭、照片、求職、個人生活場景 |
| AI 隱私安全 ＆ 企業策略 | 個資保護、API 安全、企業導入、商業決策 |

若有明顯新主題（例如未來出現「AI 法律」系列），可新增分類並更新主題分類 section。

---

## 注意事項

- MD 與 HTML 兩個檔案必須同步更新，不可只更新其中一個
- 不維護網站頁次表格, 完整文章清單以主題分類為準, 每篇文章至少歸入一個分類
- 更新紀錄只保留最近 2 次(本次與前次), 更早的紀錄直接刪除
- 發布時間保留抓取當下網站原始的相對時間(「3天前」, 「1個月前」), 不轉換為絕對日期, 只出現在更新紀錄中
- 每次更新後，更新摘要 section 的「新增數」顯示本次新增篇數（非累計）
