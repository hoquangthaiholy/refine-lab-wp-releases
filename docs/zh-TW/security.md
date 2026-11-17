# 安全性模組參考

類別 `RefineLab_Security` · 選項 `refinelab_security_settings` · 後台頁面 **Refine Lab → 安全性** · 檔案 `includes/modules/security/class-security.php`。

[English](../security.md)

## 後台頁面配置

卡片依序為：總開關 → **登入防護**（自訂登入網址、IP 允許清單、嘗試次數限制、雙重驗證、通用錯誤訊息）→ **存取強化** → **HTTP 標頭** → **隱藏 WordPress** → **伺服器層級隱藏**。總開關關閉時，其下的所有項目都會隱藏。相依欄位只在其上層開關開啟時才會顯示：登入位址／被封鎖請求的行為（自訂登入網址）、允許的 IP（IP 限制）、嘗試次數／鎖定分鐘數（嘗試次數限制）。隱藏的欄位會保留已儲存的值。此機制是通用的：將 `depends`（核取方塊 id）與 `depends_on`（目前狀態）傳給 `setting-row` 元件，`core-admin.js` 會在變更時切換 `hidden`。

每個選項都以 `1`／`0` 儲存（數字與字串另有註明）。總開關 `enabled` 會關閉整個模組。缺少的鍵會套用預設值，因此既有網站升級後行為不變；除非另有註明，新選項預設為關閉。

## 強化

| 鍵 | 預設 | 行為 |
| :--- | :--- | :--- |
| `disable_xmlrpc` | 開 | `xmlrpc_enabled` 設為 false、移除 `X-Pingback` 標頭，帶有 `XMLRPC_REQUEST` 的請求回應 403。 |
| `block_user_enumeration` | 開 | `/?author=N` 轉址到首頁、對訪客隱藏 REST `/wp/v2/users` 路由、從核心 XML 網站地圖移除 `users` 提供者。 |
| `security_headers` | 開 | `X-Frame-Options: SAMEORIGIN`、`X-Content-Type-Options: nosniff`、`Referrer-Policy: strict-origin-when-cross-origin`、`X-XSS-Protection`。 |
| `permissions_policy` | 關 | `Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()`。 |
| `hsts` | 關 | `Strict-Transport-Security: max-age=31536000; includeSubDomains`，僅在 HTTPS 下送出。 |
| `restrict_rest_api` | 關 | 除了篩選器 `refinelab_security_rest_allowed_namespaces` 允許的命名空間（預設 `refinelab`、`oembed`）外，訪客存取每一條 REST 路由都會得到 HTTP 401。可能會讓公開表單或無頭（headless）外掛失效。 |
| `disable_application_passwords` | 關 | `wp_is_application_passwords_available` 回傳 false。 |
| `disallow_file_edit` | 開 | 定義 `DISALLOW_FILE_EDIT`（僅在尚未定義時）。 |
| `block_uploads_php` | 關 | 在 `uploads/.htaccess` 寫入標記區塊，拒絕 `.php`、`.phtml`、`.phar` 等檔案；關閉時會移除。僅適用 Apache/LiteSpeed；nginx 請加入「伺服器層級隱藏」中的規則。**立即測試**會上傳一個用完即刪的探測檔、以 HTTP 請求並回報 PHP 是否會被執行（探測檔一律刪除）。 |
| `disallow_file_mods` | 關 | `file_mod_allowed` 為 false：後台無法安裝、更新或刪除外掛與佈景主題（同 `DISALLOW_FILE_MODS`）。關閉前包含本外掛在內的更新皆不會執行。 |

## 安全警示

透過 `wp_mail()`（因此會套用郵件模組設定）寄信到 `alert_email`（預設為網站管理員信箱）。同一類警示每 5 分鐘最多一封。

