# Security module reference

**Language:** English · [繁體中文](zh-TW/security.md)

Class `RefineLab_Security` · option `refinelab_security_settings` · admin page **Refine Lab → Security** · file `includes/modules/security/class-security.php`.

## Admin page layout

Cards, in order: master switch → **Sign-in protection** (custom login URL, IP allowlist, attempt limiter, two-factor, generic errors) → **Access hardening** → **HTTP headers** → **Hide WordPress** → **Server-level hiding**. Everything below the master switch is hidden while it is off. Dependent fields appear only while their parent switch is on: login address / blocked-request behaviour (custom login URL), allowed IPs (IP restriction), attempts / lockout minutes (attempt limiter). Hidden fields keep their saved values. The mechanism is generic: pass `depends` (checkbox id) and `depends_on` (current state) to the `setting-row` component; `core-admin.js` toggles `hidden` on change.

Every option is stored as `1`/`0` (numbers and strings noted). The master switch `enabled` turns the whole module off. Defaults apply to missing keys, so existing sites keep their behaviour after upgrades; new options are off by default unless noted.

## Hardening

| Key | Default | Behaviour |
| :--- | :--- | :--- |
| `disable_xmlrpc` | on | `xmlrpc_enabled` false, `X-Pingback` header removed, requests with `XMLRPC_REQUEST` get 403. |
| `block_user_enumeration` | on | `/?author=N` redirects home, REST `/wp/v2/users` routes hidden from guests, `users` provider removed from the core XML sitemap. |
| `security_headers` | on | `X-Frame-Options: SAMEORIGIN`, `X-Content-Type-Options: nosniff`, `Referrer-Policy: strict-origin-when-cross-origin`, `X-XSS-Protection`. |
| `permissions_policy` | off | `Permissions-Policy: camera=(), microphone=(), geolocation=(), payment=(), usb=()`. |
| `hsts` | off | `Strict-Transport-Security: max-age=31536000; includeSubDomains`, only sent over HTTPS. |
| `restrict_rest_api` | off | Guests get HTTP 401 on every REST route except namespaces from the filter `refinelab_security_rest_allowed_namespaces` (default `refinelab`, `oembed`). May break public form / headless plugins. |
| `disable_application_passwords` | off | `wp_is_application_passwords_available` false. |
| `disallow_file_edit` | on | Defines `DISALLOW_FILE_EDIT` (only if not already defined). |
| `block_uploads_php` | off | Writes a marked block to `uploads/.htaccess` that denies `.php`, `.phtml`, `.phar` and similar files, and removes it when switched off. Apache/LiteSpeed only; on nginx add the rule from "Server-level hiding". **Test now** uploads a throw-away probe file, requests it over HTTP and reports whether PHP would run (the probe is always deleted). |
| `disallow_file_mods` | off | `file_mod_allowed` false: no installs, updates or deletions of plugins and themes from the admin (like `DISALLOW_FILE_MODS`). Updates, including this plugin's, will not run until it is turned off. |

## Security alerts

An e-mail (through `wp_mail()`, so the Mailer applies) to `alert_email` (default: the site admin e-mail). Alerts of one kind are limited to one every 5 minutes.

| Key | Sent when |
| :--- | :--- |
| `alert_admin_login` | An administrator signs in from an address not seen before. Addresses are stored only as salted hashes in the user meta `_refinelab_known_ips` (last 10); the first sign-in after turning this on is only recorded. |
| `alert_lockout` | The login limiter locks an address out (needs `limit_login_attempts`). |
| `alert_admin_change` | A user is created as, or given the role of, administrator. |
| `alert_plugin_change` | A plugin is activated, deactivated or deleted, or the theme is switched. |

## Passwords and sessions

| Key | Values | Behaviour |
| :--- | :--- | :--- |
| `password_min_length` | `0` (WordPress default), 8, 10, 12, 14, 16 | Minimum length of a new password. |
| `password_require_mix` | on / off | At least three of lowercase, capitals, digits, symbols. |
| `password_hibp` | on / off | Refuses passwords found in the Pwned Passwords list. Only the first 5 characters of the SHA-1 hash are sent (k-anonymity), never the password; if the service cannot be reached the password is accepted. |
| `idle_logout_minutes` | `0` (never), 15, 30, 60, 120, 240, 480 | Signs out users who can edit content after this long without a page request (Heartbeat does not count). Tracked in user meta `_refinelab_last_seen`, written at most once a minute. |
| `max_sessions` | `0` (no limit), 1, 2, 3, 5 | At sign-in the oldest sessions beyond the limit are ended. |

