# Modules reference

All modules extend `RefineLab_Abstract_Module` (`get_slug()`, `get_title()`, settings helpers, `render_admin_tab()` → `views/tab.php`). They are created by `RefineLab_Plugin` (`load_core_modules()`, lazy `get_module()`), and are reachable through `refinelab_tools()-><slug>`. Module on/off flags live in the option `refinelab_tools_settings` as `enable_<slug>`; module settings in `refinelab_<slug>_settings`.

Admin pages (Refine Lab menu): HTML Blocks, Enhancements, Media Tools, Mailer, Forms, Languages, SEO, Security, Custom Types, System Status, Library.

---

## admin
`RefineLab_Admin_Settings` (`includes/modules/admin/`), `RefineLab_Extended_Settings`, `RefineLab_Admin_UI`.
- Registers the menu (`refine-lab-wp`), tab routing and the Settings API groups `refinelab_tools_settings_group` and `refinelab_<module>_settings_group`.
- Sanitizers: `refinelab_sanitize_{mailer,forms,multilang,seo,security}_settings()` (in `class-extended-settings.php`); the main option passes through the filter `refinelab_tools_sanitize_settings`.
- UI components: `RefineLab_Admin_UI::render( $name, $args )` / `::get()` render the partials in `templates/admin/components/` (`badge`, `button`, `card`, `choice-list`, `code-block`, `data-table`, `empty-state`, `modal`, `notice`, `option-group`, `page-header`, `save-bar`, `select`, `setting-row`, `tab-panel`, `tabs`, `text-field`, `textarea`, `toggle`). Arguments are documented at the top of each partial.
- Main option `refinelab_tools_settings` keys: `enable_html_blocks`, `enable_svg_upload`, `enable_sticky_post_title`, `enable_resizable_sidebar` (all default on).

## html-blocks
`RefineLab_HTML_Blocks` + traits `Parser`, `Renderer`, `Admin`. See [html-blocks.md](html-blocks.md).
- Registers blocks on `init`, category via `block_categories_all`, editor assets on `enqueue_block_editor_assets`, clears the cache on `save_post` and `after_switch_theme`.
- Admin actions: `refinelab_create_sample_theme`, `refinelab_install_sample_blocks`, `refinelab_clear_blocks_cache`.
- Methods worth knowing: `get_detected_blocks()`, `get_scan_directories()`, `get_global_block_attributes()`, `is_enabled()`, `is_sticky_title_enabled()`, `is_resizable_sidebar_enabled()`.
- The editor script is a generated bundle shipped in the plugin; its sources are maintained in the source repository.

## multilang
`RefineLab_MultiLang` + traits `Routing`, `Query`, `Admin`, `Taxonomies`, `Strings`, `Switcher`. Options: `refinelab_multilang_settings`, `refinelab_multilang_strings`.
- Settings: `enabled`, `languages` (code, name, flag, locale), `default_lang`, `url_mode` (`prefix` → `/en/about`, `query` → `?lang=en`), `url_format`, `hide_default_prefix`, `browser_redirect`, `missing_translation_behavior` (`home` | `404`).
- Post meta: `_refinelab_lang` (language), `_refinelab_translation_group` (links translations). Rewrite tag `%refinelab_lang%`.
- Filters on WordPress: `locale`, `language_attributes`, `page_row_actions`; adds language metabox, list-table flag columns and filter bar (post types via `refinelab_multilang_post_types`).
- Shortcode `[refinelab_language_switcher format="both|flags|names" layout="list|dropdown"]` (alias `[refinelab_lang_switcher]`); declarative `data-lang-switcher` in block templates.
- Admin actions: `refinelab_assign_language`, `refinelab_duplicate_translation`, `refinelab_unlink_translation`; AJAX `refinelab_delete_language_data`, `refinelab_language_data_summary`, `refinelab_unlink_translation_group`, `refinelab_link_translation_group`.
- Per-language header/footer template parts in the Site Editor.

## forms
`RefineLab_Forms` + traits `Ajax`, `Export`. Option `refinelab_forms_settings`.
- Settings: `enabled`, `enable_honeypot`, `send_email`, `notification_email` (default admin e-mail), `email_subject`, `success_msg`, `error_msg`, `recaptcha_site_key`, `recaptcha_secret`, `recaptcha_version` (`v2` | `v3`; minimum v3 score filter `refinelab_forms_recaptcha_min_score`, default 0.5).
- Submissions are stored in the CPT `refine_entry` (legacy `refinelab_submission` is migrated once); CSV export through `admin_post_refinelab_export_submissions_csv`.
- Front end: `assets/js/syntax-forms.js` / `.css`, enqueued only when a form is rendered. Honeypot field `_refinelab_hp`; success DOM event `refinelab:form:success`.
- AJAX `refinelab_form_submit` (public + logged-in).
- Markup: `data-form`, `data-form-label`, `data-redirect`, `data-form-field`, `data-captcha`. Full guide with webhook, attachments and rate limits: [forms.md](forms.md).

## mailer
`RefineLab_Mailer` (lazy-loaded on `phpmailer_init`). Option `refinelab_mailer_settings`: `enabled`, `smtp_host`, `smtp_port`, `smtp_secure`, `smtp_auth`, `smtp_user`, `smtp_pass`, `from_email`, `from_name`, `force_from`, plus OAuth 2.0 (Gmail / Microsoft 365): `auth_method`, `oauth_client_id`, `oauth_client_secret`, `oauth_tenant`, `oauth_refresh_token`, `oauth_email` (XOAUTH2 via PHPMailer; code in `traits/trait-mailer-oauth.php`).
- Hooks: `phpmailer_init`, `wp_mail_from`, `wp_mail_from_name`. AJAX `refinelab_send_test_email` sends a test message from the admin page.

