# Backup and restore

**Language:** English · [繁體中文](zh-TW/backup-restore.md)

One page, **Refine Lab → Tools → Backup & restore**, covers three needs: quick settings snapshots, whole-site backups that can be restored on another domain, and portable settings files.

| You want to… | Use | Contains | Where it is stored |
| :--- | :--- | :--- | :--- |
| Undo a settings change | **Settings only** backup (automatic or manual) | Refine Lab settings (no secrets) | Option `refinelab_settings_backups`, last 10 |
| Copy or move the whole site | **Full backup** | Database, uploads, plugins, themes | ZIP in `uploads/refinelab-backups/`, last 5 |
| Share a configuration | **Export a settings file** | Chosen settings and optionally block content | JSON file you download |

## 1. Settings backups

A snapshot is taken automatically before a module is switched on or off, after the plugin updates, and before every restore or import. **Create backup → Settings only** takes one on demand. Passwords and secret keys (anything whose key ends in `password`, `pass`, `secret`, `token`, `api_key`) are never included, and restoring never overwrites the secrets saved on the site. Raw custom code (the *Scripts* option) is excluded as well.

Restore from the list (**Restore**): choose the snapshot, optionally tick *Change the domain in the restored settings* (From = the address stored in the snapshot, To = this site), confirm. A fresh snapshot of the current state is taken first, so a restore can itself be undone.

## 2. Full backups

**Create a backup → Full backup**, choose what to include:

| Part | Content |
| :--- | :--- |
| Database | Every table with this site's prefix (posts, pages, comments, users, options, plugin tables). |
| Uploads | The media library folder. |
| Plugins | All plugin folders. |
| Themes | All theme folders. |

`.git`, `.svn`, `.hg`, `.idea`, `node_modules` and `.DS_Store` are left out. Add an optional note. A progress bar follows the job; the work runs in short steps, so large sites do not hit PHP time limits. Do not close the tab until it finishes.

**What the backup records** (the *Notes* of each row, also saved as `manifest.json` inside the ZIP): site address and site URL, WordPress / PHP / MySQL versions, table prefix, language, active theme, Refine Lab version, number of setting groups, counts of posts, pages, media, comments and users, database tables / rows / size, files and bytes per part, the full plugin and theme lists with versions and active state, and your note.

**Archive layout**

```text
manifest.json            site description (schema_version 1)
db.sql                   one statement per line: DROP TABLE, CREATE TABLE, INSERT
files/uploads/…          media library
files/plugins/…          plugins
files/themes/…           themes
```

**Security of the files:** the folder is protected by deny rules (`.htaccess`, `web.config`, empty `index.php`), file names contain a random part, and download goes only through an admin-post handler that checks the capability and a nonce. A full backup contains users and password hashes, so keep it private.

Requirements: the PHP `zip` extension and a single-site install (multisite is not supported). Delete a backup from the list; older ones are removed automatically beyond the limit of 5.

### Scheduled backups

**Backup → Scheduled backups** makes a full backup every day, week or month through WP-Cron (first run at 03:00 site time). Pick the parts to include and how many scheduled backups to keep (1–10; older scheduled ones are deleted, manual ones never are). A failed run e-mails the address you set (default: the admin e-mail) and is written to the activity log. Option `refinelab_backup_schedule`, cron hook `refinelab_scheduled_backup`. WP-Cron needs visits to fire; for exact timing call `wp-cron.php` from a server cron.

## 3. Restoring a full backup

