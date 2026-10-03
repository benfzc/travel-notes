# 旅遊寶典 — 靜態 Markdown 網站

> 用純靜態 HTML/JavaScript 將 Markdown 旅遊筆記渲染成可瀏覽的網站。  
> 無需建置工具、無需框架，推上 GitHub 後由 Cloudflare Pages 自動部署。

## 目錄結構

```
travel-notes/
├── index.html          ← 主程式（通常不需修改）
├── pages.json          ← 頁面設定檔（新增 md 時改這裡）
├── .gitignore
├── README.md           ← 你正在看的這份文件
└── docs/               ← 所有 markdown 文件放這裡
    ├── 2026_japan_ginkgo_trip.md
    ├── kyushu_onsen.md          ← 範例：未來新增
    └── hokkaido_winter.md       ← 範例：未來新增
```

## 本地預覽

瀏覽器安全限制不允許 `file://` 直接載入本地檔案，需透過 HTTP 伺服器：

```powershell
cd D:\SideProject\travel-notes
python -m http.server 8000
```

然後瀏覽器開啟 **http://localhost:8000**

---

## 部署 SOP：GitHub + Cloudflare Pages

### Step 1：建立 GitHub Repo

#### 1.1 在 GitHub 建立新 Repository

1. 前往 [github.com/new](https://github.com/new)
2. Repository name：`travel-notes`（或任何你喜歡的名稱）
3. 選擇 **Private**（旅遊資料建議私有；Cloudflare Pages 支援 Private repo）
4. **不要**勾選「Add a README」（我們已有檔案）
5. 點擊 **Create repository**

#### 1.2 在本地初始化 Git 並推送

```powershell
cd D:\SideProject\travel-notes

git init
git add .
git commit -m "Initial commit: static markdown travel guide"

git remote add origin https://github.com/<你的帳號>/travel-notes.git
git branch -M main
git push -u origin main
```

> 將 `<你的帳號>` 替換為你的 GitHub 使用者名稱。

---

### Step 2：Cloudflare Pages 設定

#### 2.1 連接 GitHub

1. 登入 [Cloudflare Dashboard](https://dash.cloudflare.com/)
2. 左側選單 → **Workers & Pages**
3. 點擊 **Create** → 選擇 **Pages** 分頁 → **Connect to Git**
4. 授權 Cloudflare 存取你的 GitHub 帳號
5. 選擇 `travel-notes` repository

#### 2.2 建置設定（純靜態，幾乎不用設定）

| 設定項目 | 值 |
|---------|-----|
| **Production branch** | `main` |
| **Framework preset** | `None` |
| **Build command** | _(留空)_ |
| **Build output directory** | _(留空，或填 `/`)_ |

6. 點擊 **Save and Deploy**

#### 2.3 部署完成

- Cloudflare 會給你一個免費子網域，例如：`https://travel-notes.pages.dev`
- 之後每次 `git push` 到 `main`，Cloudflare 會自動重新部署（通常 < 30 秒）

#### 2.4（選用）綁定自訂網域

1. 進入你的 Pages 專案 → **Custom domains**
2. 點擊 **Set up a custom domain**
3. 輸入你的網域（需事先在 Cloudflare 管理 DNS）
4. Cloudflare 會自動處理 SSL 憑證

---

### Step 3：日常使用 — 新增一篇旅遊文件

只需 **兩步 + 一個 push**：

#### 3.1 新增 markdown 檔案

把新的 `.md` 檔案放進 `docs/` 資料夾：

```
docs/kyushu_onsen.md
```

#### 3.2 更新 pages.json

在 `pages` 陣列中加一個條目：

```json
{
  "site": {
    "title": "旅遊寶典",
    "icon": ""
  },
  "pages": [
    {
      "file": "docs/japan_trip.md",
      "title": "日本賞銀杏與懷舊體驗",
      "description": "全日本旅遊寶典（依機場分類）"
    },
    {
      "file": "docs/kyushu_onsen.md",
      "title": "九州溫泉巡禮",
      "description": "別府、由布院、黑川溫泉攻略"
    }
  ]
}
```

> 超過一個頁面時，側邊欄會自動出現頁面導覽列表。

#### 3.3 推送

```powershell
git add .
git commit -m "Add kyushu onsen guide"
git push
```

Cloudflare Pages 自動偵測 → 重新部署 → 完成。

---

## 注意事項

### 檔名規範
- markdown 檔名建議用 **小寫英文 + 底線**（例如 `kyushu_onsen.md`）
- 避免中文檔名，以防 URL 編碼問題

### Cloudflare Pages 免費額度（非常夠用）
- 每月 **500 次**建置
- **無限制**頻寬與請求數
- **免費** SSL 憑證

### 隱私考量
- GitHub 設為 **Private** → 原始碼不公開
- 但 Cloudflare Pages 部署後的**網站本身是公開的**（知道網址就能看）
- 若需限制存取，可用 Cloudflare **Access** 設定密碼保護（免費方案支援 50 位使用者）

---

## 技術細節

| 技術 | 說明 |
|------|------|
| [marked.js](https://github.com/markedjs/marked) | 瀏覽器端 Markdown → HTML 解析 |
| 路由 | Hash-based（`#docs_xxx_md`），可分享特定頁面連結 |
| 目錄導覽 | 自動從 h2/h3 標題生成，Scroll Spy 自動高亮 |
| 響應式 | 手機版側邊欄自動收合，漢堡選單開關 |
| CDN | marked.js 透過 jsDelivr CDN 載入 |
