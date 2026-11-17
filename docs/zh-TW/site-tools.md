# 網站工具

[English](../site-tools.md)

營運類模組的參考。下列每個選項都顯示在其後台頁面；鍵名就是你在儲存的選項、匯出與 WP-CLI 中看到的名稱。安全性強化另有獨立指南：[security.md](security.md)。備份：[backup-restore.md](backup-restore.md)。表單：[forms.md](forms.md)。語言：[multilang.md](multilang.md)。

## 效能（Refine Lab → 效能 → 速度）

小巧、可復原的前台最佳化。一切**預設為關閉**，且在後台、登入畫面、feed、customizer 預覽，或開啟 `SCRIPT_DEBUG` 時完全不會執行。選項 `refinelab_performance_settings`。

| 鍵 | 效果 | 範例 |
| :--- | :--- | :--- |
| `defer_scripts` + `defer_handles` | 為列出的指令碼 handle 加上 `defer`（也會套用到本外掛自己的前台指令碼）。`defer_handles` 接受以空白、逗號或換行分隔的 WordPress 指令碼 handle（最多 30 個）。 | `jquery-migrate`、`my-theme-slider` |
| `decoding_async` | 為內容圖片加上 `decoding="async"`。 | — |
| `remove_emoji` | 移除表情符號偵測指令碼與樣式。 | — |
| `remove_embeds` | 移除 `wp-embed` 指令碼與 oEmbed 探索。 | — |
| `preload_fonts` | 每行一個網址；每個都會在 head 開頭成為 `<link rel="preload" href="…" as="font" type="font/woff2" crossorigin>`（type 依 `woff2`、`woff`、`ttf`、`otf` 選擇）。請使用你自己網站的檔案。 | `https://example.com/fonts/inter.woff2` |

指令碼 handle 可在頁面原始碼找到（`id="…-js"`，去掉 `-js`）。關閉選項即可回到原本的行為。

### 伺服器負載（Refine Lab → 效能 → 速度）

與上述選項不同，這些在後台也會生效。預設維持 WordPress 原有行為。

| 金鑰 | 值 | 效果 |
| :--- | :--- | :--- |
| `heartbeat_admin`、`heartbeat_editor`、`heartbeat_front` | `default`、`slow`（60 秒）、`off` | 各處的 Heartbeat 頻率。編輯器設為關閉時，也會停止文章鎖定與登入逾時提示。 |
| `revisions_limit` | `default`、`0`、`3`、`5`、`10`、`25`、`50` | `wp_revisions_to_keep`。**刪除舊修訂版本**會刪除超過上限的最舊版本（每次最多 2000 筆；記錄為 `revisions_cleaned`）。 |
| `autosave_interval` | `default`、`120`、`300`、`600` 秒 | 區塊編輯器自動儲存間隔。 |

### 資源提示與資料庫工具（Refine Lab → 效能）

| 金鑰／工具 | 效果 |
| :--- | :--- |
| `preconnect_hosts` | 每行一個位址（最多 6 個）；透過 `wp_resource_hints` 輸出為 `<link rel="preconnect">`。 |
| `fonts_swap` | 為缺少 `display=` 的 Google Fonts 樣式表加上 `display=swap`。 |
| 資料庫清理 | 統計並移除過期暫存、已刪除文章的中繼資料、垃圾與回收桶留言、超過 7 天的自動草稿（每次每類最多 1,000 筆；記錄為 `db_cleaned`）。請先備份。 |
| 自動載入的選項 | 總大小與最大的 20 個自動載入選項。**停止自動載入**會將該選項設為不自動載入（`wp_set_option_autoload`，記錄為 `autoload_off`）；核心選項受保護。 |

| 金鑰 | 效果 |
| :--- | :--- |
| `speculation` | `off`、`conservative`（按下時預取）或 `moderate`（滑過時預取）：輸出 Speculation Rules，讓 Chromium 瀏覽器預取下一頁。不適用於已登入的使用者；後台、登入、上傳與含 nonce 的網址以及 `rel=nofollow` 連結會被排除。會取代核心的規則。 |
| `dequeue_handles` | 對未登入訪客移除的指令碼／樣式控制代碼（最多 30 個）。新增後請檢查頁面。 |
| `strip_core_ver` | 當資源網址的 `?ver=` 等於 WordPress 版本時將其移除。 |