The password rules apply when a password is set in a profile, on the new-user screen and on password reset; existing passwords are not touched. **Sign me out of all my other sessions** is on the same card.

## Content Security Policy

`csp_mode`: `off`, `report_only` (header `Content-Security-Policy-Report-Only`, nothing is blocked) or `enforce`. `csp_preset`: `relaxed` (inline scripts and HTTPS sources allowed), `strict` (same-site only, no inline scripts) or `custom` (`csp_custom`, plain directives separated by semicolons). Sent on front-end requests only. Browsers report violations to `POST /wp-json/refinelab/v1/csp-report` (public by nature: throttled to 120 reports an hour, query strings dropped, at most 50 distinct directive/address pairs kept in `refinelab_csp_reports`); the list is on the Security page. Start with report only, fix the policy until the list stays empty, then enforce.

## Login protection

| Key | Default | Behaviour |
| :--- | :--- | :--- |
| `obscure_login_errors` | on | Replaces login errors with “Invalid login credentials.” |
| `two_factor_enabled` | off | Two-factor authentication (TOTP), see below. |
| `ip_restrict_enabled`, `ip_allowlist` | off, `''` | Backend IP allowlist, see below. |
| `limit_login_attempts` | off | Counts failures per client IP in the transient `refinelab_ll_<md5(ip)>`; after `login_max_attempts` (3–20, default 5) the client is rejected for `login_lockout_minutes` (1–1440, default 15). A success clears the counter. IP = `REMOTE_ADDR`; filter `refinelab_security_client_ip` to read a trusted proxy header. |
| `custom_login_enabled`, `login_slug`, `login_blocked_action` | off, `''`, `404` | See below. |

### Restrict backend access by IP

`ip_allowlist` holds IPv4/IPv6 addresses and CIDR ranges (one per line; invalid lines are dropped with a warning).

- Clients outside the list get **403** on the login form (`/wp-login.php` or the custom slug) and on `/wp-admin/*`, and cannot authenticate by password anywhere (login form, XML-RPC, REST basic auth).
- Not restricted: public pages, `admin-ajax.php`, `admin-post.php`, WP-Cron/WP-CLI, `wp-login.php?action=postpass`.
- Already-signed-in cookies are blocked from `/wp-admin` too, because the check is per request.
- Safety: saving is refused (feature left off) when the list is empty or does not contain your current IP. Client IP is `REMOTE_ADDR` (filter `refinelab_security_client_ip` behind a proxy/CDN — without it every visitor may look like the proxy).
- **Recovery**: define `REFINELAB_DISABLE_IP_RESTRICTION` as `true` in `wp-config.php`.

### Two-factor authentication (TOTP)

Class `RefineLab_Two_Factor` (`includes/modules/security/class-two-factor.php`), loaded when `two_factor_enabled` is on. RFC 6238: SHA-1, 6 digits, 30 s steps, ±1 step of clock drift.

- Each user opts in under **Users → Profile → Two-factor authentication**: enter the shown key in an authenticator app (manual entry, or the `otpauth://` link on mobile), confirm with a code, and save 10 one-time **recovery codes** (shown once). There is no QR code, because rendering one would need a library or an external service that would see the secret.
- Users who did not enable it log in as before. The login form gets an extra “Authentication code” field; the code is checked after the password. A used code cannot be replayed (last accepted time step is stored).
- Disabling or regenerating codes requires a valid current or recovery code. An administrator with `edit_users` can reset another user from that user’s profile (checkbox “Turn off two-factor…”).
- Failed codes fire `wp_login_failed`, so the login limiter counts them. In addition, `verify_login()` throttles guessing per user: after `MAX_FAILS` (5) invalid codes the user is locked for `LOCK_SECONDS` (15 min) via transient `refinelab_2fa_fail_<id>`; the lock is logged as `twofactor_locked` in the activity log. A code step cannot be reused (`_refinelab_2fa_last_step`).
- Storage: user meta `_refinelab_2fa_{enabled,secret,pending,recovery,last_step}`. The secret is encrypted with libsodium using a key derived from the `AUTH` salt; **changing the salts invalidates enrolled secrets** (users must use recovery codes or be reset). Recovery codes are stored as HMAC hashes.
- The prompt is a field on the login form, not a second screen, so a wrong or missing code reveals that the password was correct. Application passwords and REST cookie auth do not go through the login form and are not affected (disable application passwords separately if needed).
- **Recovery**: define `REFINELAB_DISABLE_2FA` as `true` in `wp-config.php` to skip all 2FA checks and hide the UI.