| 金鑰 | 寄出時機 |
| :--- | :--- |
| `alert_admin_login` | 管理員從未見過的位址登入。位址僅以加鹽雜湊存於使用者中繼資料 `_refinelab_known_ips`（最近 10 筆）；啟用後的第一次登入只會記錄。 |
| `alert_lockout` | 登入限制器鎖定某個位址時（需啟用 `limit_login_attempts`）。 |
| `alert_admin_change` | 新增管理員，或使用者被授予管理員角色。 |
| `alert_plugin_change` | 外掛被啟用、停用、刪除，或切換佈景主題。 |

## 密碼與工作階段

| 金鑰 | 值 | 行為 |
| :--- | :--- | :--- |
| `password_min_length` | `0`（WordPress 預設）、8、10、12、14、16 | 新密碼的最小長度。 |
| `password_require_mix` | 開／關 | 小寫、大寫、數字、符號至少三種。 |
| `password_hibp` | 開／關 | 拒絕出現在 Pwned Passwords 清單中的密碼。僅傳送 SHA-1 雜湊的前 5 個字元（k-匿名），絕不傳送密碼；若無法連線服務則接受該密碼。 |
| `idle_logout_minutes` | `0`（從不）、15、30、60、120、240、480 | 可編輯內容的使用者在此時間內沒有頁面請求就登出（Heartbeat 不算）。記錄於使用者中繼資料 `_refinelab_last_seen`，每分鐘最多寫入一次。 |
| `max_sessions` | `0`（不限）、1、2、3、5 | 登入時，超過上限的最舊工作階段會被結束。 |

密碼規則適用於在個人資料中設定密碼、新增使用者畫面與密碼重設；既有密碼不受影響。同一張卡片上有 **登出我所有的其他工作階段**。

## 內容安全政策（CSP）

`csp_mode`：`off`、`report_only`（標頭 `Content-Security-Policy-Report-Only`，不會阻擋任何內容）或 `enforce`。`csp_preset`：`relaxed`（允許行內指令碼與 HTTPS 來源）、`strict`（僅限同站、無行內指令碼）或 `custom`（`csp_custom`，以分號分隔的純指令）。僅在前台請求送出。瀏覽器會將違規回報到 `POST /wp-json/refinelab/v1/csp-report`（本質上是公開的：每小時最多 120 筆、移除查詢字串、`refinelab_csp_reports` 中最多保留 50 組不同的指令／位址）；清單顯示在「安全性」頁面。請先用僅回報模式，調整政策直到清單不再增加，再啟用強制執行。

## 登入防護## 登入防護

| 鍵 | 預設 | 行為 |
| :--- | :--- | :--- |
| `obscure_login_errors` | 開 | 將登入錯誤訊息取代為「Invalid login credentials.」。 |
| `two_factor_enabled` | 關 | 雙重驗證（TOTP），見下文。 |
| `ip_restrict_enabled`、`ip_allowlist` | 關、`''` | 後台 IP 允許清單，見下文。 |
| `limit_login_attempts` | 關 | 以暫存資料 `refinelab_ll_<md5(ip)>` 依用戶端 IP 計算失敗次數；超過 `login_max_attempts`（3–20，預設 5）後，該用戶端會被拒絕 `login_lockout_minutes`（1–1440，預設 15）分鐘。登入成功會清除計數。IP 取自 `REMOTE_ADDR`；可用篩選器 `refinelab_security_client_ip` 讀取受信任的代理標頭。 |
| `custom_login_enabled`、`login_slug`、`login_blocked_action` | 關、`''`、`404` | 見下文。 |

### 以 IP 限制後台存取

`ip_allowlist` 存放 IPv4／IPv6 位址與 CIDR 範圍（每行一筆；無效的行會被丟棄並顯示警告）。

- 清單以外的用戶端在登入表單（`/wp-login.php` 或自訂 slug）與 `/wp-admin/*` 會得到 **403**，且無法在任何地方以密碼驗證（登入表單、XML-RPC、REST 基本驗證）。
- 不受限制：公開頁面、`admin-ajax.php`、`admin-post.php`、WP-Cron／WP-CLI、`wp-login.php?action=postpass`。
- 已登入的 cookie 也會被擋在 `/wp-admin` 之外，因為檢查是逐次請求進行的。
- 安全機制：清單為空或不包含你目前的 IP 時，會拒絕儲存（功能維持關閉）。用戶端 IP 為 `REMOTE_ADDR`（位於代理／CDN 之後請使用篩選器 `refinelab_security_client_ip`，否則所有訪客看起來可能都是代理的 IP）。
- **復原**：在 `wp-config.php` 將 `REFINELAB_DISABLE_IP_RESTRICTION` 定義為 `true`。

