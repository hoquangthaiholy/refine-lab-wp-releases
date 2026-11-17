# HTML Blocks — authoring reference

The engine scans the active theme (and parent theme) for block templates and registers each as a Gutenberg block `theme/<slug>`.

Scanned directories: `<theme>/blocks/`, `<theme>/template-parts/blocks/`, `<theme>/html-blocks/`. Extend with the filter `refinelab_html_blocks_directories`. A block is either `<slug>.html` or `<slug>/index.html` (multi-file, see §9). Block definitions are cached in the transient `refinelab_blocks_<md5(stylesheet)>`, invalidated by file mtime, theme switch and `save_post`.

## 1. Root element attributes

One root element per template. Optional metadata:

| Attribute | Values | Default | Effect |
| :--- | :--- | :--- | :--- |
| `data-title` | string | from filename | Name in the inserter. |
| `data-name` | `slug` or `ns/slug` | `theme/<file slug>` | Block name; `theme/` is prefixed when no namespace is given. |
| `data-category` | `theme` or a core category slug | theme category | Inserter category. Slug/title filters: `refinelab_html_blocks_category_slug`, `refinelab_html_blocks_category_title`. |
| `data-icon` | Dashicon slug | `block-default` | Inserter icon. |
| `data-description` | string | empty | Description in the inspector. |
| `data-parent` | slug or comma list | empty | Restricts insertion to children of those blocks. |
| `data-align` | `full` `wide` `center` `left` `right` | `full` | Default alignment. |
| `data-editing-mode` | `default` `contentOnly` `disabled` | `default` | Block editing mode; the attribute is removed from the front end. |
| `data-supports` | `margin`, `padding`, `spacing` (both), comma list | none | Opt in to native Gutenberg block supports. |

## 2. Bindings: `data-bind`, `data-bind-prop`

`data-bind="key"` makes an element editable and creates block attributes from `key`:

| Element | Stored attributes | Editor UI |
| :--- | :--- | :--- |
| `h1`–`h6` | `key`, `key_align`, `key_tag` | RichText, level switcher H1–H6, alignment. |
| `p`, `span`, `div`, `li`, … | `key`, `key_align` | RichText (`innerHTML`). |
| `<img>` | `key` (media id), `key_url`, `key_alt` (+ `srcset`/`sizes`) | Media Library, external URL, alt text. |
| `<a>` | `key` (text), `key_url`, `key_new_tab` (+ `rel`) | Link popover. |

`data-bind-prop="attr"` targets a single attribute instead (`placeholder`, `title`, `aria-label`, `style`, `href`, `data-lucide`, …):

```html
<input data-bind="hint" data-bind-prop="placeholder" placeholder="Search…">
<div data-bind="bg" data-bind-prop="style" style="background:#eee">
<i data-bind="icon" data-bind-prop="data-lucide" data-lucide="star"></i>
```

Rules: keys are unique per block; never rename a key once content exists. Sample text in the template is the default value.

## 3. Repeaters

`data-repeater="items"` on the repeated element; several sample elements become several default items.

