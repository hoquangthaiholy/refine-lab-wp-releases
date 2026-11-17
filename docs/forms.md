# Forms

**Language:** English · [繁體中文](zh-TW/forms.md)

Zero-setup form handling: write a normal HTML `<form>`, add `data-form`, and Refine Lab validates it, stores the submission as an **Entry**, e-mails a notification and can forward it to a webhook. No shortcode and no form builder are involved.

Admin: **Refine Lab → Forms** (settings and statistics) and **Entries** (the list of submissions). Module class `RefineLab_Forms`, option `refinelab_forms_settings`, entries are the custom post type `refine_entry`.

## 1. Markup

```html
<form data-form="contact" data-form-label="Contact form" data-redirect="/thank-you/">
  <input data-form-field="name" type="text" placeholder="Full name" required />
  <input data-form-field="email" type="email" placeholder="Email" required />
  <textarea data-form-field="message" placeholder="Message"></textarea>
  <button type="submit">Send</button>
</form>
```

| Attribute | On | Meaning |
| :--- | :--- | :--- |
| `data-form="id"` | `<form>` | Required. The form identifier, used to filter entries in the admin and in the e-mail subject. Forms are picked up automatically on page load. |
| `data-form-label="text"` | `<form>` | Human-readable name shown in entries and e-mails (falls back to the id). `data-label` is accepted as a fallback. |
| `data-redirect="url"` | `<form>` | Page to open after a successful submission. Without it the success message is shown. |
| `data-form-field="key"` | `input`, `textarea`, `select` | Names the field in the entry. If missing, the field's `name`, then `id`, is used. |
| `data-captcha="v2"` | any element inside the form | Where the reCAPTCHA v2 checkbox is drawn. If you leave it out, one is created before the submit button when v2 is configured. |

Fields named `g-recaptcha-response`, `_refinelab_hp` and submit buttons are never stored. Checkbox groups and multi-selects are stored as lists and joined with commas in e-mails. The sender (`Reply-To`) of the notification is taken from a field keyed `email`, `user_email`, `contact_email` or `mail` (it must be a valid address) and the sender name from `name`, `full_name`, `username` or `contact_name`.

### File upload

```html
<form data-form="job-application" data-form-label="Job application">
  <input data-form-field="applicant_name" type="text" required />
  <input data-form-field="resume" type="file" accept=".pdf,.doc,.docx" />
  <button type="submit">Apply</button>
</form>
```

Uploads need **Forms → Attachments** switched on. Each file is checked against the allowed extensions and real MIME type and the size limit, stored under random names in `uploads/refinelab-forms/` (not web accessible), and attached to the entry. Editors download them only through the capability-checked link on the entry screen.

## 2. Settings (Refine Lab → Forms)

| Key | Default | Meaning |
| :--- | :--- | :--- |
| `enabled` | on | Master switch for the front-end script. |
| `send_email` | on | Send a notification for every submission. Entries are always saved; this only controls the e-mail. |
| `notification_email` | site admin e-mail | Recipient. |
| `email_subject` | `New Contact Form Submission: {form_id}` | Placeholders: `{form_id}`, `{form_label}` (both resolve to the label when present) and `{site_name}`. |
| `success_msg` / `error_msg` | English texts | Messages returned to the visitor. |
| `enable_honeypot` | on | Adds a hidden field `_refinelab_hp`; a submission that fills it is silently discarded (the bot still sees a success message). |
| `recaptcha_version`, `recaptcha_site_key`, `recaptcha_secret` | `v2`, empty | Google reCAPTCHA. v2 shows the checkbox; v3 verifies invisibly in the background for every form. |
| `rate_limit_enabled`, `rate_limit_max`, `rate_limit_window` | on, 5, 10 | At most `max` submissions per form and visitor IP in `window` **minutes**. |
| `webhook_url`, `webhook_secret` | empty | Forward each entry to your service (see below). |
| `attachments_enabled`, `attachments_max_mb`, `attachments_types` | off, 5, `pdf,jpg,jpeg,png,webp,doc,docx,txt` | File upload policy. The allowed extensions are limited to `pdf, jpg, jpeg, png, gif, webp, doc, docx, xls, xlsx, ppt, pptx, txt, csv`. |

The notification contains the form name and id, date, IP, every field and a link to the entry. A page-level rate limit can be adjusted per form with the filter `refinelab_forms_rate_limit` (see below). The statistics card shows submissions per day and per form for the last 30 days.

## 3. What the browser does

1. `assets/js/syntax-forms.js` finds every `form[data-form]`, adds the honeypot, renders reCAPTCHA if configured.
2. On submit it sends the fields as `FormData` to `admin-ajax.php` (`action=refinelab_form_submit`) with a nonce.
3. The server answers JSON: `{ "success": true, "data": { "message": "…", "redirect": "…" } }` or `{ "success": false, "data": { "message": "…" } }` (HTTP 403 when the security nonce is invalid, 429 when rate limited, 400 for reCAPTCHA and upload errors, 500 when the entry cannot be saved; a filled honeypot gets a fake success so bots do not retry).
4. The script shows the message (or follows `data-redirect`) and fires a DOM event.

```js
document.addEventListener('refinelab:form:success', function (e) {
  // e.detail = { formId: 'contact', response: { ...server data... } }
  if (e.detail.formId === 'contact') {
    window.dataLayer && window.dataLayer.push({ event: 'lead', form: e.detail.formId });
  }
});
```

