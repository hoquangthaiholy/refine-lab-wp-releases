# 語言（多語言內容）

[English](../multilang.md)

在同一個 WordPress 網站上執行多種語言：語言網址、翻譯的文章與詞彙、語言切換器、每種語言各自的頁首／頁尾範本部件，以及顯示哪些內容已翻譯、已過期或缺漏的儀表板。

後台：**Refine Lab → 語言**（設定分頁與**翻譯**分頁）。模組類別 `RefineLab_MultiLang`；選項 `refinelab_multilang_settings`（設定）與 `refinelab_multilang_strings`（字串翻譯）。

## 1. 設定

| 鍵 | 預設 | 意義 |
| :--- | :--- | :--- |
| `enabled` | 開 | 模組總開關。 |
| `languages` | English | 語言清單。每一筆：`code`（例如 `vi`）、`name`、`prefix`（網址片段，預設為 code）、`flag`（表情符號）、`locale`（WordPress 地區代碼，如 `vi` 或 `zh_TW`）、`hidden`（準備某個語言期間，只有管理員看得到）。 |
| `default_lang` | `en` | 用於沒有語言的內容與網站根目錄的語言。 |
| `url_mode`／`url_format` | `prefix` | `prefix` 得到 `/vi/about/`；`query` 得到 `/about/?lang=vi`。 |
| `hide_default_prefix` | 開 | 預設語言位於 `/about/`（沒有 `/en/`）。 |
| `browser_redirect` | 關 | 將首次造訪者導向其瀏覽器的語言。 |
| `missing_translation_behavior` | `home` | 頁面在所請求的語言沒有翻譯時：`home` 轉址到該語言的首頁，`404` 顯示找不到頁面。 |
| `sync_featured_image`、`sync_post_template`、`sync_taxonomies` | 開 | 從來源建立新翻譯時一併複製。 |

目前語言也會驅動 WordPress 的 `locale` 篩選器與 `<html lang>` 屬性（`language_attributes`），因此翻譯過的外掛與佈景主題字串會以正確的語言載入。

## 2. 翻譯內容

文章、頁面與你允許的文章類型（篩選器 `refinelab_multilang_post_types`）會在編輯器中得到**語言**方塊，並在清單表格中顯示旗幟欄與位於清單上方的語言篩選列。

- 在方塊中**建立或連結翻譯**：每個語言都有一個*複製（Duplicate）*動作，會建立草稿翻譯並複製內容、特色圖片、範本與詞彙（依 `sync_*` 選項）；*將此文章與另一篇文章連結*選擇器會把你已寫好的文章加入同一組，*取消連結（Unlink）*則讓文章變成獨立的。
- 同一內容的所有翻譯共用一個**翻譯群組**（`_refinelab_translation_group`），每篇文章的語言存於 `_refinelab_lang`。沒有語言的文章視為預設語言。
- **詞彙**（分類、標籤、自訂分類法）以相同方式翻譯；彙整頁只顯示目前語言的詞彙。
- **翻譯**分頁是內容 × 語言的矩陣，狀態為*已翻譯*、*缺漏*或*已過期*（來源文章，即群組中最舊的一篇，在翻譯之後被修改過）。可依類型、語言或狀態篩選，並批次將缺漏的翻譯建立為草稿。

## 3. 語言切換器

### 短代碼

```text
[refinelab_language_switcher format="both" layout="list"]
[refinelab_lang_switcher layout="dropdown" format="names"]
```

| 屬性 | 值 | 預設 |
| :--- | :--- | :--- |
| `format` | `both`、`flags`、`names` | `both` |
| `layout` | `list`、`dropdown` | `list` |

輸出：`<div class="refinelab-lang-switcher refinelab-layout-list|dropdown">`，每個語言一個連結（`refinelab-lang-flag`、`refinelab-lang-name`；下拉式另有 `refinelab-dropdown-wrap` 與 `refinelab-dropdown-toggle`）。每個連結指向*目前頁面*在該語言的翻譯，沒有翻譯時指向該語言的首頁。隱藏的語言只有管理員看得到。

### 在 HTML 區塊中

```html
<ul data-lang-switcher>
  <li><a href="#" data-lang="en">English</a></li>
</ul>
```

第一個 `<a>` 是範本：它會為每個語言複製一份，並填入 `href`、`data-lang` 與名稱／旗幟。見 [html-blocks.md](html-blocks.md)。

