# 表單

[English](../forms.md)

零設定的表單處理：寫一個普通的 HTML `<form>`，加上 `data-form`，Refine Lab 就會驗證它、將提交內容儲存為**項目（Entry）**、寄出通知郵件，並可轉送到 webhook。過程中不需要短代碼，也不需要表單建立器。

後台：**Refine Lab → 表單**（設定與統計）與**項目（Entries）**（提交清單）。模組類別 `RefineLab_Forms`，選項 `refinelab_forms_settings`，項目是自訂文章類型 `refine_entry`。

## 1. 標記

```html
<form data-form="contact" data-form-label="Contact form" data-redirect="/thank-you/">
  <input data-form-field="name" type="text" placeholder="Full name" required />
  <input data-form-field="email" type="email" placeholder="Email" required />
  <textarea data-form-field="message" placeholder="Message"></textarea>
  <button type="submit">Send</button>
</form>
```

| 屬性 | 位置 | 意義 |
| :--- | :--- | :--- |
| `data-form="id"` | `<form>` | 必填。表單識別碼，用於在後台篩選項目及郵件主旨。頁面載入時會自動偵測表單。 |
| `data-form-label="text"` | `<form>` | 項目與郵件中顯示的易讀名稱（沒有時退回使用 id）。`data-label` 可作為備用。 |
| `data-redirect="url"` | `<form>` | 提交成功後開啟的頁面。沒有此屬性則顯示成功訊息。 |
| `data-form-field="key"` | `input`、`textarea`、`select` | 為項目中的欄位命名。沒有時依序使用欄位的 `name`、`id`。 |
| `data-captcha="v2"` | 表單內的任何元素 | reCAPTCHA v2 核取方塊繪製的位置。省略時，若已設定 v2，會在送出按鈕前自動建立一個。 |

名為 `g-recaptcha-response`、`_refinelab_hp` 的欄位與送出按鈕絕不會被儲存。核取方塊群組與多選會以清單儲存，郵件中以逗號連接。通知郵件的寄件者（`Reply-To`）取自鍵名為 `email`、`user_email`、`contact_email` 或 `mail` 的欄位（必須是有效的電子郵件），寄件者名稱取自 `name`、`full_name`、`username` 或 `contact_name`。

### 檔案上傳

```html
<form data-form="job-application" data-form-label="Job application">
  <input data-form-field="applicant_name" type="text" required />
  <input data-form-field="resume" type="file" accept=".pdf,.doc,.docx" />
  <button type="submit">Apply</button>
</form>
```

上傳需要開啟 **表單 → 附件**。每個檔案都會依允許的副檔名、實際 MIME 類型與大小上限檢查，以隨機名稱儲存在 `uploads/refinelab-forms/`（無法從網頁存取），並附加到項目上。編輯者只能透過項目畫面上有權限檢查的連結下載。

## 2. 設定（Refine Lab → 表單）

| 鍵 | 預設 | 意義 |
| :--- | :--- | :--- |
| `enabled` | 開 | 前台指令碼的總開關。 |
| `send_email` | 開 | 每次提交都寄出通知。項目一律會儲存；此選項只控制郵件。 |
| `notification_email` | 網站管理員信箱 | 收件者。 |
| `email_subject` | `New Contact Form Submission: {form_id}` | 佔位符：`{form_id}`、`{form_label}`（有標籤時兩者都以標籤代入）與 `{site_name}`。 |
| `success_msg`／`error_msg` | 英文文字 | 回覆給訪客的訊息。 |
| `enable_honeypot` | 開 | 加入隱藏欄位 `_refinelab_hp`；填寫它的提交會被靜默丟棄（機器人仍會看到成功訊息）。 |
| `recaptcha_version`、`recaptcha_site_key`、`recaptcha_secret` | `v2`、空 | Google reCAPTCHA。v2 顯示核取方塊；v3 在背景對所有表單進行無感驗證。 |
| `rate_limit_enabled`、`rate_limit_max`、`rate_limit_window` | 開、5、10 | 每個表單、每個訪客 IP 在 `window` **分鐘**內最多 `max` 次提交。 |
| `webhook_url`、`webhook_secret` | 空 | 將每個項目轉送到你的服務（見下文）。 |
| `attachments_enabled`、`attachments_max_mb`、`attachments_types` | 關、5、`pdf,jpg,jpeg,png,webp,doc,docx,txt` | 檔案上傳政策。允許的副檔名限於 `pdf, jpg, jpeg, png, gif, webp, doc, docx, xls, xlsx, ppt, pptx, txt, csv`。 |

通知內容包含表單名稱與 id、日期、IP、每個欄位，以及項目的連結。每個表單的頻率限制可用篩選器 `refinelab_forms_rate_limit` 調整（見下文）。統計卡片顯示最近 30 天每天與每個表單的提交數。

## 3. 瀏覽器端的行為

1. `assets/js/syntax-forms.js` 找出每個 `form[data-form]`、加入 honeypot，並在已設定時渲染 reCAPTCHA。
2. 送出時，它把欄位以 `FormData` 連同 nonce 傳到 `admin-ajax.php`（`action=refinelab_form_submit`）。
3. 伺服器回應 JSON：`{ "success": true, "data": { "message": "…", "redirect": "…" } }` 或 `{ "success": false, "data": { "message": "…" } }`（安全性 nonce 無效時 HTTP 403、超過頻率限制時 429、reCAPTCHA 與上傳錯誤時 400、無法儲存項目時 500；填寫了 honeypot 的請求會得到假的成功，讓機器人不會重試）。
4. 指令碼顯示訊息（或前往 `data-redirect`）並觸發 DOM 事件。