此頁面也提供壓縮與長效瀏覽器快取的 nginx 與 Apache 範本（伺服器規則無法由 PHP 設定）。

### 網站健康

Refine Lab 在 **工具 → 網站健康** 新增四項檢測（近期備份、郵件寄送、檔案完整性、uploads 防護），並在「偵錯資訊」加入「Refine Lab」區段（版本、啟用的模組、保留的備份數）。這些檢測只讀取已儲存的結果。

## 媒體（Refine Lab → 效能 → 媒體）

**媒體**分頁有四張卡片，每張各自儲存，並有各自的*儲存變更*按鈕：**媒體資料夾**（見下文）、**圖片最佳化**、**既有圖片**（批次轉換）與 **SVG 上傳**。圖片工具，選項 `refinelab_media_settings`。

| 鍵 | 預設 | 意義 |
| :--- | :--- | :--- |
| `webp_enabled`、`avif_enabled` | 關 | 為 JPEG 與 PNG 上傳檔建立 WebP／AVIF 副本（只有伺服器支援該格式時才提供）。 |
| `quality` | 82 | 副本的壓縮品質。 |
| `max_dimension` | 0（關） | 寬或高的上限；更大的上傳檔會被縮小。 |

副本與每個檔案及尺寸放在一起（`photo-300x200.jpg` → `photo-300x200.jpg.webp`），且**只有比原檔小時才保留**。附件中繼資料 `_refinelab_modern_images` 記錄哪些檔案有副本；`wp_content_img_tag` 上的篩選器會透過 `<picture>` 提供它們，不支援的瀏覽器仍使用原本的 `<img>`。批次工具可轉換現有媒體庫（`wp_ajax_refinelab_modern_batch`）。

### 媒體資料夾（Refine Lab → 效能 → 媒體）

以巢狀資料夾整理原生媒體庫。資料夾是**虛擬的**：它們是分類法 `refinelab_media_folder` 的詞彙，檔案仍留在磁碟上原來的位置，所以圖片網址永遠不會改變，備份與 CDN 照常運作，刪除資料夾也絕不會刪除檔案。用*啟用媒體資料夾*開關（`refinelab_media_settings` 中的 `folders_enabled`，預設開）啟用或停用；停用只會隱藏介面，你的資料夾與歸類都會保留。

| 位置 | 你會得到什麼 |
| :--- | :--- |
| **媒體 → 媒體庫**（網格與清單） | 左側的資料夾樹：*所有檔案*、*未歸類*與你的資料夾（含數量）。點選資料夾即可篩選。把滑鼠移到一列上會出現按鈕，可建立、重新命名與刪除資料夾（子資料夾最多 5 層）。**把檔案拖曳到資料夾**即可移動；選取多個檔案時會一起移動。清單模式另有**資料夾**欄，以及批次動作*移動到資料夾：…*與*移出資料夾*。 |
| **編輯器中的媒體挑選視窗**（選取圖片、特色圖片、任何媒體對話框） | 在其他篩選器旁多一個**資料夾篩選器**，資料夾依層級縮排。 |
| **上傳** | 新檔案會進入樹狀結構中（或挑選視窗篩選器）所選的資料夾；選取*所有檔案*時則維持未歸類。繞過媒體對話框的上傳（例如直接把檔案拖曳到圖片區塊）會是未歸類。 |

數量以每個資料夾自己計算（子資料夾中的檔案算在它們自己的資料夾）。刪除資料夾會把其子資料夾上移一層，並讓其檔案變成未歸類。一個檔案只能在一個資料夾。能上傳檔案的人都能使用資料夾；移動檔案還需要有編輯該檔案的權限（作者只能移動自己的檔案）。資料夾儲存在資料庫，因此**完整備份**會帶著它們；設定匯出只包含開關。開發者：REST 路由與資料見 [hooks-and-api.md](hooks-and-api.md)。

