# Getting started

**Language:** English · [繁體中文](zh-TW/getting-started.md)

Refine Lab is a modular WordPress toolkit: native Gutenberg blocks from plain HTML templates, forms, multilingual content, SEO, security, mail delivery, backups and everyday utilities. Switch on only what you need; every module can be turned off.

## Requirements

| | |
| :--- | :--- |
| WordPress | 7.0 or newer (single site; full backups are not available on multisite) |
| PHP | The `zip` extension is needed for full backups; `DOMDocument` for HTML blocks and SVG sanitising |
| Browser | A current Chrome, Firefox, Safari or Edge for the block editor features |

## Install and update

1. Download `refine-lab-wp-<version>.zip` from the [Releases](https://github.com/hoquangthaiholy/refine-lab-wp-releases/releases) page.
2. **Plugins → Add New → Upload Plugin**, choose the ZIP, activate.
3. Updates appear on **Plugins** and on **Refine Lab → Overview → Updates**, and can be installed automatically (see [site-tools.md](site-tools.md#updates-refine-lab--overview--updates)).

Deleting the plugin keeps your data; define `REFINELAB_WP_REMOVE_DATA` first if you want a clean removal.

## The admin map

| Menu | Pages (tabs) | What it is for |
| :--- | :--- | :--- |
| **Overview** | — | Status of every area, health check, updates, shortcuts. |
| **Blocks** | Blocks, Developer reference | HTML block engine: detected blocks, sample blocks, starter theme, cache. |
| **Custom Types** | — | Register post types and taxonomies without code. |
| **Content Blocks** | — | Shown when the theme opts in: editable content zones. |
| **Forms** | — (Entries is a separate list) | Form settings, statistics; submissions under **Entries**. |
| **Languages** | Settings, Translations | Multilingual setup and translation dashboard. |
| **SEO** | Search & social, Redirects, 404 log | Meta tags, social cards, structured data, sitemap, redirects. |
| **Performance** | Speed, Media | Front-end optimisations, WebP / AVIF. |
| **Security** | Hardening, File integrity, Activity log | Headers, login protection, 2FA, scans, audit trail. |
| **Mailer** | — | Send WordPress mail through SMTP or OAuth 2.0. |
| **Editor** | — | Sticky title and resizable sidebar in the block editor. |
| **Tools** | Backup & restore, Maintenance, Scripts, System status | Backups, maintenance mode, custom code, diagnostics. |

## Five-minute tour

### 1. Make your first block

Create `blocks/feature-card.html` in your theme:

```html
<div class="feature-card" data-title="Feature Card" data-icon="star-filled">
  <h3 data-bind="title">Lightning fast</h3>
  <p data-bind="text">One sentence about the feature.</p>
</div>
```

Open the block editor, search "Feature Card" in the inserter and edit the text directly on the canvas. No build step. More patterns: [html-blocks-cookbook.md](html-blocks-cookbook.md); every attribute: [html-blocks.md](html-blocks.md). No theme yet? **Blocks → Create starter theme / Install sample blocks** gives you a working base.

### 2. Add a form

```html
<form data-form="contact" data-redirect="/thank-you/">
  <input data-form-field="name" type="text" required />
  <input data-form-field="email" type="email" required />
  <textarea data-form-field="message"></textarea>
  <button type="submit">Send</button>
</form>
```

Submissions arrive under **Entries** and in your inbox. Details: [forms.md](forms.md).

### 3. Make the site multilingual

**Languages → Settings**: add languages, choose URL style, then translate content from the *Language* box in the editor and place `[refinelab_language_switcher]`. See [multilang.md](multilang.md).

### 4. Protect and back up

Turn on the protections you need in **Security** ([security.md](security.md)), then **Tools → Backup & restore → Create backup**. A full backup can be restored on another domain: [backup-restore.md](backup-restore.md).

### 5. Check health

**Overview → Health check** points out missing backups, SMTP, weak security options, pending updates and a maintenance mode that was left on.

## Conventions

| | |
| :--- | :--- |
| Classes | `RefineLab_*` |
| Options, hooks, AJAX actions | `refinelab_*` |
| CSS | `.refinelab-*` (the plugin never targets theme selectors) |
| Constants | `REFINELAB_WP_*` |
| Text domain | `refine-lab-wp` (English source; Traditional Chinese included) |

## Getting help

Bug reports and questions: open an issue on the [releases repository](https://github.com/hoquangthaiholy/refine-lab-wp-releases/issues) and attach the report from **Tools → System status → Copy support report**.