```js
document.addEventListener('refinelab:form:success', function (e) {
  // e.detail = { formId: 'contact', response: { ...伺服器資料... } }
  if (e.detail.formId === 'contact') {
    window.dataLayer && window.dataLayer.push({ event: 'lead', form: e.detail.formId });
  }
});
```

指令碼與其樣式表只會載入在含有表單的頁面。執行時設定可從 `window.refinelabFormsConfig` 取得（`ajaxUrl`、`nonce`、`honeypot`、`recaptchaKey`、`recaptchaVersion`、`i18n`）。

## 4. Webhook

設定 `webhook_url` 後，每個儲存的項目都會排入佇列，稍後透過 WP-Cron 以 JSON 方式 POST（訪客不必等待你的伺服器）：

```json
{
  "event": "form.submitted",
  "entry_id": 123,
  "form_id": "contact",
  "form_label": "Contact form",
  "submitted_at": "2026-11-17T07:00:00+00:00",
  "site": "https://example.com",
  "fields": { "name": "Ada", "email": "ada@example.com", "message": "Hi" },
  "attachments": [ { "name": "cv.pdf", "size": 52311, "type": "application/pdf" } ]
}
```

標頭：`X-Refinelab-Event: form.submitted`、`X-Refinelab-Attempt: 1|2|3`，若有設定密鑰則還有 `X-Refinelab-Signature: sha256=<hmac>`，其中 HMAC-SHA256 的金鑰是你的密鑰，訊息是原始請求本文。信任請求前請先驗證：

```php
$body = file_get_contents( 'php://input' );
$sent = $_SERVER['HTTP_X_REFINELAB_SIGNATURE'] ?? '';
$calc = 'sha256=' . hash_hmac( 'sha256', $body, 'your-secret' );
if ( ! hash_equals( $calc, $sent ) ) { http_response_code( 401 ); exit; }
```

非 2xx 的回應視為失敗。傳送會重試兩次（5 分鐘後與 30 分鐘後）；每個項目的狀態顯示在其畫面上，最近的失敗記錄保存在選項 `refinelab_forms_webhook_log`。不會跟隨轉址，私有或不安全的位址會被拒絕。

## 5. 掛鉤

| 掛鉤 | 類型 | 參數 | 用途 |
| :--- | :--- | :--- | :--- |
| `refinelab_forms_rate_limit` | filter | `array( 'max' => int, 'window' => 秒 ), string $form_id` | 為不同表單設定不同限制。 |
| `refinelab_forms_recaptcha_min_score` | filter | `float 0.5` | reCAPTCHA v3 的最低分數。 |
| `refinelab_security_client_ip` | filter | `string $ip` | 位於代理之後時，由哪個標頭取得真實訪客 IP（頻率限制也會使用）。 |

```php
// 報價表單採更嚴格的限制：每小時 2 次。
add_filter( 'refinelab_forms_rate_limit', function ( $limit, $form_id ) {
    if ( 'quote' === $form_id ) {
        return array( 'max' => 2, 'window' => HOUR_IN_SECONDS );
    }
    return $limit;
}, 10, 2 );

// 訪客經由 Cloudflare 連入。
add_filter( 'refinelab_security_client_ip', function ( $ip ) {
    return isset( $_SERVER['HTTP_CF_CONNECTING_IP'] ) ? sanitize_text_field( wp_unslash( $_SERVER['HTTP_CF_CONNECTING_IP'] ) ) : $ip;
} );
```

> 注意：後台表單頁面顯示的 PHP 範例提到動作 `refinelab_forms_after_submission`、篩選器 `refinelab_forms_notification_recipient`，以及屬性 `data-success-msg`／`data-error-msg`。這些在目前的程式碼中**尚未實作**；提交後的整合請使用 webhook，訊息與收件者請使用設定。`data-response` 與 `data-email` 會被區塊編輯器解析，但在提交時會被忽略。

## 6. 項目（Entries）

- 自訂文章類型 `refine_entry`（選單**項目 Entries**）。中繼資料鍵：`_refinelab_form_id`、`_refinelab_form_label`、`_refinelab_fields`（陣列）、`_refinelab_sender_email`、`_refinelab_sender_name`、`_refinelab_ip`、`_refinelab_user_agent`、`_refinelab_attachments`、`_refinelab_webhook`。
- 從項目畫面**匯出 CSV**（`admin_post_refinelab_export_submissions_csv`）：大型清單會分批處理，且以 `=`、`+`、`-` 或 `@` 開頭的儲存格會被中和，試算表不會把它當作公式執行。
- 舊版本的項目（文章類型 `refinelab_submission`）會自動遷移一次。

## 7. 疑難排解

| 問題 | 檢查 |
| :--- | :--- |
| 送出後沒有反應 | `<form>` 有 `data-form` 嗎？**表單 → 啟用**開著嗎？開啟瀏覽器主控台查看。 |
| 「Too many submissions」 | 該 IP 已達頻率限制；到設定或用篩選器調高。 |
| 有項目但沒有郵件 | `send_email` 關閉、收件者無效，或伺服器無法寄信：設定 [郵件](modules.md#mailer) 並寄出測試信。 |
| reCAPTCHA 失敗 | 金鑰屬於另一個版本（v2 與 v3），或網域未向 Google 註冊。 |
| 上傳被拒 | 附件功能關閉、副檔名不在 `attachments_types`，或檔案超過 `attachments_max_mb`。 |
