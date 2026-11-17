# 模組參考

[English](../modules.md)

所有模組都繼承 `RefineLab_Abstract_Module`（`get_slug()`、`get_title()`、設定輔助函式、`render_admin_tab()` → `views/tab.php`）。它們由 `RefineLab_Plugin` 建立（`load_core_modules()`、延遲載入的 `get_module()`），並可透過 `refinelab_tools()-><slug>` 存取。模組開關存放在選項 `refinelab_tools_settings` 中的 `enable_<slug>`；模組設定存放在 `refinelab_<slug>_settings`。

後台頁面（Refine Lab 選單）：HTML 區塊、編輯器增強、媒體工具、郵件、表單、語言、SEO、安全性、自訂類型、系統狀態、資源庫。

---

## admin
`RefineLab_Admin_Settings`（`includes/modules/admin/`）、`RefineLab_Extended_Settings`、`RefineLab_Admin_UI`。
- 註冊選單（`refine-lab-wp`）、分頁路由，以及 Settings API 群組 `refinelab_tools_settings_group` 與 `refinelab_<module>_settings_group`。
- 清理函式：`refinelab_sanitize_{mailer,forms,multilang,seo,security}_settings()`（位於 `class-extended-settings.php`）；主選項會經過篩選器 `refinelab_tools_sanitize_settings`。
- UI 元件：`RefineLab_Admin_UI::render( $name, $args )`／`::get()` 渲染 `templates/admin/components/` 中的部件（`badge`、`button`、`card`、`choice-list`、`code-block`、`data-table`、`empty-state`、`file-upload`、`modal`、`notice`、`option-group`、`page-header`、`save-bar`、`select`、`setting-row`、`tab-panel`、`tabs`、`text-field`、`textarea`、`toggle`）。參數記載在每個部件檔案的開頭。
- 主選項 `refinelab_tools_settings` 的鍵：`enable_html_blocks`、`enable_svg_upload`、`enable_sticky_post_title`、`enable_resizable_sidebar`（預設皆開）。

## html-blocks
`RefineLab_HTML_Blocks` + traits `Parser`、`Renderer`、`Admin`。見 [html-blocks.md](html-blocks.md)。
- 在 `init` 註冊區塊，透過 `block_categories_all` 註冊分類，在 `enqueue_block_editor_assets` 載入編輯器資源，並在 `save_post` 與 `after_switch_theme` 清除快取。
- 後台動作：`refinelab_create_sample_theme`、`refinelab_install_sample_blocks`、`refinelab_clear_blocks_cache`。
- 值得知道的方法：`get_detected_blocks()`、`get_scan_directories()`、`get_global_block_attributes()`、`is_enabled()`、`is_sticky_title_enabled()`、`is_resizable_sidebar_enabled()`。
- 編輯器指令碼是隨外掛附上的產生套件；其原始碼在原始碼儲存庫中維護。

## multilang
見 [multilang.md](multilang.md) 的完整指南。摘要：`RefineLab_MultiLang` + traits `Routing`、`Query`、`Admin`、`Taxonomies`、`Strings`、`Switcher`。選項 `refinelab_multilang_settings`、`refinelab_multilang_strings`。文章中繼資料 `_refinelab_lang`、`_refinelab_translation_group`；重寫標籤 `%refinelab_lang%`；短代碼 `[refinelab_language_switcher]`。

## forms
`RefineLab_Forms` + traits `Ajax`、`Export`、`Advanced`。選項 `refinelab_forms_settings`。完整指南：[forms.md](forms.md)。
- 設定：`enabled`、`enable_honeypot`、`send_email`、`notification_email`（預設為管理員信箱）、`email_subject`、`success_msg`、`error_msg`、`recaptcha_site_key`、`recaptcha_secret`、`recaptcha_version`（`v2` | `v3`；v3 最低分數篩選器 `refinelab_forms_recaptcha_min_score`，預設 0.5），以及頻率限制、webhook 與附件鍵。
- 提交內容儲存在自訂文章類型 `refine_entry`（舊版 `refinelab_submission` 會遷移一次）；CSV 匯出透過 `admin_post_refinelab_export_submissions_csv`。
- 前台：`assets/js/syntax-forms.js`／`.css`，只在渲染表單時載入。Honeypot 欄位 `_refinelab_hp`；成功時的 DOM 事件 `refinelab:form:success`。
- AJAX `refinelab_form_submit`（公開 + 已登入）。
- 標記：`data-form`、`data-form-label`、`data-redirect`、`data-form-field`、`data-captcha`。含 webhook、附件與頻率限制的完整指南：[forms.md](forms.md)。