**SVG 上傳**（**編輯器／媒體**，旗標 `enable_svg_upload`）：經過嚴格的清理器後，允許媒體庫使用 `.svg`：檔案以 `DOMDocument` 解析（無 DTD、無網路），只保留允許清單中的元素與屬性，指令碼、事件處理程式、外部參照與不安全的 CSS 會被移除，儲存的檔案是清理後的標記。`.svgz` 不被接受。

## 轉址與 404 記錄（Refine Lab → SEO → 轉址、404 記錄）

規則會在 `template_redirect` 早期比對。資料表 `{prefix}refinelab_redirects` 與 `{prefix}refinelab_404`（自動建立；最多 1000 條規則）。

| 欄位 | 值 |
| :--- | :--- |
| `source` | 請求路徑，例如 `/old-page/`。 |
| `target` | 目的網址或路徑。410 不使用。 |
| `type` | `301` 永久、`302` 暫時、`410` 已移除。 |
| `match` | `exact`；`wildcard`，其中 `*` 擷取其餘部分（`/old/*` → `/new/$1`）；`regex`（PCRE 模式，反向參照 `$1`…）。 |

儲存規則時會拒絕迴圈與轉址鏈。**404 記錄：** 前台的 404 請求會被計數（略過機器人與探測路徑，每次請求只有一次輕量寫入，且有上限），每一列都有一鍵*建立轉址*。

**CSV 格式**（匯入最多 500 列／500 KB，匯出全部規則）：標題列 `source,target,type,match`。

```csv
source,target,type,match
/old-shop/,/shop/,301,exact
/blog/2019/*,/archive/$1,301,wildcard
/promo,,410,exact
```

無效的列會被略過；匯出時以 `=`、`+`、`-` 或 `@` 開頭的儲存格會被中和。

## 指令碼（Refine Lab → 工具 → 指令碼）

在五個位置輸出你自己的程式碼（分析、像素、行內 CSS 或 JS）。選項 `refinelab_scripts_settings`；每個位置最多 100 KB。

| 位置鍵 | 輸出於 |
| :--- | :--- |
| `front_head` | 前台，`</head>` 之前（`wp_head`）。 |
| `front_body` | 前台，緊接 `<body>` 之後（`wp_body_open`；佈景主題必須呼叫它）。 |
| `front_footer` | 前台，`</body>` 之前（`wp_footer`）。 |
| `admin_head`、`admin_footer` | 後台頁首／頁尾。 |

`skip_editors` 會讓有權編輯文章的已登入使用者**不**輸出前台程式碼（避免自己的造訪被納入分析）。程式碼會**原樣**輸出，所以請自行包含 `<script>` 或 `<style>` 標籤，且只有具備 `unfiltered_html` 的使用者才能變更。此選項刻意排除在備份與匯出之外。若某段程式碼讓頁面壞掉，請在 `wp-config.php` 加入 `define( 'REFINELAB_WP_DISABLE_SCRIPTS', true );`，再到後台修正。

```html
<!-- front_head -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXX"></script>
<script>window.dataLayer=window.dataLayer||[];function gtag(){dataLayer.push(arguments)}gtag('js',new Date());gtag('config','G-XXXX');</script>
```

## 維護模式（Refine Lab → 工具 → 維護）

選項 `refinelab_maintenance_settings`。

| 鍵 | 意義 |
| :--- | :--- |
| `enabled` | 開啟此模式。開啟期間工具列會顯示徽章。 |
| `page_id` | 要顯示的已發佈頁面（會套用你佈景主題的區塊與樣式）；留空則使用 `title` 與 `message`。 |
| `title`、`message` | 內建訊息。 |
| `retry_after` | 分鐘（5–1440，預設 60），送在 `Retry-After` 標頭中。 |
| `token` | 預覽連結 `https://example.com/?refinelab_preview=<token>` 的密鑰；造訪它會設定 12 小時的 cookie（`refinelab_maint_bypass`）以略過此模式。*產生新連結*會使舊連結失效。 |