1. **Restore** next to a backup in the list, or upload a `.zip` made on another site under **Restore from a file** (limited by PHP's upload size; the page shows your limit).
2. Choose **what to restore**: Database, Uploads, Plugins, Themes (only parts present in the backup are enabled; plugins and themes are disabled when file changes are blocked with `DISALLOW_FILE_MODS` or the account lacks `install_plugins` / `install_themes`).
3. **Domain.** Tick *Change the domain in the restored content* to rewrite the old address to a new one. From shows the address in the backup, To defaults to this site's address.
4. Keep *Make a backup of this site first* ticked (recommended), confirm the warning, press **Restore now**.

The restore replaces what you selected: database tables are dropped and recreated, plugin / theme / upload files are overwritten (files that are not in the backup are left alone).

### Domain replacement

Every text column of every table is scanned and the old address is replaced, including inside **PHP-serialized** values (string lengths are recalculated, nothing is unserialized, so no object can be created), JSON with escaped slashes (`https:\/\/old.example.com`), protocol-relative addresses (`//old.example.com`) and both `http` and `https`. Matching stops at a boundary, so `old.example.com` does not match `old.example.community`. The `guid` column of posts is intentionally untouched. `home` and `siteurl` are set to the new address.

Use the same tool for a path change too: a site moved from `https://a.com` to `https://b.com/blog` is rewritten as a whole, sub-folder included.

### What else the restore handles

- **Table prefix:** when the backup's prefix differs from this site's, tables are created with this site's prefix and the prefix-bound keys (roles in `options`, `capabilities` and other keys in `usermeta`) are renamed.
- **Plugin stays active:** Refine Lab is added back to the active plugins if the restored list lacks it, and this plugin's own folder is never overwritten.
- **Backup list:** the list of backups of *this* site is kept, not replaced by the source site's.
- **Transients and rewrite rules** are cleared.
- **Safety:** only `DROP TABLE`, `CREATE TABLE` and `INSERT INTO` statements on tables with the backup's prefix are executed, paths inside the ZIP are normalised and may not leave their target folder, script files (`.php`, `.phtml`, `.phar`, …) and `.ht*` files are not written to uploads.

### Staying logged in

The database is replaced, including the users and their login sessions, so a restore normally ends with a login screen. The restore runs through a **standalone runner** (`restore-runner.php`) that loads only WordPress core's minimal environment, so it keeps working while the options, plugins, users and sessions it relies on are being swapped. After a restore **on the same address**, your session is merged back when the same account (same ID and login) exists in the backup and its password is unchanged, so you stay logged in. After a **change of domain** the login cookie name changes, so you log in again with an account from the backup. The result message says which case applies.

If the runner file is blocked by the web server, the page falls back to `admin-ajax.php` automatically.

### Troubleshooting a restore

| Symptom | What to do |
| :--- | :--- |
| "This job expired or is not valid" | Jobs expire after 2 hours; start again. |
| Restore stops with a database error | The message names the failing statement; check MySQL privileges (`DROP`, `CREATE`) and `max_allowed_packet`. Re-run after fixing. |
| Site loads but looks unstyled / links point to the old domain | The domain option was not ticked. Restore the safety backup, or restore again with the domain option. |
| Locked out after restore | Log in with an account from the backup; use the password reset for your address. |

## 4. Settings files (export and import)

**Export a settings file** lists every settings group with a plain name and a description (General settings, Forms, Email delivery (SMTP), Languages, SEO and social sharing, Performance, Security hardening, …) and an optional **Block content** item that saves the attributes of every HTML block in pages and posts. Download the JSON.

```json
{
  "schema_version": 1,
  "plugin_version": "1.1.0",
  "exported_at": "2026-11-17T07:00:00+00:00",
  "site": { "home": "https://old.example.com" },
  "options": { "refinelab_seo_settings": { "...": "..." } },
  "blocks": [ { "post_type": "page", "slug": "home", "title": "Home", "blocks": [ { "path": "0", "name": "theme/hero", "attrs": { } } ] } ]
}
```

**Restore from a file** accepts that JSON (and `.zip` for full backups) in one upload box. A settings file is **reviewed before anything changes**: every group is listed as *New*, *Will change* or *Identical* and ticked accordingly, block content is matched to posts by post type and slug (posts without a match are skipped), and you can tick *Change the domain in the imported settings and block content* (From is read from the file). A settings snapshot is taken first. Files up to 2 MB are accepted; a file made by a newer plugin version is refused.

## 5. Filters and endpoints

| Item | Purpose |
| :--- | :--- |
| `refinelab_backup_options` (filter, `string[]`) | Add or remove option names that belong in snapshots and exports. Names must start with `refinelab_`. |
| `admin_post_refinelab_backup_create|restore|delete|download` | Settings snapshots. |
| `admin_post_refinelab_export`, `refinelab_import_upload`, `refinelab_import_apply` | Settings file export / import. |
| `wp_ajax_refinelab_site_backup_step` (+ `nopriv`) and `restore-runner.php` | Stepped full backup / restore jobs, authorised by a per-job secret. |
| `admin_post_refinelab_site_backup_download|delete|upload` | Full backup files. |

Activity-log events: *Backup created*, *Backup restored*, *Settings exported*, *Settings imported*, *Full backup created / downloaded / uploaded / restored*.

Uninstalling keeps backups unless `REFINELAB_WP_REMOVE_DATA` is set (then the backups folder and options are deleted).