## mailer
`RefineLab_Mailer`（在 `phpmailer_init` 延遲載入）。選項 `refinelab_mailer_settings`：`enabled`、`smtp_host`、`smtp_port`、`smtp_secure`、`smtp_auth`、`smtp_user`、`smtp_pass`、`from_email`、`from_name`、`force_from`，加上 OAuth 2.0（Gmail／Microsoft 365）：`auth_method`、`oauth_client_id`、`oauth_client_secret`、`oauth_tenant`、`oauth_refresh_token`、`oauth_email`（透過 PHPMailer 的 XOAUTH2；程式碼在 `traits/trait-mailer-oauth.php`）。
- 掛鉤：`phpmailer_init`、`wp_mail_from`、`wp_mail_from_name`。AJAX `refinelab_send_test_email` 從後台頁面寄出測試郵件。
- **郵件送達率（DNS 記錄）。** 當網域在你的 DNS 主機發佈 SPF、DKIM 與 DMARC 記錄時，來自你網域的郵件更可靠地進入收件匣：

  ```text
  # SPF：根網域（@）上的 TXT 記錄。請使用你的服務供應商對應的 include。
  Google Workspace: v=spf1 include:_spf.google.com ~all
  Microsoft 365:    v=spf1 include:spf.protection.outlook.com ~all
  SendGrid:         v=spf1 include:sendgrid.net ~all
  Brevo:            v=spf1 include:spf.sendinblue.com ~all

  # DMARC：_dmarc.yourdomain.com 上的 TXT 記錄
  v=DMARC1; p=quarantine; rua=mailto:dmarc-reports@yourdomain.com; pct=100; sp=quarantine

  # DKIM：從服務供應商的控制台複製 CNAME／TXT 記錄。
  ```

  每個網域只保留一筆 SPF 記錄（若使用多個供應商，請合併 `include:` 部分），並使用已驗證信箱或網域的寄件者位址。

## seo
`RefineLab_SEO` + trait `RefineLab_SEO_Schema`（`includes/modules/seo/traits/trait-seo-schema.php`）。後台頁面**社群與 SEO**。選項 `refinelab_seo_settings`：

| 群組 | 鍵 |
| :--- | :--- |
| 搜尋 | `enabled`、`home_title`、`home_desc`、`title_separator`、`enable_sitemap` |
| Open Graph／Twitter | `og_enabled`、`default_og_img`、`og_use_featured`、`twitter_card`（`summary_large_image` \| `summary`）、`twitter_site`（`@handle`）、`facebook_app_id` |
| JSON-LD | `schema_enabled`、`schema_type`（Organization、Corporation、LocalBusiness、NGO、EducationalOrganization、Person）、`org_name`、`org_logo`、`social_profiles`（網址，每行一個 → `sameAs`）、`schema_article_type`（Article、BlogPosting、NewsArticle）、`schema_breadcrumbs`、`schema_website_search` |