訪客會得到 **HTTP 503**，附帶 `Retry-After` 與 `noindex`，因此搜尋引擎會保留你的排名。不受影響：有權編輯頁面的已登入使用者、登入畫面、後台、AJAX、WP-Cron 與 REST API。

## 檔案完整性（Refine Lab → 安全性 → 檔案完整性）

回報被修改、遺失與新增的檔案；它**絕不**編輯或刪除任何東西。每個範圍最多 4000 個檔案。

| 範圍 | 比對對象 |
| :--- | :--- |
| WordPress 核心 | api.wordpress.org 的官方校驗碼（網站健康狀態使用的同一份）。 |
| 本外掛 | 外掛內附的 `hashes.json`（在建置版本時產生），否則使用在本站取得的基準。 |
| 使用中的佈景主題 | 你在本站取得的基準（對檔案滿意時按*設定基準*）。 |

設定（`refinelab_integrity_settings`）：`weekly`（透過 cron 掛鉤 `refinelab_integrity_scan` 排程掃描）與 `email`（有變動時寄信給管理員）。最近一次的報告保存在 `refinelab_integrity_report`，基準保存在 `refinelab_integrity_baseline`。報告可下載為 CSV。

## 活動記錄（Refine Lab → 安全性 → 活動記錄）

誰做了什麼，存放在資料表 `{prefix}refinelab_activity`。設定（`refinelab_activity_settings`）：`enabled`（預設開）、`retention_days`（預設 90，由 cron `refinelab_activity_prune` 每日清理）、`store_ip`（預設關）。設定的值永遠不會被記錄，只記錄哪個選項變更了。

事件：登入、登入失敗、雙重驗證鎖定、外掛啟用／停用、切換佈景主題、設定變更、內容發佈、備份建立／還原、完整備份建立／下載／上傳／還原、設定匯出／匯入、完整性掃描、完整性基準設定、維護模式變更、翻譯建立。可依事件篩選、匯出 CSV，並在確認後清除。

## 診斷（Refine Lab → 工具 → 診斷）

- **排程事件（cron）：**列出每個事件與下次執行時間；*立即執行*會執行一次（仍保留排程），*刪除*會移除非核心事件（核心 `wp_*` 事件受保護）。延遲超過 10 分鐘的事件會被標示，並提示改用真正的伺服器 cron。記錄為 `cron_run`／`cron_deleted`。
- **偵錯記錄：**`debug.log` 的最後 200 行（可依文字篩選、下載、清空）。僅在 `WP_DEBUG_LOG` 啟用且檔案位於 wp-content 或 WordPress 資料夾內時可用。
- **郵件記錄**（預設關閉，選項 `refinelab_maillog_settings`）：在 `refinelab_maillog` 記住最近 100 封郵件（時間、收件者、主旨、結果、錯誤）。絕不儲存郵件內文，因此沒有重新寄送功能。

## 搜尋與取代（Refine Lab → 工具 → 搜尋與取代）

在本網站每個具單一欄位主鍵的資料表中取代文字（略過 `guid` 欄位），包含序列化資料內部（會修正長度，不會反序列化）。勾選*網站位址*時，也會比對 http/https、無通訊協定與 JSON 跳脫的形式，並在主機名稱結尾處停止。請務必先執行**試跑**（不勾選*套用*）：它只統計各資料表符合的列數而不寫入。每次執行約 20 秒後會停止並提供**繼續**。套用前請先備份；記錄為 `db_replaced`。

## 管理列

管理員會看到 **Refine Lab** 選單：清除區塊快取、開啟或關閉維護模式（開啟時顯示圓點）、前往備份與還原和診斷。

## WP-CLI## WP-CLI