The script and its stylesheet load only on pages that contain a form. The runtime configuration is available as `window.refinelabFormsConfig` (`ajaxUrl`, `nonce`, `honeypot`, `recaptchaKey`, `recaptchaVersion`, `i18n`).

## 4. Webhook

When `webhook_url` is set, every saved entry is queued and POSTed as JSON a moment later (visitors never wait for your server), through WP-Cron:

```json
{
  "event": "form.submitted",
  "entry_id": 123,
  "form_id": "contact",
  "form_label": "Contact form",
  "submitted_at": "2026-11-17T07:00:00+00:00",
  "site": "https://example.com",
  "language": "zh-tw",
  "fields": { "name": "Ada", "email": "ada@example.com", "message": "Hi" },
  "attachments": [ { "name": "cv.pdf", "size": 52311, "type": "application/pdf" } ]
}
```

Headers: `X-Refinelab-Event: form.submitted`, `X-Refinelab-Attempt: 1|2|3` and, when a secret is set, `X-Refinelab-Signature: sha256=<hmac>` where the HMAC-SHA256 key is your secret and the message is the raw request body. Verify it before trusting a request:

```php
$body = file_get_contents( 'php://input' );
$sent = $_SERVER['HTTP_X_REFINELAB_SIGNATURE'] ?? '';
$calc = 'sha256=' . hash_hmac( 'sha256', $body, 'your-secret' );
if ( ! hash_equals( $calc, $sent ) ) { http_response_code( 401 ); exit; }
```

A response outside 2xx counts as a failure. Delivery is retried twice (after 5 and 30 minutes); the status of each entry is shown on its screen and the last failures are kept in the option `refinelab_forms_webhook_log`. Redirects are not followed and private or unsafe addresses are rejected.

## 5. Hooks

| Hook | Type | Arguments | Use |
| :--- | :--- | :--- | :--- |
| `refinelab_forms_rate_limit` | filter | `array( 'max' => int, 'window' => seconds ), string $form_id` | Different limits per form. |
| `refinelab_forms_recaptcha_min_score` | filter | `float 0.5` | Minimum reCAPTCHA v3 score. |
| `refinelab_security_client_ip` | filter | `string $ip` | Which header holds the real visitor IP behind a proxy (also used by rate limiting). |

```php
// Stricter limit for the quote form: 2 per hour.
add_filter( 'refinelab_forms_rate_limit', function ( $limit, $form_id ) {
    if ( 'quote' === $form_id ) {
        return array( 'max' => 2, 'window' => HOUR_IN_SECONDS );
    }
    return $limit;
}, 10, 2 );

// Visitors come through Cloudflare.
add_filter( 'refinelab_security_client_ip', function ( $ip ) {
    return isset( $_SERVER['HTTP_CF_CONNECTING_IP'] ) ? sanitize_text_field( wp_unslash( $_SERVER['HTTP_CF_CONNECTING_IP'] ) ) : $ip;
} );
```

> Notes: the Forms page in the admin shows a PHP snippet that mentions the action `refinelab_forms_after_submission`, the filter `refinelab_forms_notification_recipient` and the attributes `data-success-msg` / `data-error-msg`. These are **not implemented** in the current code; use the webhook for post-submission integrations and the settings for messages and recipient. `data-response` and `data-email` are parsed by the block editor but ignored at submission time.

## 6. Entries

**Entries by language.** When the Languages module is on with two or more languages, each entry remembers the language of the page the form was sent from (the front-end script sends it; the server accepts only a configured language code and otherwise leaves the entry untagged). The language is stored in the post meta `_refinelab_lang`, the same key as for translated content. Entries saved earlier, or sent when only one language was configured, count as the **default language**, like untagged posts. You get:

- a **Language** column and the *All Languages | English (n) | 繁體中文 (n)* links above the Entries list (they follow the admin language switch);
- the language on the entry screen and as a `Language:` line in the notification e-mail;
- a **Language** column (code) at the end of the CSV export, and an export limited to the language selected in the list;
- a `language` field in the webhook payload (the code, or an empty string when entries are not split by language).


- Custom post type `refine_entry` (menu **Entries**). Meta keys: `_refinelab_lang`, `_refinelab_form_id`, `_refinelab_form_label`, `_refinelab_fields` (array), `_refinelab_sender_email`, `_refinelab_sender_name`, `_refinelab_ip`, `_refinelab_user_agent`, `_refinelab_attachments`, `_refinelab_webhook`.
- **Export CSV** from the Entries screen (`admin_post_refinelab_export_submissions_csv`): batched for large lists, and any cell starting with `=`, `+`, `-` or `@` is neutralised so spreadsheets cannot run it as a formula.
- Entries from older versions (post type `refinelab_submission`) are migrated automatically once.

## 7. Troubleshooting

| Problem | Check |
| :--- | :--- |
| Nothing happens on submit | Is the `<form>` carrying `data-form`? Is **Forms → enabled** on? Open the browser console. |
| "Too many submissions" | Rate limit reached for that IP; raise it in settings or with the filter. |
| No e-mail but entries exist | `send_email` off, invalid recipient, or the server cannot send mail: configure [Mailer](modules.md#mailer) and send a test message. |
| reCAPTCHA fails | Keys belong to the other version (v2 vs v3), or the domain is not registered with Google. |
| Upload rejected | Attachments off, extension not in `attachments_types`, or the file is larger than `attachments_max_mb`. |
