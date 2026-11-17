# 掛鉤、函式與端點

[English](../hooks-and-api.md)

開發者可以呼叫或擴充的一切，附範例與適用情境。只列出程式碼中確實存在的項目。前綴：`refinelab_`（選項、掛鉤、動作）、`RefineLab_`（類別）、`.refinelab-*`（CSS）、`REFINELAB_WP_*`（常數）。

## 1. PHP 篩選器

| 篩選器 | 參數 | 預設／回傳 | 執行時機 |
| :--- | :--- | :--- | :--- |
| `refinelab_html_blocks_directories` | `string[] $dirs` | `<theme>/blocks`、`<theme>/template-parts/blocks`、`<theme>/html-blocks`（及父佈景主題的） | 掃描區塊範本時。 |
| `refinelab_html_blocks_category_slug` | `string $slug, string $theme` | `theme` | 沒有 `data-category` 的區塊所屬的插入器分類。 |
| `refinelab_html_blocks_category_title` | `string $title, string $theme` | 佈景主題名稱 | 同上，標籤部分。 |
| `refinelab_html_blocks_global_attributes` | `array $globals` | 見 [html-blocks.md §9](html-blocks.md) | 建立範本情境物件（`site`、`context`、`post`、…）時。 |
| `refinelab_html_blocks_breadcrumb_list` | `array $crumbs, WP_Post|null $post` | 依目前請求產生的路徑 | 麵包屑資料。每個項目：`title`、`url`、`is_current`、`is_last`。 |
| `refinelab_multilang_post_types` | `string[] $types` | 公開文章類型 | 哪些文章類型會有語言介面。 |
| `refinelab_seo_post_types` | `string[] $types` | 公開文章類型 | 哪些文章類型會有 SEO 方塊與預覽。 |
| `refinelab_seo_schema_graph` | `array $graph, array $ctx` | 產生的 JSON-LD 節點 | 輸出前編輯結構化資料。 |
| `refinelab_forms_rate_limit` | `array{max:int,window:int} $limit, string $form_id` | 來自設定（window 以秒計） | 每個表單的提交限制。 |
| `refinelab_forms_recaptcha_min_score` | `float` | `0.5` | reCAPTCHA v3 門檻。 |
| `refinelab_security_client_ip` | `string $ip` | `REMOTE_ADDR` | 登入限制器、活動記錄與表單頻率限制使用的訪客 IP。 |
| `refinelab_security_rest_allowed_namespaces` | `string[]` | `refinelab`、`oembed` | 「限制 REST API」開啟時，訪客可使用的 REST 命名空間。 |
| `refinelab_backup_options` | `string[] $names` | 所有 `refinelab_*` 選項（內部使用的除外） | 哪些選項會進入設定快照與匯出。 |
| `refinelab_tools_sanitize_settings` | `array $sanitized, array $input` | — | 主選項 `refinelab_tools_settings` 的清理函式。 |
| `refinelab_remove_data_on_uninstall` | `bool` | `false` | 回傳 `true` 會在刪除外掛時刪除所有外掛資料。 |
| `refinelab_update_manifest_url` | `string $url` | 發佈儲存庫的 `update.json` | 重新導向更新檢查（測試通道）。 |

動作：`refinelab_tools_render_content`（`array $options`）讓模組在 HTML 區塊設定頁繪製卡片。

### 範例

```php
// 1. 也從共用資料夾載入區塊範本。
add_filter( 'refinelab_html_blocks_directories', function ( $dirs ) {
    $dirs[] = WP_CONTENT_DIR . '/shared-blocks';
    return $dirs;
} );

// 2. 把所有區塊放進你自己的插入器分類。
add_filter( 'refinelab_html_blocks_category_slug',  fn() => 'acme' );
add_filter( 'refinelab_html_blocks_category_title', fn() => 'Acme blocks' );

// 3. 為每個範本提供額外資料：{{ brand.phone }}
add_filter( 'refinelab_html_blocks_global_attributes', function ( $globals ) {
    $globals['brand'] = array( 'phone' => get_option( 'acme_phone' ) );
    return $globals;
} );

// 4. 重新命名首頁麵包屑。
add_filter( 'refinelab_html_blocks_breadcrumb_list', function ( $crumbs ) {
    if ( isset( $crumbs[0] ) ) { $crumbs[0]['title'] = 'Start'; }
    return $crumbs;
} );

// 5. 在 JSON-LD 圖譜加入組織資料。
add_filter( 'refinelab_seo_schema_graph', function ( $graph, $ctx ) {
    if ( isset( $graph['organization'] ) ) {
        $graph['organization']['telephone'] = '+84 28 0000 0000';
    }
    return $graph;
}, 10, 2 );

// 6. 為自訂文章類型提供語言介面。
add_filter( 'refinelab_multilang_post_types', fn( $t ) => array_merge( $t, array( 'portfolio' ) ) );

// 7. 在設定備份中多保留一個選項。
add_filter( 'refinelab_backup_options', function ( $names ) {
    $names[] = 'refinelab_my_addon_settings'; // 必須以 refinelab_ 開頭
    return $names;
} );

// 8. 測試用更新通道（僅限你掌控的非正式站）。
add_filter( 'refinelab_update_manifest_url', fn() => 'https://raw.githubusercontent.com/hoquangthaiholy/refine-lab-wp-releases/staging/update.json' );
```

