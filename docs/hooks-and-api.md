# Hooks, functions and endpoints

**Language:** English · [繁體中文](zh-TW/hooks-and-api.md)

Everything a developer can call or extend, with an example and the context to use it in. Only items that exist in the code are listed. Prefixes: `refinelab_` (options, hooks, actions), `RefineLab_` (classes), `.refinelab-*` (CSS), `REFINELAB_WP_*` (constants).

## 1. PHP filters

| Filter | Arguments | Default / return | When it runs |
| :--- | :--- | :--- | :--- |
| `refinelab_html_blocks_directories` | `string[] $dirs` | `<theme>/blocks`, `<theme>/template-parts/blocks`, `<theme>/html-blocks` (and the parent theme's) | Scanning for block templates. |
| `refinelab_html_blocks_category_slug` | `string $slug, string $theme` | `theme` | Inserter category of blocks without `data-category`. |
| `refinelab_html_blocks_category_title` | `string $title, string $theme` | Theme name | Same, the label. |
| `refinelab_html_blocks_global_attributes` | `array $globals` | See [html-blocks.md §9](html-blocks.md) | Building the template context objects (`site`, `context`, `post`, …). |
| `refinelab_html_blocks_breadcrumb_list` | `array $crumbs, WP_Post|null $post` | Trail from the current request | Breadcrumb data. Each crumb: `title`, `url`, `is_current`, `is_last`. |
| `refinelab_multilang_post_types` | `string[] $types` | Public post types | Which post types get the language UI. |
| `refinelab_seo_post_types` | `string[] $types` | Public post types | Which post types get the SEO box and previews. |
| `refinelab_seo_schema_graph` | `array $graph, array $ctx` | Generated JSON-LD nodes | Edit structured data before output. |
| `refinelab_forms_rate_limit` | `array{max:int,window:int} $limit, string $form_id` | From settings (window in seconds) | Per-form submission limit. |
| `refinelab_forms_recaptcha_min_score` | `float` | `0.5` | reCAPTCHA v3 threshold. |
| `refinelab_security_client_ip` | `string $ip` | `REMOTE_ADDR` | Visitor IP for the login limiter, activity log and form rate limit. |
| `refinelab_security_rest_allowed_namespaces` | `string[]` | `refinelab`, `oembed` | REST namespaces that guests may use when "Restrict REST API" is on. |
| `refinelab_backup_options` | `string[] $names` | All `refinelab_*` options except internal ones | Which options go into settings snapshots and exports. |
| `refinelab_tools_sanitize_settings` | `array $sanitized, array $input` | — | Sanitizer of the main option `refinelab_tools_settings`. |
| `refinelab_remove_data_on_uninstall` | `bool` | `false` | Return `true` to delete all plugin data when the plugin is deleted. |
| `refinelab_update_manifest_url` | `string $url` | The release repository's `update.json` | Redirect update checks (staging channel). |

Actions: `refinelab_tools_render_content` (`array $options`) lets a module draw a card on the HTML blocks settings page.

### Examples

```php
// 1. Load block templates from a shared folder as well.
add_filter( 'refinelab_html_blocks_directories', function ( $dirs ) {
    $dirs[] = WP_CONTENT_DIR . '/shared-blocks';
    return $dirs;
} );

// 2. Put every block in your own inserter category.
add_filter( 'refinelab_html_blocks_category_slug',  fn() => 'acme' );
add_filter( 'refinelab_html_blocks_category_title', fn() => 'Acme blocks' );

// 3. Expose extra data to every template: {{ brand.phone }}
add_filter( 'refinelab_html_blocks_global_attributes', function ( $globals ) {
    $globals['brand'] = array( 'phone' => get_option( 'acme_phone' ) );
    return $globals;
} );

// 4. Rename the home crumb.
add_filter( 'refinelab_html_blocks_breadcrumb_list', function ( $crumbs ) {
    if ( isset( $crumbs[0] ) ) { $crumbs[0]['title'] = 'Start'; }
    return $crumbs;
} );

// 5. Add organisation data to the JSON-LD graph.
add_filter( 'refinelab_seo_schema_graph', function ( $graph, $ctx ) {
    if ( isset( $graph['organization'] ) ) {
        $graph['organization']['telephone'] = '+84 28 0000 0000';
    }
    return $graph;
}, 10, 2 );

// 6. Language UI for a custom post type.
add_filter( 'refinelab_multilang_post_types', fn( $t ) => array_merge( $t, array( 'portfolio' ) ) );

// 7. Keep an extra option in settings backups.
add_filter( 'refinelab_backup_options', function ( $names ) {
    $names[] = 'refinelab_my_addon_settings'; // must start with refinelab_
    return $names;
} );

// 8. Staging update channel (only on a non-production site you control).
add_filter( 'refinelab_update_manifest_url', fn() => 'https://raw.githubusercontent.com/hoquangthaiholy/refine-lab-wp-releases/staging/update.json' );
```

Keys of the JSON-LD graph in `refinelab_seo_schema_graph`: `organization`, `website`, `webpage`, `image`, `article`, `breadcrumb`, and `custom_*` for per-post JSON.

## 2. Functions

| Function | Returns | Notes |
| :--- | :--- | :--- |
| `refinelab_tools()` | `RefineLab_Tools` singleton | Module access: `refinelab_tools()->security`, `->seo`, `->multilang`, `->html_blocks`, … |
| `refinelab_get_languages( $post_id = null )` | `array[]` | Languages with link data. See [multilang.md](multilang.md). |
| `refinelab_get_current_language()` | `string` | Active language code (`'en'` when the module is off). |
| `refinelab_get_current_language_data( $post_id = null )` | `array` | Data of the active language. |
| `refinelab__( $text, $domain = 'default' )`, `refinelab_e( … )` | `string` / echo | Per-language fixed strings. |
| `RefineLab_Admin_UI::render( $component, $args )` / `::get()` | — / `string` | Admin components in `templates/admin/components/`. |
| `RefineLab_Security::get_settings()`, `RefineLab_SEO::get_settings()`, `RefineLab_Forms::get_settings()`, `RefineLab_Mailer::get_settings()`, `RefineLab_MultiLang::get_settings()`, `RefineLab_Performance::get_settings()` | `array` | Saved settings merged with defaults. |
| `RefineLab_Updater::get_status()` | `array` | `current`, `latest`, `available`, `checked`, `error`, `auto`. |
| `RefineLab_Search_Replace` | class | `new RefineLab_Search_Replace( array $from_urls, string $to_url )`, then `->replace( $value )` or `->replace_deep( $array )`: serialized- and JSON-safe address replacement. |

```php
// A template that follows the active language.
$lang = refinelab_get_current_language();
$menu = ( 'vi' === $lang ) ? 'primary-vi' : 'primary';
wp_nav_menu( array( 'theme_location' => $menu ) );

// Replace a domain inside any stored value, serialized data included.
$replacer = new RefineLab_Search_Replace( array( 'https://old.example.com' ), 'https://new.example.com' );
$fixed    = $replacer->replace( get_option( 'my_option' ) );
```

## 3. Shortcodes

| Shortcode | Attributes | Purpose |
| :--- | :--- | :--- |
| `[refinelab_language_switcher]` (alias `[refinelab_lang_switcher]`) | `format="both|flags|names"`, `layout="list|dropdown"` | Language switcher ([multilang.md](multilang.md)). |
| `[refinelab_content id="" zone="" term=""]` | `id`, `zone`, `term` | Render a Content Blocks item (see [modules.md](modules.md#content)). |

## 4. Theme support

```php
add_theme_support( 'refinelab-content', array(
    'taxonomies' => array( 'category' ),
    'zones'      => array(
        'hero' => array( 'label' => 'Hero', 'default' => 'theme/hero', 'blocks' => array() ),
    ),
) );
```

Enables **Content Blocks**: editable content zones attached to taxonomy terms, edited in a modal that hosts the real block editor.

## 5. JavaScript

| API | Where | Purpose |
| :--- | :--- | :--- |
| `window.refineLab.registerBlock( slugOrName, hooks )` | editor and front end | Lifecycle hooks `init` (both), `insert`, `update`, `destroyed` (editor). See [html-blocks.md §13](html-blocks.md). |
| `window.refineLab.getBlockHooks( slugOrName )` | both | Registered hooks. |
| `refinelab:init` DOM event | front end, bubbles from the block root | `event.detail` = `{ el, name, slug, isEditor }`. |
| `refinelab:insert`, `refinelab:init`, `refinelab:update`, `refinelab:destroyed` DOM events | editor | Same events as the hooks. |
| `refinelab.block.<event>`, `refinelab.block.<slug>.<event>`, `refinelab.block.theme.<slug>.<event>` | `wp.hooks` actions in the editor | Observe any block without registering hooks. |
| `refinelab:form:success` DOM event | front end, on `document` | `detail` = `{ formId, response }` after a successful submission. |
| `window.refinelabFormsConfig` | pages with a form | `ajaxUrl`, `nonce`, `honeypot`, `recaptchaKey`, `recaptchaVersion`, `i18n`. |
| `window.refineLabThemeCategory`, `refineLabThemeBlocks`, `refineLabGlobalAttributes`, `refineLabEditorSettings` | block editor | Localized data of the editor script. |

Browser storage: `localStorage['refinelab_sidebar_width']` (resizable editor sidebar).

## 6. Endpoints

**REST** (`current_user_can( 'edit_post', post_id )`): `GET /wp-json/refinelab/v1/block-history?post_id=<id>&path=<index path>&name=<block name>`, where `path` is the block's position as dot-separated indexes (`0`, `2.1.0`) and `name` its block name (`theme/hero`). It returns the saved snapshots of that block, newest first, as `[ { id, time, user, attrs } ]`. Up to 20 snapshots (400 KB) are kept per post and one is recorded on each save that changed something; the editor's *Block history* panel uses it to roll a block back.

**Media folders REST** (`upload_files`; assigning also needs `edit_post` on each file): `GET /wp-json/refinelab/v1/media-folders` returns `{ folders: [ { id, name, parent, count } ], total, unassigned }`; `POST /media-folders` with `{ name, parent }` creates a folder (names up to 80 characters, nesting up to 5 levels, duplicates under the same parent give 409); `PATCH /media-folders/<id>` with `{ name, parent }` renames or moves it (a move into itself or a subfolder is refused); `DELETE /media-folders/<id>` deletes it (files become unassigned, subfolders move up); `POST /media-folders/assign` with `{ ids: [..up to 200], folder }` moves files (`folder` 0 = unassigned) and returns the new state plus `moved` / `denied`. The media queries accept `refinelab_folder` (a folder id, or `none`): in the media grid and modal as the `query[refinelab_folder]` property, in list mode as `upload.php?refinelab_folder=`. New uploads join the folder sent as `refinelab_folder` with the upload request.

**Public AJAX**: only `wp_ajax_refinelab_form_submit` and `wp_ajax_nopriv_refinelab_form_submit` (form submission; nonce, honeypot, rate limit, optional reCAPTCHA). `wp_ajax_nopriv_refinelab_site_backup_step` accepts only requests that carry a valid per-job secret created by an administrator.

**Admin-only** (nonce and capability checked): see `admin_post_refinelab_*` and `wp_ajax_refinelab_*` below.

| Area | Actions |
| :--- | :--- |
| HTML blocks | `admin_post_refinelab_clear_blocks_cache`, `refinelab_create_sample_theme`, `refinelab_install_sample_blocks` |
| Forms | `admin_post_refinelab_export_submissions_csv`, `refinelab_download_attachment`; cron `refinelab_forms_deliver_webhook` |
| Mailer | `wp_ajax_refinelab_send_test_email`, `admin_post_refinelab_mailer_oauth_start|callback|disconnect` |
| Languages | `admin_post_refinelab_assign_language`, `refinelab_duplicate_translation`, `refinelab_unlink_translation`, `refinelab_translations_bulk`; AJAX `refinelab_delete_language_data`, `refinelab_language_data_summary`, `refinelab_unlink_translation_group`, `refinelab_link_translation_group` |
| Custom types | `wp_ajax_refinelab_save_cpt`, `refinelab_delete_cpt`, `refinelab_save_taxonomy`, `refinelab_delete_taxonomy` |
| Content blocks | `admin_post_refinelab_create_content`, `refinelab_delete_content`; preview `?refinelab_content_preview=<id>` |
| SEO | `admin_post_refinelab_seo_test_schema`, `refinelab_redirect_save|delete|import|export|clear404` |
| Media | `admin_post_refinelab_media_settings`, `wp_ajax_refinelab_modern_batch` |
| Performance, scripts, maintenance | `admin_post_refinelab_performance_settings`, `refinelab_scripts_settings`, `refinelab_maintenance_save|token` |
| Security | `admin_post_refinelab_integrity_run|baseline|settings|export`, `refinelab_activity_export|settings|clear`; cron `refinelab_integrity_scan`, `refinelab_activity_prune` |
| Media folders | `admin_post_refinelab_media_folders_settings`; REST routes above |
| Backup | `admin_post_refinelab_backup_create|restore|delete|download`, `refinelab_export`, `refinelab_import_upload|apply`, `refinelab_site_backup_download|delete|upload`; `wp_ajax_refinelab_site_backup_step`; standalone `restore-runner.php` |
| Updates | `admin_post_refinelab_check_updates`, `refinelab_toggle_auto_update` |

## 7. Constants

Define these in `wp-config.php`.

| Constant | Effect |
| :--- | :--- |
| `REFINELAB_WP_REMOVE_DATA` | `true` before deleting the plugin removes its options, tables, entries, meta and uploaded files. Default keeps everything. |
| `REFINELAB_WP_DISABLE_SCRIPTS` | `true` stops all output of the Scripts module (recovery if a snippet breaks pages). |
| `REFINELAB_DISABLE_CUSTOM_LOGIN` | Bypass the custom login URL. |
| `REFINELAB_DISABLE_2FA` | Bypass two-factor checks (emergency access). |
| `REFINELAB_DISABLE_IP_RESTRICTION` | Bypass the back-end IP allowlist. |
| `REFINELAB_WP_UPDATE_URL` | Override the update manifest address (HTTPS; non-release addresses are accepted only when `wp_get_environment_type()` is `local`). |
| `SCRIPT_DEBUG` | Load unminified plugin assets. |

Read-only: `REFINELAB_WP_VERSION`, `REFINELAB_WP_FILE`, `REFINELAB_WP_PATH`, `REFINELAB_WP_URL`, `REFINELAB_WP_BASENAME` (and legacy `REFINELAB_TOOLS_*` aliases).

## 8. Data the plugin stores

| Kind | Keys |
| :--- | :--- |
| Options | `refinelab_tools_settings`, `refinelab_security_settings`, `refinelab_mailer_settings`, `refinelab_forms_settings`, `refinelab_seo_settings`, `refinelab_multilang_settings`, `refinelab_multilang_strings`, `refinelab_performance_settings`, `refinelab_media_settings`, `refinelab_scripts_settings`, `refinelab_maintenance_settings`, `refinelab_activity_settings`, `refinelab_integrity_settings`, `refinelab_integrity_baseline`, `refinelab_integrity_report`, `refinelab_custom_post_types`, `refinelab_custom_taxonomies`, `refinelab_settings_backups`, `refinelab_site_backups`, `refinelab_forms_webhook_log`, `refinelab_redirects_db` |
| Tables | `{prefix}refinelab_activity`, `{prefix}refinelab_redirects`, `{prefix}refinelab_404` |
| Taxonomy | `refinelab_media_folder` (hierarchical, attachments only, no public UI): media folders |
| Post types | `refine_entry` (form entries), `refinelab_content` (content blocks, hidden) |
| Post meta | `_refinelab_lang`, `_refinelab_translation_group`, `_refinelab_seo_*`, `_refinelab_zone`, `_refinelab_block_history`, `_refinelab_modern_images`, `_refinelab_form_*`, `_refinelab_fields`, `_refinelab_attachments`, `_refinelab_webhook` |
| User meta | `_refinelab_2fa_*` |
| Transients | `refinelab_blocks_<md5>`, `refinelab_update_manifest`, `refinelab_og_<md5>`, `refinelab_ll_<md5>`, `refinelab_rl_<md5>`, `refinelab_flush_rewrite_rules` |
| Files | `uploads/refinelab-forms/` (form attachments), `uploads/refinelab-backups/` (full backups) |

## 9. CSS

Plugin front-end and admin CSS uses `.refinelab-*` only and never targets a theme. Useful hooks for theming: `.refinelab-lang-switcher`, `.refinelab-layout-list`, `.refinelab-layout-dropdown`, `.refinelab-lang-flag`, `.refinelab-lang-name`, `.lang-item`, `.is-active`; inside blocks `.refinelab-has-active-item`, `.is-item-active` (see repeaters with `data-index`). In the editor, scope your rules to `.editor-styles-wrapper`.