## seo
`RefineLab_SEO` + trait `RefineLab_SEO_Schema` (`includes/modules/seo/traits/trait-seo-schema.php`). Admin page **Social & SEO**. Option `refinelab_seo_settings`:

| Group | Keys |
| :--- | :--- |
| Search | `enabled`, `home_title`, `home_desc`, `title_separator`, `enable_sitemap` |
| Open Graph / Twitter | `og_enabled`, `default_og_img`, `og_use_featured`, `twitter_card` (`summary_large_image` \| `summary`), `twitter_site` (`@handle`), `facebook_app_id` |
| JSON-LD | `schema_enabled`, `schema_type` (Organization, Corporation, LocalBusiness, NGO, EducationalOrganization, Person), `org_name`, `org_logo`, `social_profiles` (URLs, one per line → `sameAs`), `schema_article_type` (Article, BlogPosting, NewsArticle), `schema_breadcrumbs`, `schema_website_search` |

- **Page context**: `build_context_for_request()` / `build_context_for_post( $post )` return one array (kind `home|singular|term|archive|search|none`, title, description, image with size/alt, canonical, og type, robots, per-post schema override) used by the meta output, the previews and the JSON-LD generator. A static front page is treated as `home`.
- **Open Graph output** (`wp_head`, only when `og_enabled`): `og:locale/site_name/type/title/url/description/image(+width,height,alt)`, `article:published_time/modified_time` (posts), `fb:app_id`, `twitter:card/site/title/description/image(+alt)`. Image order: post OG image → featured image (`og_use_featured`) → default image. Image dimensions are looked up once per URL (transient `refinelab_og_<md5>`).
- **Editor box** (all public post types; filter `refinelab_seo_post_types`): live **Google**, **Facebook / LinkedIn** and **X** previews (block and classic editor; updates from title, excerpt, featured image and fields), counters, per-post SEO title/description, social title/description/image (Media Library), noindex, structured-data type and (for users with `unfiltered_html`) extra JSON-LD, plus a read-only view of the generated JSON-LD.
- **Auto JSON-LD**: one `@graph` per page — Organization/Person (logo, sameAs), WebSite (optional SearchAction), WebPage / CollectionPage / SearchResultsPage, Article-type node for posts (author name only, no author URL, to avoid exposing usernames), ImageObject, BreadcrumbList (pages → ancestors, posts → category chain, terms → ancestors) and per-post custom JSON merged in. All nodes link by `@id`. Empty values are pruned; output is encoded with `JSON_HEX_TAG|JSON_HEX_AMP`. Filter `refinelab_seo_schema_graph( $graph, $ctx )` edits nodes (keys `organization`, `website`, `webpage`, `image`, `article`, `breadcrumb`, `custom_*`).
- Per-post meta (`_refinelab_seo_*`): `title`, `desc`, `og_title`, `og_desc`, `og_image`, `og_image_id`, `noindex`, `nofollow`, `schema_type`, `schema_json`.
- Sitemap: rewrite `^sitemap\.xml$` → `?refinelab_sitemap=1`, rendered on `template_redirect`.
- Assets: `assets/js/admin/metabox-seo.js` + `css/admin/metabox-seo.css` (editor), `tab-seo.js` (settings page).

## security
See [security.md](security.md): hardening headers, XML-RPC, user enumeration, login limiter, custom login URL, hiding WordPress.

## svg-upload
`RefineLab_SVG_Upload`. Flag `enable_svg_upload`. Filters `upload_mimes`, `wp_check_filetype_and_ext`, `wp_handle_upload_prefilter` (`RefineLab_SVG_Upload::sanitize_svg()`: `DOMDocument` parse with `LIBXML_NONET`; rejects DOCTYPE/entities and non-`<svg>` roots; element allowlist (`allowed_elements()`); removes `on*`, external `href`/`url()`, `javascript:`/`data:` values and unsafe CSS; `<use>` and `href` only to `#id`, `<image>` only raster data URIs; the stored file is the sanitized markup; `.svgz` is not allowed), `wp_prepare_attachment_for_js` (dimensions).

## custom-types
`RefineLab_Custom_Types` + traits `Registrar`, `Ajax`. Options `refinelab_custom_post_types`, `refinelab_custom_taxonomies`; transient `refinelab_flush_rewrite_rules` triggers a rewrite flush.
- Declarative CPT and taxonomy registration on `init`. AJAX `refinelab_save_cpt`, `refinelab_delete_cpt`, `refinelab_save_taxonomy`, `refinelab_delete_taxonomy` (admin only, nonce + capability).

## content
`RefineLab_Content`. Opt-in per theme:

```php
add_theme_support( 'refinelab-content', array(
	'taxonomies' => array( 'category' ),
	'zones'      => array(
		'hero' => array( 'label' => 'Hero', 'default' => 'theme/hero', 'blocks' => array() ),
	),
) );
```

- Hidden CPT `refinelab_content`; zone stored in `_refinelab_zone`; language in `_refinelab_lang` (shared with MultiLang, which gets the CPT through `refinelab_multilang_post_types`).
- Block `refinelab/content` previews the content and edits it in a modal hosting the real block editor (assets `content-block.js`, `content-modal.js`, `content.css`). Modes: `term` (zone of the current term, parent inheritance, single posts use their terms) and `entry` (standalone reusable item).
- Preview endpoint `?refinelab_content_preview=<id>` (nonce). Shortcode `[refinelab_content id="" zone="" term=""]`.
- Admin-post actions `refinelab_create_content`, `refinelab_delete_content`.