### 雙重驗證（TOTP）

類別 `RefineLab_Two_Factor`（`includes/modules/security/class-two-factor.php`），在 `two_factor_enabled` 開啟時載入。RFC 6238：SHA-1、6 位數、30 秒步長、允許 ±1 步的時鐘誤差。

- 每位使用者在 **使用者 → 個人資料 → 雙重驗證** 自行啟用：將顯示的金鑰輸入驗證器應用程式（手動輸入，或在手機上使用 `otpauth://` 連結），以驗證碼確認，並保存 10 組一次性**復原碼**（只顯示一次）。沒有 QR code，因為產生 QR code 需要函式庫或會看到密鑰的外部服務。
- 未啟用的使用者照常登入。登入表單會多一個「Authentication code」欄位；驗證碼在密碼之後檢查。用過的驗證碼無法重複使用（會儲存最後接受的時間步）。
- 停用或重新產生復原碼需要有效的目前驗證碼或復原碼。具有 `edit_users` 權限的管理員可以在該使用者的個人資料頁重設其雙重驗證（核取方塊「Turn off two-factor…」）。
- 驗證碼錯誤會觸發 `wp_login_failed`，因此登入限制器會計入。此外 `verify_login()` 會針對每位使用者節流猜測：無效驗證碼達 `MAX_FAILS`（5）次後，該使用者會被鎖定 `LOCK_SECONDS`（15 分鐘），透過暫存資料 `refinelab_2fa_fail_<id>` 實作；鎖定事件會以 `twofactor_locked` 記錄在活動記錄中。驗證碼的時間步不能重複使用（`_refinelab_2fa_last_step`）。
- 儲存：使用者中繼資料 `_refinelab_2fa_{enabled,secret,pending,recovery,last_step}`。密鑰以 libsodium 加密，金鑰由 `AUTH` salt 衍生；**更換 salt 會使已登錄的密鑰失效**（使用者必須使用復原碼或由管理員重設）。復原碼以 HMAC 雜湊儲存。
- 驗證碼提示是登入表單上的一個欄位，而不是第二個畫面，因此錯誤或缺少驗證碼會透露密碼是正確的。應用程式密碼與 REST cookie 驗證不經過登入表單，不受影響（如有需要請另外停用應用程式密碼）。
- **復原**：在 `wp-config.php` 將 `REFINELAB_DISABLE_2FA` 定義為 `true`，即可略過所有雙重驗證檢查並隱藏相關介面。

### 依角色要求兩步驟驗證

`twofactor_required_roles`（角色）、`twofactor_grace_days`（0、3、7、14、30）與 `twofactor_required_since`（首次選擇角色時設定）。具有這些角色但尚未設定兩步驟驗證的使用者，在寬限期結束前只會看到提醒；之後除了個人資料、`admin-post.php` 與 `admin-ajax.php` 外，所有後台畫面都會導向個人資料，直到設定完成。需要啟用 `two_factor_enabled`。若所有人都被鎖在外面，可在 wp-config.php 定義 `REFINELAB_DISABLE_2FA` 以關閉強制要求。不提供電子郵件一次性驗證碼。

### 自訂登入網址

啟用後，登入表單會在 `https://site/<slug>/` 提供：

