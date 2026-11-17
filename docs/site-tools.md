# Site tools

**Language:** English · [繁體中文](zh-TW/site-tools.md)

Reference for the operational modules. Every option below is shown on its admin page; the keys are what you see in stored options, exports and WP-CLI. Security hardening has its own guide: [security.md](security.md). Backups: [backup-restore.md](backup-restore.md). Forms: [forms.md](forms.md). Languages: [multilang.md](multilang.md).

## Performance (Refine Lab → Performance → Speed)

Small, reversible front-end optimisations. Everything is **off by default**, and nothing runs in the admin, on the login screen, in feeds, in customizer previews, or when `SCRIPT_DEBUG` is on. Option `refinelab_performance_settings`.

| Key | Effect | Example |
| :--- | :--- | :--- |
| `defer_scripts` + `defer_handles` | Adds `defer` to the listed script handles (and to this plugin's own front-end scripts). `defer_handles` takes WordPress script handles separated by spaces, commas or new lines (up to 30). | `jquery-migrate`, `my-theme-slider` |
| `decoding_async` | Adds `decoding="async"` to content images. | — |
| `remove_emoji` | Removes the emoji detection script and styles. | — |
| `remove_embeds` | Removes the `wp-embed` script and oEmbed discovery. | — |
| `preload_fonts` | One URL per line; each becomes `<link rel="preload" href="…" as="font" type="font/woff2" crossorigin>` at the start of the head (type chosen from `woff2`, `woff`, `ttf`, `otf`). Use files of your own site. | `https://example.com/fonts/inter.woff2` |

Find script handles in the page source (`id="…-js"`, without the `-js`). Turn an option off to return to stock behaviour.

## Media (Refine Lab → Performance → Media)

Image tools, option `refinelab_media_settings`.

| Key | Default | Meaning |
| :--- | :--- | :--- |
| `webp_enabled`, `avif_enabled` | off | Create WebP / AVIF copies of JPEG and PNG uploads (only offered when the server supports the format). |
| `quality` | 82 | Compression quality of the copies. |
| `max_dimension` | 0 (off) | Largest width or height; bigger uploads are scaled down. |

Copies sit next to each file and size (`photo-300x200.jpg` → `photo-300x200.jpg.webp`) and are kept **only when smaller** than the original. The attachment meta `_refinelab_modern_images` lists which files have copies; the filter on `wp_content_img_tag` serves them through `<picture>` so browsers without support keep the original `<img>`. A batch tool converts the existing library (`wp_ajax_refinelab_modern_batch`).

**SVG upload** (**Enhancements / Media**, flag `enable_svg_upload`): allows `.svg` in the Media Library after a strict sanitiser: the file is parsed with `DOMDocument` (no DTD, no network), only allow-listed elements and attributes survive, scripts, event handlers, external references and unsafe CSS are removed, and the stored file is the cleaned markup. `.svgz` is refused.

## Redirects and 404 log (Refine Lab → SEO → Redirects, 404 log)

Rules are matched early on `template_redirect`. Tables `{prefix}refinelab_redirects` and `{prefix}refinelab_404` (created automatically; up to 1000 rules).

| Field | Values |
| :--- | :--- |
| `source` | Request path, e.g. `/old-page/`. |
| `target` | Destination URL or path. Not used for 410. |
| `type` | `301` permanent, `302` temporary, `410` gone. |
| `match` | `exact`; `wildcard` where `*` captures the rest (`/old/*` → `/new/$1`); `regex` (a PCRE pattern, back-references `$1`…). |

Saving a rule rejects loops and redirect chains. **404 log:** front-end 404 requests are counted (bots and probe paths are skipped, one cheap write per request, capped) and each row has a one-click *Create redirect*.

**CSV format** (import up to 500 rows / 500 KB, export of all rules): header `source,target,type,match`.

```csv
source,target,type,match
/old-shop/,/shop/,301,exact
/blog/2019/*,/archive/$1,301,wildcard
/promo,,410,exact
```

Invalid rows are skipped; cells starting with `=`, `+`, `-` or `@` are neutralised on export.

## Scripts (Refine Lab → Tools → Scripts)

Print your own code (analytics, pixels, inline CSS or JS) in five places. Option `refinelab_scripts_settings`; each location holds up to 100 KB.

| Location key | Printed |
| :--- | :--- |
| `front_head` | Front end, before `</head>` (`wp_head`). |
| `front_body` | Front end, right after `<body>` (`wp_body_open`; the theme must call it). |
| `front_footer` | Front end, before `</body>` (`wp_footer`). |
| `admin_head`, `admin_footer` | Admin header / footer. |

`skip_editors` stops the **front-end** code for logged-in users who can edit posts (keeps your own visits out of analytics). The code is printed **verbatim**, so include the `<script>` or `<style>` tags yourself, and only users with `unfiltered_html` may change it. The option is excluded from backups and exports on purpose. If a snippet breaks a page, add `define( 'REFINELAB_WP_DISABLE_SCRIPTS', true );` to `wp-config.php` and fix it in the admin.

```html
<!-- front_head -->
<script async src="https://www.googletagmanager.com/gtag/js?id=G-XXXX"></script>
<script>window.dataLayer=window.dataLayer||[];function gtag(){dataLayer.push(arguments)}gtag('js',new Date());gtag('config','G-XXXX');</script>
```

## Maintenance mode (Refine Lab → Tools → Maintenance)

Option `refinelab_maintenance_settings`.

| Key | Meaning |
| :--- | :--- |
| `enabled` | Turn the mode on. An admin-bar badge shows while it is on. |
| `page_id` | A published page to show (your theme's blocks and styles apply); empty uses `title` and `message`. |
| `title`, `message` | Built-in message. |
| `retry_after` | Minutes (5–1440, default 60) sent in the `Retry-After` header. |
| `token` | Secret for the preview link `https://example.com/?refinelab_preview=<token>`; visiting it sets a 12-hour cookie (`refinelab_maint_bypass`) that bypasses the mode. *Generate a new link* invalidates the old one. |

Visitors get **HTTP 503** with `Retry-After` and `noindex`, so search engines keep your rankings. Not affected: logged-in users who can edit pages, the login screen, admin, AJAX, WP-Cron and the REST API.

## File integrity (Refine Lab → Security → File integrity)

Reports changed, missing and new files; it **never** edits or deletes anything. Up to 4000 files per scope.

| Scope | Compared with |
| :--- | :--- |
| WordPress core | Official checksums from api.wordpress.org (what Site Health uses). |
| This plugin | `hashes.json` shipped in the plugin (generated when a release is built), else a baseline taken on this site. |
| Active theme | A baseline you take on this site (*Set baseline* when you are happy with the files). |

Settings (`refinelab_integrity_settings`): `weekly` (scheduled scan via cron hook `refinelab_integrity_scan`) and `email` (mail the administrator when something changed). The last report is kept in `refinelab_integrity_report`, the baseline in `refinelab_integrity_baseline`. Reports can be downloaded as CSV.

## Activity log (Refine Lab → Security → Activity log)

Who did what, in the table `{prefix}refinelab_activity`. Settings (`refinelab_activity_settings`): `enabled` (default on), `retention_days` (default 90, pruned daily by cron `refinelab_activity_prune`), `store_ip` (default off). Values of settings are never logged, only which option changed.

Events: login, failed login, two-factor locked, plugin activated / deactivated, theme switched, settings changed, content published, backup created / restored, full backup created / downloaded / uploaded / restored, settings exported / imported, integrity scan, integrity baseline set, maintenance mode changed, translations created. Filter by event, export CSV, clear with confirmation.

## Block history (editor)

Each time a post is saved, the attributes of every `theme/*` block are snapshotted in the post meta `_refinelab_block_history` (up to 20 snapshots / 400 KB per post). In the block sidebar, **Block history** lists earlier states of the selected block; *Restore* rolls it back, and the change is undone like any edit. See the REST route in [hooks-and-api.md](hooks-and-api.md).

## Editor enhancements (Refine Lab → Editor)

| Flag in `refinelab_tools_settings` | Default | Effect |
| :--- | :--- | :--- |
| `enable_sticky_post_title` | on | Keeps the post title pinned (blurred) while you scroll the editor. |
| `enable_resizable_sidebar` | on | Drag the left edge of the editor sidebar to resize it; double-click resets to 280 px; width kept in `localStorage['refinelab_sidebar_width']`. Requires the HTML Blocks module. |

## Updates (Refine Lab → Overview → Updates)

Releases are published in the public repository `hoquangthaiholy/refine-lab-wp-releases`. The plugin header declares that repository as its `Update URI`, so WordPress itself shows the update notice on **Plugins**, offers one-click update and the per-plugin *Enable auto-updates* switch. The Overview card adds *Check for updates* and *Install new versions automatically* (WordPress' own auto-update list).

- The plugin reads `update.json` from the repository's `main` branch (cached 12 hours; 1 hour after a failure) and shows *What is new* from `sections.changelog`.
- The ZIP must be an asset of that repository and its **SHA-256** must match the manifest; the download is verified before WordPress unpacks it, otherwise nothing is installed.
- No token is stored or sent. Override the address only for staging: constant `REFINELAB_WP_UPDATE_URL` or filter `refinelab_update_manifest_url`.

Manifest fields: `version`, `package`, `sha256`, `requires`, `requires_php`, `tested`, `released`, `homepage`, `sections.changelog`.

## System status (Refine Lab → Tools → System status)

Environment details for support (WordPress, theme, PHP, server, limits, extensions, which Refine Lab modules are on, counts of blocks and custom types). *Copy support report* puts a plain-text version on the clipboard. Attach it to bug reports.
