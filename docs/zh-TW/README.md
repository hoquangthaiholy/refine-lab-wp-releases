# Refine Lab 文件

給網站建置者、佈景主題作者與外掛開發者的參考文件。這裡的內容皆已對照所屬版本的程式碼逐項確認。

**語言：** 繁體中文 · [English](../README.md)

## 從這裡開始

| 文件 | 適用對象 | 內容 |
| :--- | :--- | :--- |
| [getting-started.md](getting-started.md) | 所有人 | 系統需求、安裝與更新、後台選單地圖、五分鐘導覽。 |

## 以 HTML 區塊建置

| 文件 | 內容 |
| :--- | :--- |
| [html-blocks.md](html-blocks.md) | 完整語法：根元素屬性、`data-bind`、重複器（含巢狀）、樹狀結構、佔位符、條件與迴圈、表單／麵包屑／語言元件、側邊欄控制項與選擇器、全域情境物件、多檔案區塊、JavaScript 生命週期 API。 |
| [html-blocks-cookbook.md](html-blocks-cookbook.md) | 可直接複製修改的範本：主視覺、團隊、含巢狀重複器的價格表、導覽樹、站內文章、條件、表單、含 PHP 與 JS 的多檔案區塊；常見錯誤。 |

## 模組

| 文件 | 內容 |
| :--- | :--- |
| [forms.md](forms.md) | 宣告式表單：標記屬性、上傳、設定、JSON 回應、帶簽章的 webhook、頻率限制、項目（Entries）、CSV 匯出。 |
| [multilang.md](multilang.md) | 語言、網址模式、翻譯文章與詞彙、語言切換器（短代碼、區塊、PHP）、固定字串、翻譯儀表板。 |
| [security.md](security.md) | 所有強化與登入選項、自訂登入網址、IP 允許清單、雙重驗證、復原。 |
| [backup-restore.md](backup-restore.md) | 設定快照、完整備份、在其他網域還原並取代位址、設定檔。 |
| [site-tools.md](site-tools.md) | 效能、媒體（WebP／AVIF、SVG）、轉址與 404 記錄、指令碼、維護模式、檔案完整性、活動記錄、區塊歷史、編輯器選項、更新、系統狀態。 |
| [modules.md](modules.md) | 各模組內部：設定鍵、文章中繼資料、SEO、郵件、自訂類型、內容區塊。 |

## 給外掛開發者

| 文件 | 內容 |
| :--- | :--- |
| [hooks-and-api.md](hooks-and-api.md) | 所有篩選器與動作（含範例）、PHP 函式、短代碼、佈景主題支援、JavaScript API 與事件、REST／AJAX／admin-post 端點、常數、儲存的資料、CSS 掛鉤。 |
| [developer-reference.md](developer-reference.md) | 常數、選項、新增模組。 |

慣例：類別 `RefineLab_*`、選項／掛鉤／動作 `refinelab_*`、CSS `.refinelab-*`、常數 `REFINELAB_WP_*`；文字網域 `refine-lab-wp`。程式碼、屬性名稱與掛鉤名稱一律維持英文原樣。