| Attribute | Where | Effect |
| :--- | :--- | :--- |
| `data-repeater` | repeated element | Repeater key (array attribute). |
| `data-label` | repeated element | Item noun in the UI (“Plan #1”). |
| `data-mode` | first node | Repeater mode (default `list`). |
| `data-option-values="a:A\|b:B"` | repeated element | Per-item choice in the toolbar; combine with `data-bind` + `data-bind-prop`. |
| `data-option-label` | repeated element | Label of that choice. |
| `data-repeater-item-label="key"` | repeater | List items by the text of control `key` instead of “Item n”. |
| `data-index` | elements sharing one repeater | Spreads one repeater over several containers; the selected item gets `.is-item-active` and the block root `.refinelab-has-active-item` + `data-active-repeater` / `data-active-index`. |

On-canvas actions: Move up, Move down, Duplicate, Delete, Add.

**Nested repeaters.** A repeater may contain another repeater with a different key (a pricing plan with a list of features). Every sample item can carry its own sample sub-items, which become that item's defaults. Give both levels `data-label` and `data-repeater-item-label` so the sidebar list shows readable titles.

```html
<div data-repeater="plans" data-label="Plan" data-repeater-item-label="planTitle">
  <h3 data-bind="planTitle">Basic</h3>
  <ul>
    <li data-repeater="features" data-label="Feature" data-repeater-item-label="featureText" data-bind="featureText">1 website</li>
    <li data-repeater="features" data-label="Feature" data-repeater-item-label="featureText" data-bind="featureText">Email support</li>
  </ul>
</div>
```

**Field editing control.** In the "Edit all items" dialog a bound field is a single-line input by default. Add `data-bind-control="textarea"` (or `text`) next to `data-bind` to choose the control for long text:

```html
<p class="plan-desc" data-bind="planDesc" data-bind-control="textarea">For personal websites.</p>
```

## 4. Trees (hierarchical menus)

| Attribute | Where | Effect |
| :--- | :--- | :--- |
| `data-tree="key"` + `data-label` | container | Tree attribute. |
| `data-tree-item` | each item | Item template. |
| `data-tree-children` | child container (or any `ul`/`ol`) | Children slot. |
| `data-tree-grid` | container | Grid-style tree. |
| `data-tree-part="left\|right"` | several containers | Split one tree across containers. |
| `data-tree-slice="offset,length"` | container | Render only a range of the items. |
| `data-tree-item-label="control"` | container | Title each item with that control’s text. |

Items without children lose `has-dropdown` and `chevron-down`. Do not nest a repeater inside a tree.

## 5. Placeholders `{{ … }}`

In text and attribute values: `{{ key }}`, `{{ site.url }}`, `{{ context.title }}`, `{{ current_lang_name }}`, `{{ item.title }}` inside loops. Dot paths are supported. Block attributes created by inspector controls can drive class names, which is the usual way to offer layout switches:

```html
<section class="hero is-align-{{alignStyle}}">
  <inspector-controls>
    <panel-body title="Hero">
      <select-control data-bind="alignStyle" label="Alignment">
        <select-control-option value="center">Center</select-control-option>
        <select-control-option value="left">Left</select-control-option>
      </select-control>
    </panel-body>
  </inspector-controls>
  <h1 data-bind="title">Headline</h1>
</section>
```

## 6. Conditions and loops

| Directive | Purpose |
| :--- | :--- |
| `data-if="expr"` / `data-elseif="expr"` / `data-else` | Conditional rendering (server and editor). |
| `data-for="item in list"` / `data-for="(item, i) in list"` | Loop over a context array or attribute. |
| `data-fe-if`, `data-fe-elseif`, `data-fe-else`, `data-fe-for` | Same, evaluated on the front end only (editor shows the sample). |

Expressions: truthiness, `!x`, dot paths, `==`, `!=`, `===`, `!==`. Loop scope: `index`, `is_first`, `is_last` (or your aliases, e.g. `crumb.is_last`). Use `<template data-for>` as a transparent wrapper.

## 7. Declarative components

| Attribute | Behaviour |
| :--- | :--- |
| `data-breadcrumb` (+ `data-separator`) | Container filled with the breadcrumb trail; first `<a>`/`<span>` is the style sample; separator is `data-separator` or detected from the sample. |
| `data-lang-switcher` | Container filled with one clone of its first `<a>` per configured language (`href`, `data-lang`, name/flag). Needs the MultiLang module. |
| `data-form="id"` (+ `data-form-label`, `data-redirect`) | Marks a form handled by the Forms module (full guide: [forms.md](forms.md)). `data-form-label` = readable name in entries and e-mails; `data-redirect` = URL opened after success. Fields: any `input/textarea/select[name]` or `[data-form-field]`. The front-end script also dispatches the DOM event `refinelab:form:success` (`detail.formId`, `detail.response`). The editor offers per-block overrides, stored as `form_<id>_{id,label,redirect,response,email}` attributes. **Reserved:** `data-response` and `data-email` are parsed, editable in the inspector and written back to the HTML, but the current front-end script and server handler do not read them (the recipient and messages come from the Forms settings). |

## 8. Inspector controls

```html
<inspector-controls>
  <panel-body title="Settings" initialOpen="true">
    <text-control data-bind="k" label="…" help="…"></text-control>
    <textarea-control data-bind="k" rows="3"></textarea-control>
    <toggle-control data-bind="k" default="true"></toggle-control>
    <range-control data-bind="k" min="0" max="100" default="40"></range-control>
    <select-control data-bind="k" default="3"><select-control-option value="3">3</select-control-option></select-control>
    <color-picker data-bind="k" default="#fff"></color-picker>
    <link-control data-bind="k"></link-control>
    <code-control data-bind="k"></code-control>
    <post-picker data-bind="k" post-type="post" multiple="true" max_items="3" category="news" context="off"></post-picker>
    <term-picker data-bind="k" taxonomy="category" multiple="true" posts_bind="k_posts" posts_per_page="6" orderby="date" order="DESC" context="auto"></term-picker>
  </panel-body>
</inspector-controls>
```

Pickers are resolved to full post/term objects at render time and can be looped with `data-for`.

Fields on a resolved **post** item: `id`, `title`, `slug`, `type`, `url` (aliases `href`, `link`), `excerpt` (alias `desc`), `date` (`Y.m.d`), `date_formatted` (site date format), `author` (display name), `category` / `category_name` (first category), `categories` (names), `meta` (date plus category label), and, when the post has a featured image, `thumb` and related image keys. Fields on a resolved **term** item: `id`, `name`, `slug`, `taxonomy`, `count`, `url` (`href`, `link`), `description`, `posts` (the term's post items, filled according to `posts_bind`, `posts_per_page`, `orderby`, `order`). `context="auto|always|off"` chooses whether the current page is the source (editors override it with `<bind>_context`). Nothing resolves in the editor/REST previews.

`<inner-blocks allowed-blocks="core/paragraph,theme/card" template-lock="false">` declares nested blocks.

## 9. Global context objects

Available in every template (`refinelab_html_blocks_global_attributes` filters the array):

| Name | Content |
| :--- | :--- |
| `site` | `name`, `description`, `url`, `admin_url`, `language`, `charset`, `theme` |
| `context` | `title`, `slug`, `url`, `type`, `post_type`, `id`, `is_front_page`, `is_home`, `is_single`, `is_page`, `is_archive`, `is_category`, `is_tag`, `is_tax`, `is_search`, `is_404`, `paged` |
| `post` | `id`, `title`, `slug`, `type`, `url`, `excerpt`, `date`, `thumbnail`, `author` |
| `posts` | recent/queried post objects |
| `term`, `terms` | queried term; category list |
| `languages` | list of `code`, `name`, `flag`, `url`, `is_current`, `label` |
| `current_language` | active language object |
| `current_lang_code`, `current_lang_name`, `current_lang_flag`, `current_lang_label`, `current_lang_url` | flat shortcuts for the active language |
| `breadcrumbs` | crumbs with `title`, `url`, `is_current`, `is_last` (filter `refinelab_html_blocks_breadcrumb_list`) |

## 10. Multi-file blocks (`blocks/<slug>/`)

| File | Loaded | Purpose |
| :--- | :--- | :--- |
| `index.html` | always | The template (same syntax as a single-file block). |
| `index.css` | editor + front end | Shared styles. |
| `index.web.css`, `index.web.js` | front end only, only on pages that use the block | Front-end styles and behaviour. |
| `index.editor.css`, `index.editor.js` | editor only | Editor-only styles; lifecycle hooks (see §13). |
| `index.php` | server render | Replaces template rendering with your own PHP. Variables in scope: `$attributes` (block attributes), `$content` (rendered InnerBlocks), `$block` (`WP_Block`), `$block_def` (parsed block schema), `$renderer` (`RefineLab_HTML_Blocks`). Use `get_block_wrapper_attributes()` for the root element and escape everything you print. |

Link attributes are stored as `<key>` (text) and `<key>_url`; images as `<key>`, `<key>_url`, `<key>_alt`. A complete example ships in `templates/sample-blocks/interactive-box/` (Refine Lab → Blocks → install sample blocks).

## 11. Gotchas

- Icons: `<i data-lucide="name">`; themes must only replace `[data-lucide]:not(svg)` and never re-run `lucide.createIcons()` on every DOM mutation.
- Plain `href`s are disabled in the editor; bind links with `data-bind`.
- Theme styling belongs in the theme, scoped to `.editor-styles-wrapper` for the editor.
- Decorative images: `data-bg="true"` or `data-editor-interactive="false"` on an `<img>` (or an ancestor) keeps it out of the Media Library binding in the editor, for background or non-interactive pictures.
- Icons: `data-lucide` and `data-feather` are both preserved as icon markers.
- Cache stuck? Touch the file or call `admin_post_refinelab_clear_blocks_cache` (Refine Lab → Blocks → Clear cache).

## 12. Editor enhancements (Refine Lab → Enhancements)

- **Pin the post title**: sticky, blurred title in the editor (`enable_sticky_post_title`, default on).
- **Resizable sidebar**: drag the left edge of the editor sidebar to resize; double-click resets to 280 px; width saved in `localStorage['refinelab_sidebar_width']` (`enable_resizable_sidebar`, default on). When off, the body class `refinelab-resizable-sidebar` is not added, the scoped CSS does not apply and the JS handle is not created. Requires the HTML Blocks module (the editor bundle) to be enabled.

## 13. Client-side lifecycle (JavaScript)

Register behaviour for a block with `window.refineLab.registerBlock( slugOrName, hooks )`. `slugOrName` is `hero` or `theme/hero`.

```js
(function (refineLab) {
  refineLab.registerBlock('hero', {
    // Front end and editor: called once per block element when it is ready.
    init: function (context) { /* context.el, context.name, context.slug */ },

    // Editor only
    insert:    function (context) { /* block newly added to the canvas */ },
    update:    function (context) { /* context.attributes, context.prevAttributes */ },
    destroyed: function (context) { /* context.name, context.clientId */ }
  });
})(window.refineLab = window.refineLab || {});
```

| Where | Context | Notes |
| :--- | :--- | :--- |
| Front end (`index.web.js` or any theme script) | `{ el, name, slug, isEditor: false }` | Runs on `DOMContentLoaded`, or immediately if the DOM is ready; each element is initialised once (`data-refinelab-initialized`). The root is matched by `.wp-block-theme-<slug>`, `.wp-block-<slug>` or `[data-block]`. |
| Editor (`index.editor.js`) | `{ el, name, clientId, attributes, prevAttributes, … }` | Hooks run for each block instance. |

The same events are published for other code:

- DOM events on the block root: `refinelab:init` (front end), `refinelab:insert`, `refinelab:init`, `refinelab:update`, `refinelab:destroyed` (editor), all bubbling, with the context in `event.detail`.
- Editor `wp.hooks` actions: `refinelab.block.<event>`, `refinelab.block.<slug>.<event>` and `refinelab.block.theme.<slug>.<event>`, called with `( context, blockDef )`.

```js
wp.hooks.addAction('refinelab.block.hero.update', 'my-theme', function (context) {
  console.log('Hero changed', context.attributes);
});
document.addEventListener('refinelab:init', function (e) { console.log(e.detail.slug); });
```

Helpers: `refineLab.getBlockHooks( slugOrName )` returns the registered hooks object.

## 14. Reading the sample blocks

`templates/sample-blocks/` contains runnable templates that show each feature (see [html-blocks-cookbook.md](html-blocks-cookbook.md)).
