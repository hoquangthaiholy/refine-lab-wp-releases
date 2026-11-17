# 開發者參考

[English](../developer-reference.md)

## 常數（`refine-lab-wp.php`）

| 常數 | 意義 |
| :--- | :--- |
| `REFINELAB_WP_VERSION`、`_FILE`、`_PATH`、`_URL`、`_BASENAME` | 外掛中繼資料與路徑。請同時更新標頭版本與 `REFINELAB_WP_VERSION`。 |
| `REFINELAB_TOOLS_VERSION/_FILE/_PATH/_URL/_BASENAME` | 上述常數的舊版別名。請保留。 |
| `REFINELAB_DISABLE_CUSTOM_LOGIN` | 選用，在 `wp-config.php` 定義以略過自訂登入網址。 |
| `REFINELAB_DISABLE_2FA` | 選用，在 `wp-config.php` 定義以略過雙重驗證檢查。 |
| `REFINELAB_WP_REMOVE_DATA` | 選用，在刪除外掛前於 `wp-config.php` 定義為 `true`，會一併移除選項、資料表、項目、中繼資料與上傳檔案（見 `uninstall.php`；預設保留資料）。篩選器：`refinelab_remove_data_on_uninstall`。 |
| `REFINELAB_DISABLE_IP_RESTRICTION` | 選用，在 `wp-config.php` 定義以略過後台 IP 允許清單。 |
| `SCRIPT_DEBUG` | 載入未壓縮的資源（`get_asset_suffix()`）。 |

完整常數清單（含更新與指令碼模組）見 [hooks-and-api.md §7](hooks-and-api.md)。

## 選項

| 選項 | 擁有者 |
| :--- | :--- |
| `refinelab_tools_settings` | 管理：`enable_html_blocks`、`enable_svg_upload`、`enable_sticky_post_title`、`enable_resizable_sidebar` |
| `refinelab_security_settings` | 安全性，見 [security.md](security.md) |
| `refinelab_mailer_settings`、`refinelab_forms_settings`、`refinelab_seo_settings`、`refinelab_multilang_settings`、`refinelab_multilang_strings` | 各自的模組 |
| `refinelab_custom_post_types`、`refinelab_custom_taxonomies` | 自訂類型 |
| `refinelab_migrated_to_refine_entry` | 一次性遷移旗標（表單） |

暫存資料：`refinelab_blocks_<md5(stylesheet)>`（區塊定義）、`refinelab_flush_rewrite_rules`、`refinelab_ll_<md5(ip)>`（登入失敗計數）。
使用者中繼資料：`_refinelab_2fa_*`（安全性的雙重驗證）。
文章中繼資料：`_refinelab_lang`、`_refinelab_translation_group`、`_refinelab_seo_{title,desc,og_title,og_desc,og_image,og_image_id,noindex,nofollow,schema_type,schema_json}`、`_refinelab_zone`。

所有儲存資料的完整清單：[hooks-and-api.md §8](hooks-and-api.md)。

## 公開 PHP API

- `refinelab_tools()` 回傳 `RefineLab_Tools` 單例；透過它的魔術 getter 存取模組：`refinelab_tools()->security`、`->seo`、`->multilang`。（`get_module()` 只存在於 `RefineLab_Plugin`，不在 `RefineLab_Tools`。）
- 模組設定：`RefineLab_<Module>::get_settings()`（類別中標明為 static 者）與 `OPTION_KEY` 常數。
- 後台標記使用 `RefineLab_Admin_UI::render()`／`::get()`。

## 篩選器

| 篩選器 | 參數 | 位置 |
| :--- | :--- | :--- |
| `refinelab_html_blocks_directories` | `string[] $dirs` | 掃描資料夾 |
| `refinelab_html_blocks_category_slug`／`_category_title` | `$value, $theme` | 預設分類 |
| `refinelab_html_blocks_global_attributes` | `array $globals` | 新增／變更範本全域變數 |
| `refinelab_html_blocks_breadcrumb_list` | `array $crumbs` | 麵包屑項目 |
| `refinelab_forms_recaptcha_min_score` | `float`（0.5） | reCAPTCHA v3 門檻 |
| `refinelab_multilang_post_types` | `string[]` | 有語言介面的文章類型 |
| `refinelab_tools_sanitize_settings` | `array $sanitized, $input` | 主選項的清理函式 |
| `refinelab_seo_post_types` | `string[]` | 有 SEO 方塊與預覽的文章類型 |
| `refinelab_seo_schema_graph` | `array $graph, array $ctx` | 編輯 JSON-LD 節點 |
| `refinelab_security_rest_allowed_namespaces` | `string[]` | 訪客可用的 REST 命名空間 |
| `refinelab_security_client_ip` | `string` | 登入限制器使用的 IP |

動作：`refinelab_tools_render_content`（後台卡片）。經過確認的完整掛鉤、函式與端點清單（含範例）在 [hooks-and-api.md](hooks-and-api.md)。

## AJAX 與 admin-post 端點

完整端點清單：[hooks-and-api.md §6](hooks-and-api.md)。所有具權限的處理程式都會驗證 nonce 與 `current_user_can()`；公開的只有 `refinelab_form_submit`。

## 區塊編輯器中的 JavaScript 全域變數

本地化到 `refinelab-html-blocks-editor`：

| 全域變數 | 內容 |
| :--- | :--- |
| `refineLabThemeCategory` | 佈景主題區塊分類的 `{ slug, title }` |
| `refineLabThemeBlocks` | 偵測到的區塊定義 |
| `refineLabGlobalAttributes` | 範本全域變數（見 html-blocks.md §9），含目前語言 |
| `refineLabEditorSettings` | `{ stickyTitle, resizableSidebar, stickyTitleCssUrl }` |

瀏覽器儲存：`localStorage['refinelab_sidebar_width']`。CSS 變數 `--refinelab-sidebar-width`；body class `refinelab-resizable-sidebar`（後台）限定側邊欄 CSS 的範圍。前台的 DOM 事件 `refinelab:form:success`。

## CSS／命名

條件式列：`setting-row` 的參數 `depends`／`depends_on`（`core-admin.js` 中的 `data-refinelab-depends`）。

僅使用 `.refinelab-*`；外掛不使用任何佈景主題選擇器。編輯器規則請使用 `.editor-styles-wrapper`、`.block-editor-*`、`.components-*`。
