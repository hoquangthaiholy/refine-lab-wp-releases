# 備份與還原

[English](../backup-restore.md)

一個頁面 **Refine Lab → 工具 → 備份與還原**，滿足三種需求：快速的設定快照、可在其他網域還原的整站備份，以及可攜式設定檔。

| 你想要… | 使用 | 內容 | 儲存位置 |
| :--- | :--- | :--- | :--- |
| 復原一次設定變更 | **僅設定**備份（自動或手動） | Refine Lab 設定（不含密鑰） | 選項 `refinelab_settings_backups`，最近 10 份 |
| 複製或搬移整個網站 | **完整備份** | 資料庫、上傳檔案、外掛、佈景主題 | `uploads/refinelab-backups/` 內的 ZIP，最近 5 份 |
| 分享一份設定 | **匯出設定檔** | 選擇的設定，以及可選的區塊內容 | 你下載的 JSON 檔 |

## 1. 設定備份

在啟用或停用模組之前、外掛更新之後，以及每次還原或匯入之前，都會自動建立快照。**建立備份 → 僅設定**可隨時手動建立。密碼與密鑰（鍵名以 `password`、`pass`、`secret`、`token`、`api_key` 結尾者）一律不包含，還原也不會覆蓋網站上已儲存的密鑰。原始自訂程式碼（*指令碼*選項）同樣被排除。

從清單還原（**還原**）：選擇快照，可選擇勾選*變更還原設定中的網域*（來源 = 快照中儲存的位址，目標 = 本站），然後確認。系統會先為目前狀態建立新的快照，因此還原本身也可以復原。

## 2. 完整備份

**建立備份 → 完整備份**，選擇要包含的內容：

| 部分 | 內容 |
| :--- | :--- |
| 資料庫 | 所有使用本站前綴的資料表（文章、頁面、留言、使用者、選項、外掛資料表）。 |
| 上傳檔案 | 媒體庫資料夾。 |
| 外掛 | 所有外掛資料夾。 |
| 佈景主題 | 所有佈景主題資料夾。 |

`.git`、`.svn`、`.hg`、`.idea`、`node_modules` 與 `.DS_Store` 會被排除。可加入選填的備註。進度列會追蹤工作進度；工作以短步驟執行，因此大型網站不會碰到 PHP 時間限制。完成前請不要關閉分頁。

**備份記錄的資訊**（每一列的*備註*，同時以 `manifest.json` 存在 ZIP 內）：網站位址與網站網址、WordPress／PHP／MySQL 版本、資料表前綴、語言、使用中的佈景主題、Refine Lab 版本、設定群組數量、文章／頁面／媒體／留言／使用者數量、資料庫資料表／資料列／大小、各部分的檔案數與位元組、完整的外掛與佈景主題清單（含版本與啟用狀態），以及你的備註。

**封存檔結構**

```text
manifest.json            網站描述（schema_version 1）
db.sql                   每行一個陳述式：DROP TABLE、CREATE TABLE、INSERT
files/uploads/…          媒體庫
files/plugins/…          外掛
files/themes/…           佈景主題
```

**檔案的安全性：** 資料夾以拒絕規則保護（`.htaccess`、`web.config`、空的 `index.php`），檔名含有隨機部分，下載只能透過檢查權限與 nonce 的 admin-post 處理程式。完整備份含有使用者與密碼雜湊，請妥善保管。

需求：PHP `zip` 擴充功能與單一網站安裝（不支援多站網路）。從清單刪除備份；超過 5 份的舊備份會自動移除。

## 3. 還原完整備份

1. 在清單中的備份旁按**還原**，或在**從檔案還原**上傳其他網站建立的 `.zip`（受 PHP 上傳大小限制，頁面會顯示你的上限）。
2. 選擇**要還原的內容**：資料庫、上傳檔案、外掛、佈景主題（只有備份內含的部分才能勾選；若以 `DISALLOW_FILE_MODS` 阻擋檔案變更，或帳號缺少 `install_plugins`／`install_themes`，則外掛與佈景主題會停用）。
3. **網域。** 勾選*變更還原內容中的網域*，把舊位址改寫為新位址。來源顯示備份中的位址，目標預設為本站位址。
4. 保持勾選*先備份此網站*（建議），確認警告，按**立即還原**。

還原會取代你所選的內容：資料表會被刪除並重新建立，外掛／佈景主題／上傳檔案會被覆寫（備份中沒有的檔案不會動）。

### 網域取代

每個資料表的每個文字欄位都會被掃描並取代舊位址，包括 **PHP 序列化**的值（字串長度會重新計算，完全不做反序列化，因此不會建立任何物件）、含跳脫斜線的 JSON（`https:\/\/old.example.com`）、省略通訊協定的位址（`//old.example.com`），以及 `http` 與 `https` 兩種。比對會在邊界處停止，所以 `old.example.com` 不會匹配 `old.example.community`。文章的 `guid` 欄位刻意不動。`home` 與 `siteurl` 會設為新位址。