- 請求會在 `wp_loaded`（優先順序 0）被攔截：相對於首頁網址的路徑等於 slug 時，載入 `wp-login.php`。
- 所有產生的登入網址（`site_url`、`network_site_url`、`wp_redirect`：登入、登出、忘記密碼、重設郵件、`wp_login_url()`）都會從 `wp-login.php` 改寫為 slug。
- 未登入者對 `wp-login.php` 與 `/wp-admin/*`（`admin-ajax.php` 與 `admin-post.php` 除外）的請求會得到 **404 頁面**或**轉址到首頁**（`login_blocked_action`：`404` | `home`）。`wp-login.php?action=postpass` 仍可存取，以支援快取的密碼表單頁面。
- 儲存時的驗證：至少 3 個字元；不能是保留字（`wp-admin`、`wp-login`、`wp-content`、`wp-includes`、`wp-json`、`wp-cron`、`admin`、`dashboard`、`login`、`feed`、`sitemap`）；不能是現有公開頁面／文章的 slug。失敗時功能維持關閉，並以錯誤通知說明原因。
- 不支援多站網路。WP-CLI 與 cron 不啟用。
- **復原**：在 `wp-config.php` 加入 `define( 'REFINELAB_DISABLE_CUSTOM_LOGIN', true );` 以恢復 `/wp-login.php`，或透過 SFTP 重新命名外掛資料夾。
- 儲存前請先將新位址加入書籤；在靜態標記中寫死 `/wp-login.php` 連結的外掛，會讓未登入的訪客看到 404。

## 隱藏 WordPress

這些選項移除常見的識別特徵。它們能減少自動化掃描器可偵測到的資訊，但無法完全隱藏 WordPress（`/wp-content/` 等路徑、區塊標記與 REST 路由依然存在）。

| 鍵 | 預設 | 移除／變更 |
| :--- | :--- | :--- |
| `hide_wp_version` | 開 | `<meta name="generator">`（含 feed）、指令碼／樣式的 `?ver=`。 |
| `clean_head` | 開 | RSD、Windows Live Writer、短網址與相鄰文章的 `<link>`。 |
| `hide_rest_api_links` | 關 | `api.w.org` link 標籤與 `Link` 標頭、REST RSD 項目、oEmbed 探索連結與主機 JS。REST API 本身仍保持啟用。 |
| `disable_emojis` | 關 | 表情符號偵測指令碼與樣式、TinyMCE 表情符號外掛、`s.w.org` DNS 預先擷取、feed／郵件中的表情符號。 |
| `hide_server_headers` | 關 | `X-Powered-By`、`X-Pingback`、`X-Redirect-By: WordPress`。要得到完整效果，請在 `php.ini` 設定 `expose_php = Off`。 |
| `hide_admin_branding` | 關 | 後台頁尾的版權與版本、工具列的 WordPress 標誌選單、儀表板「WordPress 活動與新聞」小工具；登入頁標誌的連結與文字改為指向本站。 |

### 伺服器層級的隱藏（無法透過 PHP 完成）

靜態檔案由網頁伺服器在 PHP 執行之前就回應了。安全性頁面會列出以下設定片段：

```nginx
location ~* ^/(readme\.html|license\.txt|wp-config-sample\.php|wp-links-opml\.php)$ { return 404; }
location ~* ^/wp-content/(uploads|upgrade)/.*\.php$ { return 403; }
server_tokens off;
```

```apache
<FilesMatch "^(readme\.html|license\.txt|wp-config-sample\.php|wp-links-opml\.php)$">
	Require all denied
</FilesMatch>
ServerTokens Prod
Header unset X-Powered-By
```

## 篩選器

| 篩選器 | 參數 | 用途 |
| :--- | :--- | :--- |
| `refinelab_security_rest_allowed_namespaces` | `string[]` | REST 限制開啟時，訪客可存取的命名空間。 |
| `refinelab_security_client_ip` | `string $ip` | 登入限制器使用的 IP。 |

## 程式介面

- `RefineLab_Security::get_settings()`／`get_defaults()`／`get_features()`（開關的中繼資料，含 `group`：`hardening`、`login`、`hide_wp`）。
- `RefineLab_Security::get_login_slug()` 回傳目前啟用的 slug，沒有則為 `''`。
- `RefineLab_Security::sanitize_input( $input )` 是兩個 Settings API 註冊共用的唯一清理函式。