| 指令 | 功能 |
| :--- | :--- |
| `wp refinelab status` | 版本、已啟用的模組、備份、排程、完整性。 |
| `wp refinelab backup-create [--components=database,uploads,plugins,themes] [--note=…]` | 在單一程序中進行整站備份。 |
| `wp refinelab backup-list [--format=json]`、`backup-delete <id>` | 列出／刪除備份。 |
| `wp refinelab integrity-scan` | 掃描核心、外掛與佈景主題檔案。 |
| `wp refinelab cache-clear` | 清除 HTML 區塊快取。 |
| `wp refinelab db-clean [--kinds=transients,spam] [--dry-run]` | 資料庫清理（重複執行直到清空）。 |
| `wp refinelab autoload [--limit=20]` | 最大的自動載入選項。 |
| `wp refinelab blocks-lint` | 範本警告與各 HTML 區塊的使用位置（有警告時結束碼為 1）。 |

## 區塊歷史（編輯器）

佈景主題區塊的編輯器捷徑：**可見性**面板可在手機（600px 以下）、平板（600–959px）或桌機（960px 以上）隱藏區塊，使用 class `refinelab-hide-mobile|tablet|desktop`（屬性 `refinelabHide`）；區塊選項選單有**複製區塊設定**／**貼上區塊設定**（限同一區塊類型）；**顯示區塊外框**（更多選單或指令面板）會標出區塊與綁定欄位。指令面板（Ctrl/Cmd+K）也可清除區塊快取、前往備份與診斷。

文件側邊欄另有 **無障礙檢查** 面板：掃描畫布中缺少替代文字的圖片、沒有名稱的連結、跳級或重複的標題層級與空標題；點選項目即可選取該區塊（手動重新檢查，不會在背景掃描）。

每次儲存文章時，每個 `theme/*` 區塊的屬性都會快照到文章中繼資料 `_refinelab_block_history`（每篇文章最多 20 份快照／400 KB）。在區塊側邊欄，**區塊歷史**會列出所選區塊先前的狀態；*還原*會將區塊回復，且該變更可像一般編輯一樣復原。REST 路由見 [hooks-and-api.md](hooks-and-api.md)。

## 編輯器增強（Refine Lab → 編輯器）

| `refinelab_tools_settings` 中的旗標 | 預設 | 效果 |
| :--- | :--- | :--- |
| `enable_sticky_post_title` | 開 | 捲動編輯器時，讓文章標題固定（模糊背景）。 |
| `enable_resizable_sidebar` | 開 | 拖曳編輯器側邊欄的左緣調整寬度；按兩下重設為 280 px；寬度存於 `localStorage['refinelab_sidebar_width']`。需要 HTML 區塊模組。 |

## 更新（Refine Lab → 總覽 → 更新）

版本發佈在公開儲存庫 `hoquangthaiholy/refine-lab-wp-releases`。外掛標頭將該儲存庫宣告為 `Update URI`，所以 WordPress 本身會在**外掛**頁面顯示更新通知、提供一鍵更新與每個外掛的*啟用自動更新*開關。總覽卡片另外提供*檢查更新*與*自動安裝新版本*（WordPress 本身的自動更新清單）。

- 外掛從儲存庫 `main` 分支讀取 `update.json`（快取 12 小時；失敗後 1 小時），並從 `sections.changelog` 顯示*新功能*。
- ZIP 必須是該儲存庫的資產，且其 **SHA-256** 必須與資訊清單相符；下載內容會在 WordPress 解壓縮之前驗證，否則什麼都不會安裝。
- 不會儲存或傳送任何權杖。只有在測試環境才需要覆寫位址：常數 `REFINELAB_WP_UPDATE_URL` 或篩選器 `refinelab_update_manifest_url`。

資訊清單欄位：`version`、`package`、`sha256`、`requires`、`requires_php`、`tested`、`released`、`homepage`、`sections.changelog`。

## 系統狀態（Refine Lab → 工具 → 系統狀態）

供支援使用的環境詳細資訊（WordPress、佈景主題、PHP、伺服器、限制、擴充功能、哪些 Refine Lab 模組已開啟、區塊與自訂類型的數量）。*複製支援報告*會把純文字版放到剪貼簿。回報錯誤時請附上。