也可用同樣的工具變更路徑：從 `https://a.com` 搬到 `https://b.com/blog` 的網站，會連同子資料夾整體改寫。

### 還原還會處理什麼

- **資料表前綴：** 備份的前綴與本站不同時，會以本站前綴建立資料表，並重新命名與前綴相關的鍵（`options` 中的角色、`usermeta` 中的 `capabilities` 與其他鍵）。
- **外掛維持啟用：** 若還原後的啟用清單缺少 Refine Lab，會自動補回，且本外掛自己的資料夾絕不會被覆寫。
- **備份清單：** 保留*本站*的備份清單，不會被來源網站的清單取代。
- **暫存資料與重寫規則**會被清除。
- **安全性：** 只會執行針對備份前綴資料表的 `DROP TABLE`、`CREATE TABLE` 與 `INSERT INTO` 陳述式；ZIP 內的路徑會被正規化且不得離開其目標資料夾；指令碼檔案（`.php`、`.phtml`、`.phar` 等）與 `.ht*` 檔案不會寫入 uploads。

### 保持登入

資料庫被取代時，使用者與其登入工作階段也一起被取代，所以還原通常以登入畫面收場。還原透過**獨立執行器**（`restore-runner.php`）進行，它只載入 WordPress 核心的最小環境，因此在它所依賴的選項、外掛、使用者與工作階段被替換的過程中仍可持續運作。在**相同位址**還原之後，若備份中存在相同帳號（相同 ID 與登入名稱）且密碼未變，你的工作階段會被併回，因此保持登入。**變更網域**後，登入 cookie 名稱會改變，你需要以備份中的帳號重新登入。結果訊息會說明屬於哪一種情況。

若網頁伺服器封鎖了執行器檔案，頁面會自動改用 `admin-ajax.php`。

### 還原疑難排解

| 症狀 | 處理方式 |
| :--- | :--- |
| 「This job expired or is not valid」 | 工作 2 小時後過期；重新開始。 |
| 還原因資料庫錯誤而中止 | 訊息會指出失敗的陳述式；請檢查 MySQL 權限（`DROP`、`CREATE`）與 `max_allowed_packet`。修正後重新執行。 |
| 網站載入但沒有樣式／連結指向舊網域 | 沒有勾選網域選項。還原安全備份，或勾選網域選項後再還原一次。 |
| 還原後被鎖在外面 | 以備份中的帳號登入；用你的信箱重設密碼。 |

## 4. 設定檔（匯出與匯入）

**匯出設定檔**會以簡單的名稱與說明列出每個設定群組（一般設定、表單、郵件寄送（SMTP）、語言、SEO 與社群分享、效能、安全性強化…），以及可選的**區塊內容**項目，用來儲存文章與頁面中每個 HTML 區塊的屬性。下載 JSON 即可。

```json
{
  "schema_version": 1,
  "plugin_version": "1.1.0",
  "exported_at": "2026-11-17T07:00:00+00:00",
  "site": { "home": "https://old.example.com" },
  "options": { "refinelab_seo_settings": { "...": "..." } },
  "blocks": [ { "post_type": "page", "slug": "home", "title": "Home", "blocks": [ { "path": "0", "name": "theme/hero", "attrs": { } } ] } ]
}
```

**從檔案還原**以同一個上傳框接受該 JSON（以及完整備份的 `.zip`）。設定檔在**任何變更之前都會先讓你檢視**：每個群組都標示為*新增*、*將變更*或*相同*並據此預設勾選，區塊內容依文章類型與 slug 對應到文章（沒有對應的文章會被略過），你也可以勾選*變更匯入設定與區塊內容中的網域*（來源從檔案讀取）。系統會先建立設定快照。接受最大 2 MB 的檔案；由較新外掛版本建立的檔案會被拒絕。

## 5. 篩選器與端點

| 項目 | 用途 |
| :--- | :--- |
| `refinelab_backup_options`（filter，`string[]`） | 新增或移除屬於快照與匯出的選項名稱。名稱必須以 `refinelab_` 開頭。 |
| `admin_post_refinelab_backup_create|restore|delete|download` | 設定快照。 |
| `admin_post_refinelab_export`、`refinelab_import_upload`、`refinelab_import_apply` | 設定檔匯出／匯入。 |
| `wp_ajax_refinelab_site_backup_step`（+ `nopriv`）與 `restore-runner.php` | 分步驟的完整備份／還原工作，以每個工作專屬的密鑰授權。 |
| `admin_post_refinelab_site_backup_download|delete|upload` | 完整備份檔案。 |

活動記錄事件：*Backup created*、*Backup restored*、*Settings exported*、*Settings imported*、*Full backup created / downloaded / uploaded / restored*。

解除安裝時會保留備份，除非設定了 `REFINELAB_WP_REMOVE_DATA`（此時會刪除備份資料夾與選項）。