### 在 PHP 範本中

```php
// 目前情境下所有語言與連結資料。
foreach ( refinelab_get_languages() as $lang ) {
    printf(
        '<a class="%s" href="%s" hreflang="%s">%s</a>',
        esc_attr( $lang['class'] ),
        esc_url( $lang['url'] ),
        esc_attr( $lang['code'] ),
        esc_html( $lang['label'] )
    );
}

$code = refinelab_get_current_language();         // 'vi'
$data = refinelab_get_current_language_data();    // 陣列結構與上方單一項目相同
```

每個語言項目包含：`code`、`name`、`flag`、`prefix`、`locale`、`url`（別名 `link`）、`is_current`（別名 `current`）、`active`（`is-active` 或空）、`class`（`lang-item lang-<code>` 加上 `is-active current-lang`／隱藏標記）、`label`（旗幟加名稱）、`hidden`（別名 `is_hidden`）。`refinelab_get_languages( $post_id )` 可傳入選擇性的文章 id，以連結到*該文章*的翻譯。

在區塊範本中，相同資料以全域變數 `languages`、`current_language` 以及扁平的 `current_lang_code`、`current_lang_name`、`current_lang_flag`、`current_lang_label`、`current_lang_url` 提供。

## 4. 翻譯固定字串

不屬於內容的文字（按鈕標籤、「Read more」）可以依語言翻譯，不需要 `.po` 檔。**目前還沒有此表的後台畫面**：翻譯存放在選項 `refinelab_multilang_strings`，結構為「原文 → 語言代碼 → 翻譯」，以程式碼或 WP-CLI 編輯。

```php
// 註冊翻譯（例如在佈景主題設定流程或一次性指令碼中）。
refinelab_tools()->multilang->save_strings( array(
    'Read more'  => array( 'vi' => 'Xem thêm', 'zh' => '閱讀更多' ),
    'Contact us' => array( 'vi' => 'Liên hệ' ),
) );
```

```bash
wp option get refinelab_multilang_strings --format=json
```

- 在前台，模組會攔截 `gettext` 查詢：當原文在表中，且目前語言有非空的翻譯時，就使用該翻譯（後台完全不受影響）。
- 在你自己的佈景主題程式碼中使用輔助函式；沒有翻譯時它們會回傳原文：

```php
<?php refinelab_e( 'Read more' ); ?>               // 輸出已跳脫的翻譯
<a href="#"><?php echo esc_html( refinelab__( 'Contact us' ) ); ?></a>
```

此表會包含在設定匯出中（*String translations*）。

## 5. 每種語言的頁首與頁尾

在網站編輯器中，範本部件可以依語言指派，讓每種語言有自己的頁首與頁尾。模組也提供資料摘要與刪除工具，在你停用某個語言時移除該語言的所有內容（後台 AJAX `refinelab_language_data_summary`、`refinelab_delete_language_data`）。

## 6. 掛鉤與端點

| 項目 | 類型 | 用途 |
| :--- | :--- | :--- |
| `refinelab_multilang_post_types` | filter（`string[]`） | 哪些文章類型會有語言方塊、旗幟欄與篩選列。內容區塊透過它加入其隱藏的文章類型。 |
| `locale`、`language_attributes`、`page_row_actions` | 模組使用的 WordPress 篩選器 | 目前地區代碼、`<html lang>`、清單表格的快速操作。 |
| `admin_post_refinelab_assign_language`、`admin_post_refinelab_duplicate_translation`、`admin_post_refinelab_unlink_translation` | admin-post | 指派語言、複製到某語言、取消翻譯連結。 |
| `refinelab_link_translation_group`、`refinelab_unlink_translation_group` | 後台 AJAX | 從編輯器方塊連結／取消連結。 |
| `admin_post_refinelab_translations_bulk` | admin-post | 批次建立缺漏的翻譯。 |

```php
// 讓自訂文章類型也有語言介面。
add_filter( 'refinelab_multilang_post_types', function ( $types ) {
    $types[] = 'portfolio';
    return $types;
} );
```

## 7. 小提醒

- 新語言先設為**隱藏**，翻譯完成後再公開。
- 變更 `url_mode` 或前綴後，到**設定 → 固定網址**儲存一次，讓重寫規則更新。
- 翻譯文章是否使用相同的 slug 結構由你決定；語言前綴本身就能讓網址保持唯一。
