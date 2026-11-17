# 快速開始

[English](../getting-started.md)

Refine Lab 是模組化的 WordPress 工具組：用純 HTML 範本建立原生 Gutenberg 區塊、表單、多語言內容、SEO、安全性、郵件寄送、備份與日常小工具。只需開啟你需要的部分，每個模組都能關閉。

## 系統需求

| | |
| :--- | :--- |
| WordPress | 7.0 或更新版本（單一網站；多站網路不提供完整備份） |
| PHP | 完整備份需要 `zip` 擴充功能；HTML 區塊與 SVG 清理需要 `DOMDocument` |
| 瀏覽器 | 使用最新版 Chrome、Firefox、Safari 或 Edge 以支援區塊編輯器功能 |

## 安裝與更新

1. 從 [Releases](https://github.com/hoquangthaiholy/refine-lab-wp-releases/releases) 頁面下載 `refine-lab-wp-<version>.zip`。
2. **外掛 → 安裝外掛 → 上傳外掛**，選擇 ZIP 並啟用。
3. 更新會顯示在**外掛**頁面與 **Refine Lab → 總覽 → 更新**，也可以自動安裝（見 [site-tools.md](site-tools.md#更新refine-lab--總覽--更新)）。

刪除外掛時會保留你的資料；若想徹底移除，請先定義 `REFINELAB_WP_REMOVE_DATA`。

## 後台選單地圖

| 選單 | 頁面（分頁） | 用途 |
| :--- | :--- | :--- |
| **總覽** | — | 各區域狀態、健康檢查、更新、捷徑。 |
| **區塊** | 區塊、開發者參考 | HTML 區塊引擎：偵測到的區塊、範例區塊、入門佈景主題、快取。 |
| **自訂類型** | — | 不寫程式碼即可註冊文章類型與分類法。 |
| **內容區塊** | — | 佈景主題啟用後才會出現：可編輯的內容區域。 |
| **表單** | —（項目 Entries 為獨立清單） | 表單設定、統計；提交內容在**項目（Entries）**。 |
| **語言** | 設定、翻譯 | 多語言設定與翻譯儀表板。 |
| **SEO** | 搜尋與社群、轉址、404 記錄 | 中繼標籤、社群卡片、結構化資料、網站地圖、轉址。 |
| **效能** | 速度、媒體 | 前台最佳化、WebP／AVIF。 |
| **安全性** | 強化、檔案完整性、活動記錄 | 標頭、登入防護、雙重驗證、掃描、稽核軌跡。 |
| **郵件** | — | 以 SMTP 或 OAuth 2.0 寄送 WordPress 郵件。 |
| **編輯器** | — | 區塊編輯器的標題固定與可調整寬度的側邊欄。 |
| **工具** | 備份與還原、維護、指令碼、系統狀態 | 備份、維護模式、自訂程式碼、診斷。 |

## 五分鐘導覽

### 1. 建立你的第一個區塊

在佈景主題中建立 `blocks/feature-card.html`：

```html
<div class="feature-card" data-title="Feature Card" data-icon="star-filled">
  <h3 data-bind="title">Lightning fast</h3>
  <p data-bind="text">One sentence about the feature.</p>
</div>
```

開啟區塊編輯器，在插入器搜尋「Feature Card」，直接在畫布上編輯文字。不需要任何建置步驟。更多模式見 [html-blocks-cookbook.md](html-blocks-cookbook.md)；所有屬性見 [html-blocks.md](html-blocks.md)。還沒有佈景主題？**區塊 → 建立入門佈景主題／安裝範例區塊**會給你一個可用的基礎。

### 2. 加入表單

```html
<form data-form="contact" data-redirect="/thank-you/">
  <input data-form-field="name" type="text" required />
  <input data-form-field="email" type="email" required />
  <textarea data-form-field="message"></textarea>
  <button type="submit">Send</button>
</form>
```

提交內容會出現在**項目（Entries）**並寄到你的信箱。詳情見 [forms.md](forms.md)。

### 3. 讓網站支援多語言

**語言 → 設定**：新增語言、選擇網址樣式，然後在編輯器的*語言*方塊翻譯內容，並放上 `[refinelab_language_switcher]`。見 [multilang.md](multilang.md)。

### 4. 保護與備份

在**安全性**開啟你需要的防護（[security.md](security.md)），再到**工具 → 備份與還原 → 建立備份**。完整備份可以在其他網域還原：[backup-restore.md](backup-restore.md)。

### 5. 檢查健康狀態

**總覽 → 健康檢查**會指出缺少的備份、SMTP、薄弱的安全選項、待安裝的更新，以及忘記關閉的維護模式。

## 慣例

| | |
| :--- | :--- |
| 類別 | `RefineLab_*` |
| 選項、掛鉤、AJAX 動作 | `refinelab_*` |
| CSS | `.refinelab-*`（外掛絕不針對佈景主題的選擇器） |
| 常數 | `REFINELAB_WP_*` |
| 文字網域 | `refine-lab-wp`（原文為英文；內含繁體中文翻譯） |

## 取得協助

回報錯誤與提問：到[發佈儲存庫](https://github.com/hoquangthaiholy/refine-lab-wp-releases/issues)開一個 issue，並附上 **工具 → 系統狀態 → 複製支援報告** 的內容。