`refinelab_seo_schema_graph` 中 JSON-LD 圖譜的鍵：`organization`、`website`、`webpage`、`image`、`article`、`breadcrumb`，以及每篇文章自訂 JSON 使用的 `custom_*`。

## 2. 函式

| 函式 | 回傳 | 說明 |
| :--- | :--- | :--- |
| `refinelab_tools()` | `RefineLab_Tools` 單例 | 存取模組：`refinelab_tools()->security`、`->seo`、`->multilang`、`->html_blocks`、… |
| `refinelab_get_languages( $post_id = null )` | `array[]` | 含連結資料的語言。見 [multilang.md](multilang.md)。 |
| `refinelab_get_current_language()` | `string` | 目前語言代碼（模組關閉時為 `'en'`）。 |
| `refinelab_get_current_language_data( $post_id = null )` | `array` | 目前語言的資料。 |
| `refinelab__( $text, $domain = 'default' )`、`refinelab_e( … )` | `string`／輸出 | 依語言的固定字串。 |
| `RefineLab_Admin_UI::render( $component, $args )`／`::get()` | —／`string` | `templates/admin/components/` 中的後台元件。 |
| `RefineLab_Security::get_settings()`、`RefineLab_SEO::get_settings()`、`RefineLab_Forms::get_settings()`、`RefineLab_Mailer::get_settings()`、`RefineLab_MultiLang::get_settings()`、`RefineLab_Performance::get_settings()` | `array` | 已儲存的設定與預設值合併後的結果。 |
| `RefineLab_Updater::get_status()` | `array` | `current`、`latest`、`available`、`checked`、`error`、`auto`。 |
| `RefineLab_Search_Replace` | 類別 | `new RefineLab_Search_Replace( array $from_urls, string $to_url )`，再呼叫 `->replace( $value )` 或 `->replace_deep( $array )`：對序列化與 JSON 安全的位址取代。 |

```php
// 跟隨目前語言的範本。
$lang = refinelab_get_current_language();
$menu = ( 'vi' === $lang ) ? 'primary-vi' : 'primary';
wp_nav_menu( array( 'theme_location' => $menu ) );

// 在任何儲存的值（含序列化資料）中取代網域。
$replacer = new RefineLab_Search_Replace( array( 'https://old.example.com' ), 'https://new.example.com' );
$fixed    = $replacer->replace( get_option( 'my_option' ) );
```

## 3. 短代碼

