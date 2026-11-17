# Languages (multilingual content)

**Language:** English · [繁體中文](zh-TW/multilang.md)

Run one WordPress site in several languages with language URLs, translated posts and terms, a language switcher, per-language header/footer template parts and a dashboard that shows what is translated, outdated or missing.

Admin: **Refine Lab → Languages** (Settings tab and **Translations** tab). Module class `RefineLab_MultiLang`; options `refinelab_multilang_settings` (configuration) and `refinelab_multilang_strings` (string translations).

## 1. Configuration

| Key | Default | Meaning |
| :--- | :--- | :--- |
| `enabled` | on | Master switch of the module. |
| `languages` | English | List of languages. Each entry: `code` (e.g. `vi`), `name`, `prefix` (URL part, defaults to the code), `flag` (an emoji), `locale` (WordPress locale such as `vi` or `zh_TW`), `hidden` (visible only to administrators while you prepare a language). |
| `default_lang` | `en` | Language used for content without a language and for the site root. |
| `url_mode` / `url_format` | `prefix` | `prefix` gives `/vi/about/`; `query` gives `/about/?lang=vi`. |
| `hide_default_prefix` | on | The default language lives at `/about/` (no `/en/`). |
| `browser_redirect` | off | Send first-time visitors to the language of their browser. |
| `missing_translation_behavior` | `home` | When a page has no translation in the requested language: `home` redirects to that language's home page, `404` shows a not-found page. |
| `sync_featured_image`, `sync_post_template`, `sync_taxonomies` | on | Copied to a new translation when it is created from the source. |

The active language also drives the WordPress `locale` filter and the `<html lang>` attribute (`language_attributes`), so translated plugin and theme strings load with the right language.

## 2. Translating content

Posts, pages and the post types you allow (filter `refinelab_multilang_post_types`) get a **Language** box in the editor and flag columns in the list tables, with a language filter bar above the list.

- **Create or link a translation** from the box: a *Duplicate* action per language creates a draft translation that copies the content, featured image, template and terms (according to the `sync_*` options); the *Link this post with another post* selector joins posts you already wrote into one group, and *Unlink* turns a post into an independent one.
- Every translation of the same content shares a **translation group** (`_refinelab_translation_group`) and each post has its language in `_refinelab_lang`. A post without a language counts as the default language.
- **Terms** (categories, tags, custom taxonomies) are translated the same way; archives show only terms of the current language.
- The **Translations** tab is a matrix of content × languages with the status *translated*, *missing* or *outdated* (the source post, the oldest in the group, was modified after the translation). Filter by type, language or status and bulk-create the missing translations as drafts.

## 3. Language switcher

### Shortcode

```text
[refinelab_language_switcher format="both" layout="list"]
[refinelab_lang_switcher layout="dropdown" format="names"]
```

| Attribute | Values | Default |
| :--- | :--- | :--- |
| `format` | `both`, `flags`, `names` | `both` |
| `layout` | `list`, `dropdown` | `list` |

Output: `<div class="refinelab-lang-switcher refinelab-layout-list|dropdown">` with one link per language (`refinelab-lang-flag`, `refinelab-lang-name`; the dropdown adds `refinelab-dropdown-wrap` and `refinelab-dropdown-toggle`). Each link points to the translation of the *current page* in that language, or to the language's home page when there is none. Hidden languages are shown only to administrators.

### In an HTML block

```html
<ul data-lang-switcher>
  <li><a href="#" data-lang="en">English</a></li>
</ul>
```

The first `<a>` is the sample: it is cloned once per language with `href`, `data-lang` and the name/flag filled in. See [html-blocks.md](html-blocks.md).

### In PHP templates

```php
// All languages with link data for the current context.
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
$data = refinelab_get_current_language_data();    // same array shape as one item above
```

Each language item contains: `code`, `name`, `flag`, `prefix`, `locale`, `url` (alias `link`), `is_current` (alias `current`), `active` (`is-active` or empty), `class` (`lang-item lang-<code>` plus `is-active current-lang` / hidden markers), `label` (flag and name), `hidden` (alias `is_hidden`). `refinelab_get_languages( $post_id )` takes an optional post id to link to *that post's* translations.

In block templates the same data is available as the globals `languages`, `current_language` and the flat `current_lang_code`, `current_lang_name`, `current_lang_flag`, `current_lang_label`, `current_lang_url`.

## 4. Translating fixed strings

Texts that are not content (button labels, "Read more") can be translated per language without `.po` files. There is **no admin screen for this table yet**: the translations live in the option `refinelab_multilang_strings`, shaped as `original text → language code → translation`, and are edited in code or with WP-CLI.

```php
// Register translations (for example from a theme setup routine or a one-off script).
refinelab_tools()->multilang->save_strings( array(
    'Read more'  => array( 'vi' => 'Xem thêm', 'zh' => '閱讀更多' ),
    'Contact us' => array( 'vi' => 'Liên hệ' ),
) );
```

```bash
wp option get refinelab_multilang_strings --format=json
```

- On the front end the module intercepts `gettext` lookups: when the original text is in the table and the current language has a non-empty translation, that translation is used (the admin is never affected).
- In your own theme code use the helpers; they return the original text when no translation exists:

```php
<?php refinelab_e( 'Read more' ); ?>               // prints the escaped translation
<a href="#"><?php echo esc_html( refinelab__( 'Contact us' ) ); ?></a>
```

The table is included in settings exports (*String translations*).

## 5. Header and footer per language

In the Site Editor, template parts can be assigned per language so each language gets its own header and footer. The module also provides a data summary and a delete tool to remove all content of a language when you retire it (admin AJAX `refinelab_language_data_summary`, `refinelab_delete_language_data`).

## 6. Hooks and endpoints

| Item | Type | Purpose |
| :--- | :--- | :--- |
| `refinelab_multilang_post_types` | filter (`string[]`) | Post types that get the language box, flag columns and filter bar. Content Blocks adds its hidden post type through it. |
| `locale`, `language_attributes`, `page_row_actions` | WordPress filters used by the module | Active locale, `<html lang>`, quick actions in list tables. |
| `admin_post_refinelab_assign_language`, `admin_post_refinelab_duplicate_translation`, `admin_post_refinelab_unlink_translation` | admin-post | Assign a language, duplicate into a language, detach a translation. |
| `refinelab_link_translation_group`, `refinelab_unlink_translation_group` | admin AJAX | Link / unlink from the editor box. |
| `admin_post_refinelab_translations_bulk` | admin-post | Bulk create missing translations. |

```php
// Give a custom post type the language UI.
add_filter( 'refinelab_multilang_post_types', function ( $types ) {
    $types[] = 'portfolio';
    return $types;
} );
```

## 7. Tips

- Prepare a new language as **hidden**, translate, then reveal it.
- After changing `url_mode` or prefixes, save **Settings → Permalinks** once so rewrite rules refresh.
- Use the same slug structure for translated posts only when you want it; the language prefix already keeps URLs unique.
