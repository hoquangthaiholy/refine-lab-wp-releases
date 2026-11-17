# Developer reference

**Language:** English · [繁體中文](zh-TW/developer-reference.md)

## Constants (`refine-lab-wp.php`)

| Constant | Meaning |
| :--- | :--- |
| `REFINELAB_WP_VERSION`, `_FILE`, `_PATH`, `_URL`, `_BASENAME` | Plugin metadata and paths. Bump the header version and `REFINELAB_WP_VERSION` together. |
| `REFINELAB_TOOLS_VERSION/_FILE/_PATH/_URL/_BASENAME` | Legacy aliases of the above. Keep. |
| `REFINELAB_DISABLE_CUSTOM_LOGIN` | Optional, define in `wp-config.php` to bypass the custom login URL. |
| `REFINELAB_DISABLE_2FA` | Optional, define in `wp-config.php` to bypass two-factor checks. |
| `REFINELAB_WP_REMOVE_DATA` | Optional, define `true` in `wp-config.php` before deleting the plugin to also remove options, tables, entries, meta and uploads (see `uninstall.php`; default keeps data). Filter: `refinelab_remove_data_on_uninstall`. |
| `REFINELAB_DISABLE_IP_RESTRICTION` | Optional, define in `wp-config.php` to bypass the backend IP allowlist. |
| `SCRIPT_DEBUG` | Loads non-minified assets (`get_asset_suffix()`). |

## Options

| Option | Owner |
| :--- | :--- |
| `refinelab_tools_settings` | Admin: `enable_html_blocks`, `enable_svg_upload`, `enable_sticky_post_title`, `enable_resizable_sidebar` |
| `refinelab_security_settings` | Security — see [security.md](security.md) |
| `refinelab_mailer_settings`, `refinelab_forms_settings`, `refinelab_seo_settings`, `refinelab_multilang_settings`, `refinelab_multilang_strings` | Respective modules |
| `refinelab_custom_post_types`, `refinelab_custom_taxonomies` | Custom Types |
| `refinelab_migrated_to_refine_entry` | One-time migration flag (Forms) |

Transients: `refinelab_blocks_<md5(stylesheet)>` (block definitions), `refinelab_flush_rewrite_rules`, `refinelab_ll_<md5(ip)>` (failed-login counter).
User meta: `_refinelab_2fa_*` (Security two-factor).
Post meta: `_refinelab_lang`, `_refinelab_translation_group`, `_refinelab_seo_{title,desc,og_title,og_desc,og_image,og_image_id,noindex,nofollow,schema_type,schema_json}`, `_refinelab_zone`.

## Public PHP API

- `refinelab_tools()` returns the `RefineLab_Tools` singleton; reach modules through its magic getter: `refinelab_tools()->security`, `->seo`, `->multilang`. (`get_module()` exists only on `RefineLab_Plugin`, not on `RefineLab_Tools`.)
- Module settings: `RefineLab_<Module>::get_settings()` (static where noted in the class) and the `OPTION_KEY` constants.
- `RefineLab_Admin_UI::render()` / `::get()` for admin markup.

## Filters

| Filter | Args | Where |
| :--- | :--- | :--- |
| `refinelab_html_blocks_directories` | `string[] $dirs` | scan directories |
| `refinelab_html_blocks_category_slug` / `_category_title` | `$value, $theme` | default category |
| `refinelab_html_blocks_global_attributes` | `array $globals` | add / change template globals |
| `refinelab_html_blocks_breadcrumb_list` | `array $crumbs` | breadcrumb items |
| `refinelab_forms_recaptcha_min_score` | `float` (0.5) | reCAPTCHA v3 threshold |
| `refinelab_multilang_post_types` | `string[]` | post types with language UI |
| `refinelab_tools_sanitize_settings` | `array $sanitized, $input` | main option sanitizer |
| `refinelab_seo_post_types` | `string[]` | post types with SEO box and previews |
| `refinelab_seo_schema_graph` | `array $graph, array $ctx` | edit JSON-LD nodes |
| `refinelab_security_rest_allowed_namespaces` | `string[]` | guests’ REST namespaces |
| `refinelab_security_client_ip` | `string` | login limiter IP |

Actions: `refinelab_tools_render_content` (admin cards). The complete, verified list of hooks, functions and endpoints with examples is in [hooks-and-api.md](hooks-and-api.md).

## AJAX and admin-post endpoints

Full list of endpoints: [hooks-and-api.md §6](hooks-and-api.md). All privileged handlers verify a nonce and `current_user_can()`; public ones are only `refinelab_form_submit`.

## JavaScript globals in the block editor

Localized onto `refinelab-html-blocks-editor`:

| Global | Content |
| :--- | :--- |
| `refineLabThemeCategory` | `{ slug, title }` of the theme block category |
| `refineLabThemeBlocks` | detected block definitions |
| `refineLabGlobalAttributes` | template globals (see html-blocks.md §9) with the current language |
| `refineLabEditorSettings` | `{ stickyTitle, resizableSidebar, stickyTitleCssUrl }` |

Browser storage: `localStorage['refinelab_sidebar_width']`. CSS variable `--refinelab-sidebar-width`; body class `refinelab-resizable-sidebar` (admin) scopes the sidebar CSS. DOM event `refinelab:form:success` on the front end.

## CSS / naming

Conditional rows: `setting-row` args `depends` / `depends_on` (`data-refinelab-depends` in `core-admin.js`).

`.refinelab-*` only; no theme selectors in the plugin. Use `.editor-styles-wrapper`, `.block-editor-*`, `.components-*` for editor rules.