| 短代碼 | 屬性 | 用途 |
| :--- | :--- | :--- |
| `[refinelab_language_switcher]`（別名 `[refinelab_lang_switcher]`） | `format="both|flags|names"`、`layout="list|dropdown"` | 語言切換器（[multilang.md](multilang.md)）。 |
| `[refinelab_content id="" zone="" term=""]` | `id`、`zone`、`term` | 渲染一個內容區塊項目（見 [modules.md](modules.md#content)）。 |

## 4. 佈景主題支援

```php
add_theme_support( 'refinelab-content', array(
    'taxonomies' => array( 'category' ),
    'zones'      => array(
        'hero' => array( 'label' => 'Hero', 'default' => 'theme/hero', 'blocks' => array() ),
    ),
) );
```

啟用**內容區塊**：附加在分類法詞彙上的可編輯內容區域，在內含真正區塊編輯器的視窗中編輯。

## 5. JavaScript

| API | 位置 | 用途 |
| :--- | :--- | :--- |
| `window.refineLab.registerBlock( slugOrName, hooks )` | 編輯器與前台 | 生命週期掛鉤：`init`（兩處皆有）、`insert`、`update`、`destroyed`（編輯器）。見 [html-blocks.md §13](html-blocks.md)。 |
| `window.refineLab.getBlockHooks( slugOrName )` | 兩處皆有 | 已註冊的掛鉤。 |
| `refinelab:init` DOM 事件 | 前台，從區塊根元素冒泡 | `event.detail` = `{ el, name, slug, isEditor }`。 |
| `refinelab:insert`、`refinelab:init`、`refinelab:update`、`refinelab:destroyed` DOM 事件 | 編輯器 | 與掛鉤相同的事件。 |
| `refinelab.block.<event>`、`refinelab.block.<slug>.<event>`、`refinelab.block.theme.<slug>.<event>` | 編輯器中的 `wp.hooks` 動作 | 不需註冊掛鉤即可觀察任何區塊。 |
| `refinelab:form:success` DOM 事件 | 前台，在 `document` 上 | 提交成功後的 `detail` = `{ formId, response }`。 |
| `window.refinelabFormsConfig` | 含表單的頁面 | `ajaxUrl`、`nonce`、`honeypot`、`recaptchaKey`、`recaptchaVersion`、`i18n`。 |
| `window.refineLabThemeCategory`、`refineLabThemeBlocks`、`refineLabGlobalAttributes`、`refineLabEditorSettings` | 區塊編輯器 | 編輯器指令碼的本地化資料。 |

瀏覽器儲存：`localStorage['refinelab_sidebar_width']`（可調整寬度的編輯器側邊欄）。

## 6. 端點

**REST**（`current_user_can( 'edit_post', post_id )`）：`GET /wp-json/refinelab/v1/block-history?post_id=<id>&path=<索引路徑>&name=<區塊名稱>`，其中 `path` 是區塊位置，以點號分隔的索引表示（`0`、`2.1.0`），`name` 是其區塊名稱（`theme/hero`）。它會依新到舊回傳該區塊儲存的快照，格式為 `[ { id, time, user, attrs } ]`。每篇文章最多保留 20 份快照（400 KB），每次有變動的儲存會記錄一份；編輯器的*區塊歷史*面板用它來回復區塊。

**公開 AJAX**：只有 `wp_ajax_refinelab_form_submit` 與 `wp_ajax_nopriv_refinelab_form_submit`（表單提交；nonce、honeypot、頻率限制、可選的 reCAPTCHA）。`wp_ajax_nopriv_refinelab_site_backup_step` 只接受帶有管理員建立之有效工作密鑰的請求。

**僅限後台**（檢查 nonce 與權限）：見下方的 `admin_post_refinelab_*` 與 `wp_ajax_refinelab_*`。

| 範圍 | 動作 |
| :--- | :--- |
| HTML 區塊 | `admin_post_refinelab_clear_blocks_cache`、`refinelab_create_sample_theme`、`refinelab_install_sample_blocks` |
| 表單 | `admin_post_refinelab_export_submissions_csv`、`refinelab_download_attachment`；cron `refinelab_forms_deliver_webhook` |
| 郵件 | `wp_ajax_refinelab_send_test_email`、`admin_post_refinelab_mailer_oauth_start|callback|disconnect` |
| 語言 | `admin_post_refinelab_assign_language`、`refinelab_duplicate_translation`、`refinelab_unlink_translation`、`refinelab_translations_bulk`；AJAX `refinelab_delete_language_data`、`refinelab_language_data_summary`、`refinelab_unlink_translation_group`、`refinelab_link_translation_group` |
| 自訂類型 | `wp_ajax_refinelab_save_cpt`、`refinelab_delete_cpt`、`refinelab_save_taxonomy`、`refinelab_delete_taxonomy` |
| 內容區塊 | `admin_post_refinelab_create_content`、`refinelab_delete_content`；預覽 `?refinelab_content_preview=<id>` |
| SEO | `admin_post_refinelab_seo_test_schema`、`refinelab_redirect_save|delete|import|export|clear404` |
| 媒體 | `admin_post_refinelab_media_settings`、`wp_ajax_refinelab_modern_batch` |
| 效能、指令碼、維護 | `admin_post_refinelab_performance_settings`、`refinelab_scripts_settings`、`refinelab_maintenance_save|token` |
| 安全性 | `admin_post_refinelab_integrity_run|baseline|settings|export`、`refinelab_activity_export|settings|clear`；cron `refinelab_integrity_scan`、`refinelab_activity_prune` |
| 備份 | `admin_post_refinelab_backup_create|restore|delete|download`、`refinelab_export`、`refinelab_import_upload|apply`、`refinelab_site_backup_download|delete|upload`；`wp_ajax_refinelab_site_backup_step`；獨立的 `restore-runner.php` |
| 更新 | `admin_post_refinelab_check_updates`、`refinelab_toggle_auto_update` |

## 7. 常數

請定義在 `wp-config.php`。

| 常數 | 效果 |
| :--- | :--- |
| `REFINELAB_WP_REMOVE_DATA` | 在刪除外掛前設為 `true`，會移除其選項、資料表、項目、中繼資料與上傳的檔案。預設全部保留。 |
| `REFINELAB_WP_DISABLE_SCRIPTS` | 設為 `true` 會停止指令碼模組的所有輸出（某段程式碼讓頁面壞掉時的復原方式）。 |
| `REFINELAB_DISABLE_CUSTOM_LOGIN` | 略過自訂登入網址。 |
| `REFINELAB_DISABLE_2FA` | 略過雙重驗證檢查（緊急存取）。 |
| `REFINELAB_DISABLE_IP_RESTRICTION` | 略過後台 IP 允許清單。 |
| `REFINELAB_WP_UPDATE_URL` | 覆寫更新資訊清單的位址（HTTPS；只有在 `wp_get_environment_type()` 為 `local` 時才接受非發佈位址）。 |
| `SCRIPT_DEBUG` | 載入未壓縮的外掛資源。 |

唯讀：`REFINELAB_WP_VERSION`、`REFINELAB_WP_FILE`、`REFINELAB_WP_PATH`、`REFINELAB_WP_URL`、`REFINELAB_WP_BASENAME`（以及舊版別名 `REFINELAB_TOOLS_*`）。

## 8. 外掛儲存的資料

| 種類 | 鍵 |
| :--- | :--- |
| 選項 | `refinelab_tools_settings`、`refinelab_security_settings`、`refinelab_mailer_settings`、`refinelab_forms_settings`、`refinelab_seo_settings`、`refinelab_multilang_settings`、`refinelab_multilang_strings`、`refinelab_performance_settings`、`refinelab_media_settings`、`refinelab_scripts_settings`、`refinelab_maintenance_settings`、`refinelab_activity_settings`、`refinelab_integrity_settings`、`refinelab_integrity_baseline`、`refinelab_integrity_report`、`refinelab_custom_post_types`、`refinelab_custom_taxonomies`、`refinelab_settings_backups`、`refinelab_site_backups`、`refinelab_forms_webhook_log`、`refinelab_redirects_db` |
| 資料表 | `{prefix}refinelab_activity`、`{prefix}refinelab_redirects`、`{prefix}refinelab_404` |
| 文章類型 | `refine_entry`（表單項目）、`refinelab_content`（內容區塊，隱藏） |
| 文章中繼資料 | `_refinelab_lang`、`_refinelab_translation_group`、`_refinelab_seo_*`、`_refinelab_zone`、`_refinelab_block_history`、`_refinelab_modern_images`、`_refinelab_form_*`、`_refinelab_fields`、`_refinelab_attachments`、`_refinelab_webhook` |
| 使用者中繼資料 | `_refinelab_2fa_*` |
| 暫存資料 | `refinelab_blocks_<md5>`、`refinelab_update_manifest`、`refinelab_og_<md5>`、`refinelab_ll_<md5>`、`refinelab_rl_<md5>`、`refinelab_flush_rewrite_rules` |
| 檔案 | `uploads/refinelab-forms/`（表單附件）、`uploads/refinelab-backups/`（完整備份） |

## 9. CSS

外掛的前台與後台 CSS 只使用 `.refinelab-*`，絕不針對佈景主題。適合用於佈景設計的掛鉤：`.refinelab-lang-switcher`、`.refinelab-layout-list`、`.refinelab-layout-dropdown`、`.refinelab-lang-flag`、`.refinelab-lang-name`、`.lang-item`、`.is-active`；區塊內有 `.refinelab-has-active-item`、`.is-item-active`（見使用 `data-index` 的重複器）。在編輯器中，請將你的規則限定在 `.editor-styles-wrapper`。
