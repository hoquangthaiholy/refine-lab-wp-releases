# Refine Lab: documentation

Reference for site builders, theme authors and plugin developers. Everything here is checked against the code of the release it ships with.

## Start here

| Document | For | Contents |
| :--- | :--- | :--- |
| [getting-started.md](getting-started.md) | Everyone | Requirements, install and update, the admin map, a five-minute tour. |

## Build with HTML blocks

| Document | Contents |
| :--- | :--- |
| [html-blocks.md](html-blocks.md) | The complete syntax: root attributes, `data-bind`, repeaters (nested too), trees, placeholders, conditions and loops, forms / breadcrumb / language components, inspector controls and pickers, global context objects, multi-file blocks, the JavaScript lifecycle API. |
| [html-blocks-cookbook.md](html-blocks-cookbook.md) | Copy-and-adapt templates: hero, team, pricing with nested repeaters, navigation tree, posts from the site, conditions, forms, multi-file block with PHP and JS; common mistakes. |

## Modules

| Document | Contents |
| :--- | :--- |
| [forms.md](forms.md) | Declarative forms: markup attributes, uploads, settings, JSON responses, webhook with signatures, rate limit, entries, CSV export. |
| [multilang.md](multilang.md) | Languages, URL modes, translating posts and terms, switcher (shortcode, block, PHP), fixed strings, translations dashboard. |
| [security.md](security.md) | Every hardening and login option, custom login URL, IP allowlist, two-factor, recovery. |
| [backup-restore.md](backup-restore.md) | Settings snapshots, full backups, restore on another domain with address replacement, settings files. |
| [site-tools.md](site-tools.md) | Performance, media (WebP/AVIF, SVG), redirects and 404 log, scripts, maintenance mode, file integrity, activity log, block history, editor options, updates, system status. |
| [modules.md](modules.md) | Per-module internals: settings keys, post meta, SEO, mailer, custom types, Content Blocks. |

## For plugin developers

| Document | Contents |
| :--- | :--- |
| [hooks-and-api.md](hooks-and-api.md) | Every filter and action with examples, PHP functions, shortcodes, theme support, JavaScript API and events, REST / AJAX / admin-post endpoints, constants, stored data, CSS hooks. |
| [developer-reference.md](developer-reference.md) | Constants, options, build and verification notes, adding a module. |

Conventions: classes `RefineLab_*`, options / hooks / actions `refinelab_*`, CSS `.refinelab-*`, constants `REFINELAB_WP_*`; text domain `refine-lab-wp`.