- **頁面情境**：`build_context_for_request()`／`build_context_for_post( $post )` 回傳單一陣列（種類 `home|singular|term|archive|search|none`、標題、描述、含尺寸／替代文字的圖片、canonical、og 類型、robots、每篇文章的結構化資料覆寫），供中繼輸出、預覽與 JSON-LD 產生器使用。靜態首頁視為 `home`。
- **Open Graph 輸出**（`wp_head`，僅在 `og_enabled` 時）：`og:locale/site_name/type/title/url/description/image(+width,height,alt)`、`article:published_time/modified_time`（文章）、`fb:app_id`、`twitter:card/site/title/description/image(+alt)`。圖片順序：文章的 OG 圖片 → 特色圖片（`og_use_featured`）→ 預設圖片。圖片尺寸每個網址只查詢一次（暫存資料 `refinelab_og_<md5>`）。
- **編輯器方塊**（所有公開文章類型；篩選器 `refinelab_seo_post_types`）：即時的 **Google**、**Facebook／LinkedIn** 與 **X** 預覽（區塊與傳統編輯器；隨標題、摘要、特色圖片與欄位更新）、字數計數器、每篇文章的 SEO 標題／描述、社群標題／描述／圖片（媒體庫）、noindex、結構化資料類型，以及（對具有 `unfiltered_html` 的使用者）額外的 JSON-LD，並可唯讀檢視產生的 JSON-LD。
- **自動 JSON-LD**：每個頁面一個 `@graph`：Organization／Person（標誌、sameAs）、WebSite（可選的 SearchAction）、WebPage／CollectionPage／SearchResultsPage、文章類型節點（只含作者名稱，不含作者網址，以避免洩漏使用者名稱）、ImageObject、BreadcrumbList（頁面 → 祖先頁面，文章 → 分類鏈，詞彙 → 祖先詞彙），並合併每篇文章的自訂 JSON。所有節點以 `@id` 連結。空值會被修剪；輸出以 `JSON_HEX_TAG|JSON_HEX_AMP` 編碼。篩選器 `refinelab_seo_schema_graph( $graph, $ctx )` 可編輯節點（鍵 `organization`、`website`、`webpage`、`image`、`article`、`breadcrumb`、`custom_*`）。
- 每篇文章的中繼資料（`_refinelab_seo_*`）：`title`、`desc`、`og_title`、`og_desc`、`og_image`、`og_image_id`、`noindex`、`nofollow`、`schema_type`、`schema_json`。
- 網站地圖：重寫規則 `^sitemap\.xml$` → `?refinelab_sitemap=1`，在 `template_redirect` 渲染。
- 資源：`assets/js/admin/metabox-seo.js` + `css/admin/metabox-seo.css`（編輯器）、`tab-seo.js`（設定頁面）。

## security
見 [security.md](security.md)：強化標頭、XML-RPC、使用者列舉、登入限制器、自訂登入網址、隱藏 WordPress。

## svg-upload
`RefineLab_SVG_Upload`。旗標 `enable_svg_upload`。篩選器 `upload_mimes`、`wp_check_filetype_and_ext`、`wp_handle_upload_prefilter`（`RefineLab_SVG_Upload::sanitize_svg()`：以 `LIBXML_NONET` 的 `DOMDocument` 解析；拒絕 DOCTYPE／實體與非 `<svg>` 根元素；元素允許清單（`allowed_elements()`）；移除 `on*`、外部 `href`／`url()`、`javascript:`／`data:` 值與不安全的 CSS；`<use>` 與 `href` 只能指向 `#id`，`<image>` 只能使用點陣圖 data URI；儲存的檔案是清理後的標記；不允許 `.svgz`）、`wp_prepare_attachment_for_js`（尺寸）。

## custom-types
`RefineLab_Custom_Types` + traits `Registrar`、`Ajax`。選項 `refinelab_custom_post_types`、`refinelab_custom_taxonomies`；暫存資料 `refinelab_flush_rewrite_rules` 會觸發重寫規則更新。
- 在 `init` 以宣告方式註冊 CPT 與分類法。AJAX `refinelab_save_cpt`、`refinelab_delete_cpt`、`refinelab_save_taxonomy`、`refinelab_delete_taxonomy`（僅限後台，檢查 nonce 與權限）。

## content
`RefineLab_Content`。由佈景主題選擇啟用：

```php
add_theme_support( 'refinelab-content', array(
	'taxonomies' => array( 'category' ),
	'zones'      => array(
		'hero' => array( 'label' => 'Hero', 'default' => 'theme/hero', 'blocks' => array() ),
	),
) );
```

- 隱藏的 CPT `refinelab_content`；區域儲存在 `_refinelab_zone`；語言儲存在 `_refinelab_lang`（與多語言模組共用，多語言模組透過 `refinelab_multilang_post_types` 取得此 CPT）。
- 區塊 `refinelab/content` 預覽內容，並在內含真正區塊編輯器的視窗中編輯（資源 `content-block.js`、`content-modal.js`、`content.css`）。模式：`term`（目前詞彙的區域，支援父層繼承，單篇文章使用其詞彙）與 `entry`（獨立可重複使用的項目）。
- 預覽端點 `?refinelab_content_preview=<id>`（nonce）。短代碼 `[refinelab_content id="" zone="" term=""]`。
- Admin-post 動作 `refinelab_create_content`、`refinelab_delete_content`。