### Requiring two-factor by role

`twofactor_required_roles` (roles), `twofactor_grace_days` (0, 3, 7, 14, 30) and `twofactor_required_since` (set when roles are first chosen). Users with those roles who have not set up two-factor see a reminder until the grace period ends; after that every admin screen except their profile, `admin-post.php` and `admin-ajax.php` redirects to the profile until it is set up. Needs `two_factor_enabled`. `REFINELAB_DISABLE_2FA` in wp-config.php switches enforcement off if everyone is ever locked out. E-mail one-time codes are not available.

### Custom login URL

When enabled the login form is served at `https://site/<slug>/`:

- The request is intercepted on `wp_loaded` (priority 0): the path (relative to the home URL) equal to the slug loads `wp-login.php`.
- Every generated login URL (`site_url`, `network_site_url`, `wp_redirect`: login, logout, lost-password, reset e-mails, `wp_login_url()`) is rewritten from `wp-login.php` to the slug.
- Signed-out requests to `wp-login.php` and to `/wp-admin/*` (except `admin-ajax.php` and `admin-post.php`) get a **404 page** or a **redirect to the home page** (`login_blocked_action`: `404` | `home`). `wp-login.php?action=postpass` stays reachable for cached password-form pages.
- Validation on save: ≥ 3 characters; not reserved (`wp-admin`, `wp-login`, `wp-content`, `wp-includes`, `wp-json`, `wp-cron`, `admin`, `dashboard`, `login`, `feed`, `sitemap`); not an existing public page/post slug. On failure the feature is left off and an error notice explains why.
- Not supported on multisite. Not active for WP-CLI and cron.
- **Recovery**: add `define( 'REFINELAB_DISABLE_CUSTOM_LOGIN', true );` to `wp-config.php` to restore `/wp-login.php`, or rename the plugin folder over SFTP.
- Bookmark the new address before saving; plugins that hard-code `/wp-login.php` links in static markup will show a 404 to signed-out visitors.

## Hide WordPress

These remove common fingerprints. They reduce what automated scanners can detect; they cannot hide WordPress completely (paths such as `/wp-content/`, block markup and REST routes still exist).

| Key | Default | Removes / changes |
| :--- | :--- | :--- |
| `hide_wp_version` | on | `<meta name="generator">` (also feeds), `?ver=` on scripts/styles. |
| `clean_head` | on | RSD, Windows Live Writer, shortlink and adjacent-post `<link>`s. |
| `hide_rest_api_links` | off | `api.w.org` link tags and `Link` header, REST RSD entry, oEmbed discovery links and host JS. The REST API itself stays enabled. |
| `disable_emojis` | off | Emoji detection script and styles, TinyMCE emoji plugin, `s.w.org` DNS prefetch, emoji in feeds/e-mail. |
| `hide_server_headers` | off | `X-Powered-By`, `X-Pingback`, `X-Redirect-By: WordPress`. Set `expose_php = Off` in `php.ini` for a complete result. |
| `hide_admin_branding` | off | Admin footer credit and version, toolbar WordPress-logo menu, dashboard “WordPress Events and News” widget, login logo link/text point to the site. |

### Server-level hiding (cannot be done from PHP)

Static files are answered by the web server before PHP runs. The Security page prints these snippets:

```nginx
location ~* ^/(readme\.html|license\.txt|wp-config-sample\.php|wp-links-opml\.php)$ { return 404; }
location ~* ^/wp-content/(uploads|upgrade)/.*\.php$ { return 403; }
server_tokens off;
```

```apache
<FilesMatch "^(readme\.html|license\.txt|wp-config-sample\.php|wp-links-opml\.php)$">
	Require all denied
</FilesMatch>
ServerTokens Prod
Header unset X-Powered-By
```

## Filters

| Filter | Args | Purpose |
| :--- | :--- | :--- |
| `refinelab_security_rest_allowed_namespaces` | `string[]` | Namespaces reachable by guests when the REST restriction is on. |
| `refinelab_security_client_ip` | `string $ip` | IP used by the login limiter. |

## Programmatic API

- `RefineLab_Security::get_settings()` / `get_defaults()` / `get_features()` (switch metadata with `group`: `hardening`, `login`, `hide_wp`).
- `RefineLab_Security::get_login_slug()` returns the active slug or `''`.
- `RefineLab_Security::sanitize_input( $input )` is the single sanitizer used by both Settings API registrations.
