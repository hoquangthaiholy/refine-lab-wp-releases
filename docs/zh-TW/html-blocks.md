# HTML 區塊：撰寫參考

[English](../html-blocks.md)

引擎會掃描目前的佈景主題（及父佈景主題）中的區塊範本，並將每一個註冊為 Gutenberg 區塊 `theme/<slug>`。

掃描的資料夾：`<theme>/blocks/`、`<theme>/template-parts/blocks/`、`<theme>/html-blocks/`。可用篩選器 `refinelab_html_blocks_directories` 擴充。一個區塊是 `<slug>.html` 或 `<slug>/index.html`（多檔案，見 §10）。區塊定義會快取在暫存資料 `refinelab_blocks_<md5(stylesheet)>`，會因檔案修改時間、切換佈景主題與 `save_post` 而失效。

## 1. 根元素屬性

每個範本只有一個根元素。可選的中繼資料：

| 屬性 | 值 | 預設 | 效果 |
| :--- | :--- | :--- | :--- |
| `data-title` | 字串 | 取自檔名 | 在插入器中的名稱。 |
| `data-name` | `slug` 或 `ns/slug` | `theme/<檔案 slug>` | 區塊名稱；未指定命名空間時會加上 `theme/` 前綴。 |
| `data-category` | `theme` 或核心分類 slug | 佈景主題分類 | 插入器分類。slug／標題篩選器：`refinelab_html_blocks_category_slug`、`refinelab_html_blocks_category_title`。 |
| `data-icon` | Dashicon slug | `block-default` | 插入器圖示。 |
| `data-description` | 字串 | 空 | 側邊欄中的說明。 |
| `data-parent` | slug 或以逗號分隔的清單 | 空 | 限制只能插入在這些區塊的子層。 |
| `data-align` | `full` `wide` `center` `left` `right` | `full` | 預設對齊。 |
| `data-editing-mode` | `default` `contentOnly` `disabled` | `default` | 區塊編輯模式；此屬性會從前台移除。 |
| `data-supports` | `margin`、`padding`、`spacing`（兩者皆啟用）、以逗號分隔的清單 | 無 | 選用原生 Gutenberg 區塊支援。 |

## 2. 綁定：`data-bind`、`data-bind-prop`

`data-bind="key"` 讓元素可編輯，並依 `key` 建立區塊屬性：

| 元素 | 儲存的屬性 | 編輯器介面 |
| :--- | :--- | :--- |
| `h1`–`h6` | `key`、`key_align`、`key_tag` | RichText、H1–H6 層級切換、對齊。 |
| `p`、`span`、`div`、`li`、… | `key`、`key_align` | RichText（`innerHTML`）。 |
| `<img>` | `key`（媒體 id）、`key_url`、`key_alt`（+ `srcset`／`sizes`） | 媒體庫、外部網址、替代文字。 |
| `<a>` | `key`（文字）、`key_url`、`key_new_tab`（+ `rel`） | 連結彈出視窗。 |

`data-bind-prop="attr"` 改為指定單一屬性（`placeholder`、`title`、`aria-label`、`style`、`href`、`data-lucide`、…）：

```html
<input data-bind="hint" data-bind-prop="placeholder" placeholder="Search…">
<div data-bind="bg" data-bind-prop="style" style="background:#eee">
<i data-bind="icon" data-bind-prop="data-lucide" data-lucide="star"></i>
```

規則：key 在每個區塊內必須唯一；內容建立後絕不要重新命名 key。範本中的示範文字就是預設值。

## 3. 重複器（Repeater）

在要重複的元素上加 `data-repeater="items"`；多個示範元素會成為多個預設項目。

| 屬性 | 位置 | 效果 |
| :--- | :--- | :--- |
| `data-repeater` | 重複的元素 | 重複器的 key（陣列屬性）。 |
| `data-label` | 重複的元素 | 介面中項目的名稱（「Plan #1」）。 |
| `data-mode` | 第一個節點 | 重複器模式（預設 `list`）。 |
| `data-option-values="a:A\|b:B"` | 重複的元素 | 工具列中每個項目的選項；可與 `data-bind` + `data-bind-prop` 搭配。 |
| `data-option-label` | 重複的元素 | 該選項的標籤。 |
| `data-repeater-item-label="key"` | 重複器 | 以控制項 `key` 的文字作為清單項目標題，取代「Item n」。 |
| `data-index` | 共用同一個重複器的元素 | 將一個重複器分散到多個容器；被選取的項目會得到 `.is-item-active`，區塊根元素會得到 `.refinelab-has-active-item` + `data-active-repeater`／`data-active-index`。 |

畫布上的操作：上移、下移、複製、刪除、新增。

**巢狀重複器。** 重複器內可以包含另一個 key 不同的重複器（例如價格方案內含功能清單）。每個示範項目都可帶有自己的示範子項目，它們會成為該項目的預設值。兩個層級都請加上 `data-label` 與 `data-repeater-item-label`，讓側邊欄清單顯示易讀的標題。

```html
<div data-repeater="plans" data-label="Plan" data-repeater-item-label="planTitle">
  <h3 data-bind="planTitle">Basic</h3>
  <ul>
    <li data-repeater="features" data-label="Feature" data-repeater-item-label="featureText" data-bind="featureText">1 website</li>
    <li data-repeater="features" data-label="Feature" data-repeater-item-label="featureText" data-bind="featureText">Email support</li>
  </ul>
</div>
```

**欄位編輯控制項。** 在「編輯全部項目」對話框中，綁定欄位預設是單行輸入。在 `data-bind` 旁加上 `data-bind-control="textarea"`（或 `text`）可為長文字選擇控制項：

```html
<p class="plan-desc" data-bind="planDesc" data-bind-control="textarea">For personal websites.</p>
```

## 4. 樹狀結構（階層式選單）

| 屬性 | 位置 | 效果 |
| :--- | :--- | :--- |
| `data-tree="key"` + `data-label` | 容器 | 樹狀屬性。 |
| `data-tree-item` | 每個項目 | 項目範本。 |
| `data-tree-children` | 子容器（或任何 `ul`／`ol`） | 子項目插槽。 |
| `data-tree-grid` | 容器 | 網格式樹狀結構。 |
| `data-tree-part="left\|right"` | 多個容器 | 將一棵樹分散到多個容器。 |
| `data-tree-slice="offset,length"` | 容器 | 只渲染項目的某個範圍。 |
| `data-tree-item-label="control"` | 容器 | 以該控制項的文字作為每個項目的標題。 |

沒有子項目的項目會移除 `has-dropdown` 與 `chevron-down`。請勿在樹狀結構內再放重複器。

## 5. 佔位符 `{{ … }}`

可用於文字與屬性值：`{{ key }}`、`{{ site.url }}`、`{{ context.title }}`、`{{ current_lang_name }}`，以及迴圈內的 `{{ item.title }}`。支援點號路徑。由側邊欄控制項建立的區塊屬性可以驅動 class 名稱，這是提供版面切換的常見做法：

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

## 6. 條件與迴圈

| 指令 | 用途 |
| :--- | :--- |
| `data-if="expr"`／`data-elseif="expr"`／`data-else` | 條件渲染（伺服器與編輯器皆適用）。 |
| `data-for="item in list"`／`data-for="(item, i) in list"` | 迭代情境陣列或屬性。 |
| `data-fe-if`、`data-fe-elseif`、`data-fe-else`、`data-fe-for` | 相同功能，但只在前台（訪客瀏覽器）計算；編輯器顯示示範內容。 |

運算式：真假值、`!x`、點號路徑、`==`、`!=`、`===`、`!==`。迴圈範圍：`index`、`is_first`、`is_last`（或你自訂的別名，例如 `crumb.is_last`）。可用 `<template data-for>` 作為透明的外層包裝。

## 7. 宣告式元件

| 屬性 | 行為 |
| :--- | :--- |
| `data-breadcrumb`（+ `data-separator`） | 容器會填入麵包屑路徑；第一個 `<a>`／`<span>` 作為樣式範本；分隔符號取自 `data-separator`，或從範本偵測。 |
| `data-lang-switcher` | 容器會為每個已設定的語言複製一份其第一個 `<a>`（含 `href`、`data-lang`、名稱／旗幟）。需要多語言模組。 |
| `data-form="id"`（+ `data-form-label`、`data-redirect`） | 標記由表單模組處理的表單（完整指南：[forms.md](forms.md)）。`data-form-label` = 項目與郵件中的易讀名稱；`data-redirect` = 成功後開啟的網址。欄位：任何 `input/textarea/select[name]` 或 `[data-form-field]`。前台指令碼也會派發 DOM 事件 `refinelab:form:success`（`detail.formId`、`detail.response`）。編輯器提供每個區塊的覆寫設定，以屬性 `form_<id>_{id,label,redirect,response,email}` 儲存。**保留：** `data-response` 與 `data-email` 會被解析、可在側邊欄編輯並寫回 HTML，但目前的前台指令碼與伺服器處理程式不會讀取它們（收件者與訊息來自表單設定）。 |

## 8. 側邊欄控制項

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

選擇器會在渲染時解析為完整的文章／詞彙物件，可用 `data-for` 迴圈。

已解析的**文章**項目欄位：`id`、`title`、`slug`、`type`、`url`（別名 `href`、`link`）、`excerpt`（別名 `desc`）、`date`（`Y.m.d`）、`date_formatted`（網站日期格式）、`author`（顯示名稱）、`category`／`category_name`（第一個分類）、`categories`（名稱清單）、`meta`（日期加分類標籤），文章有特色圖片時還有 `thumb` 與相關圖片鍵。已解析的**詞彙**項目欄位：`id`、`name`、`slug`、`taxonomy`、`count`、`url`（`href`、`link`）、`description`、`posts`（該詞彙的文章項目，依 `posts_bind`、`posts_per_page`、`orderby`、`order` 填入）。

`context="auto|always|off"` 決定是否以目前頁面作為來源（編輯者可用 `<bind>_context` 覆寫）。編輯器／REST 預覽中不會解析任何內容。

`<inner-blocks allowed-blocks="core/paragraph,theme/card" template-lock="false">` 宣告巢狀區塊。

## 9. 全域情境物件

每個範本都可使用（篩選器 `refinelab_html_blocks_global_attributes` 可修改此陣列）：

| 名稱 | 內容 |
| :--- | :--- |
| `site` | `name`、`description`、`url`、`admin_url`、`language`、`charset`、`theme` |
| `context` | `title`、`slug`、`url`、`type`、`post_type`、`id`、`is_front_page`、`is_home`、`is_single`、`is_page`、`is_archive`、`is_category`、`is_tag`、`is_tax`、`is_search`、`is_404`、`paged` |
| `post` | `id`、`title`、`slug`、`type`、`url`、`excerpt`、`date`、`thumbnail`、`author` |
| `posts` | 最近／查詢到的文章物件 |
| `term`、`terms` | 查詢到的詞彙；分類清單 |
| `languages` | 語言清單：`code`、`name`、`flag`、`url`、`is_current`、`label` |
| `current_language` | 目前語言物件 |
| `current_lang_code`、`current_lang_name`、`current_lang_flag`、`current_lang_label`、`current_lang_url` | 目前語言的扁平捷徑 |
| `breadcrumbs` | 麵包屑項目：`title`、`url`、`is_current`、`is_last`（篩選器 `refinelab_html_blocks_breadcrumb_list`） |

## 10. 多檔案區塊（`blocks/<slug>/`）

| 檔案 | 載入時機 | 用途 |
| :--- | :--- | :--- |
| `index.html` | 一律 | 範本（語法與單檔案區塊相同）。 |
| `index.css` | 編輯器 + 前台 | 共用樣式。 |
| `index.web.css`、`index.web.js` | 只在前台，且只在使用該區塊的頁面 | 前台樣式與行為。 |
| `index.editor.css`、`index.editor.js` | 只在編輯器 | 編輯器專用樣式；生命週期掛鉤（見 §13）。 |
| `index.php` | 伺服器渲染 | 以你自己的 PHP 取代範本渲染。可用變數：`$attributes`（區塊屬性）、`$content`（已渲染的 InnerBlocks）、`$block`（`WP_Block`）、`$block_def`（解析後的區塊結構）、`$renderer`（`RefineLab_HTML_Blocks`）。根元素請用 `get_block_wrapper_attributes()`，並對輸出的所有內容進行跳脫。 |

連結屬性儲存為 `<key>`（文字）與 `<key>_url`；圖片為 `<key>`、`<key>_url`、`<key>_alt`。完整範例隨外掛附在 `templates/sample-blocks/interactive-box/`（Refine Lab → 區塊 → 安裝範例區塊）。

## 11. 注意事項

- 圖示：`<i data-lucide="name">`；佈景主題只應取代 `[data-lucide]:not(svg)`，且不要在每次 DOM 變動時重新執行 `lucide.createIcons()`。
- 一般的 `href` 在編輯器中會停用；請用 `data-bind` 綁定連結。
- 佈景主題的樣式應放在佈景主題中，編輯器樣式請限定在 `.editor-styles-wrapper` 範圍內。
- 裝飾性圖片：在 `<img>`（或其祖先元素）加上 `data-bg="true"` 或 `data-editor-interactive="false"`，可讓它在編輯器中不被綁定到媒體庫，適用於背景或不可互動的圖片。
- 圖示：`data-lucide` 與 `data-feather` 都會被保留為圖示標記。
- 快取卡住了？修改檔案，或呼叫 `admin_post_refinelab_clear_blocks_cache`（Refine Lab → 區塊 → 清除快取）。

## 12. 編輯器增強（Refine Lab → 編輯器）

- **固定文章標題**：在編輯器中固定顯示並模糊背景的標題（`enable_sticky_post_title`，預設開）。
- **可調整寬度的側邊欄**：拖曳編輯器側邊欄的左緣調整寬度；按兩下重設為 280 px；寬度存於 `localStorage['refinelab_sidebar_width']`（`enable_resizable_sidebar`，預設開）。關閉時不會加上 body class `refinelab-resizable-sidebar`，範圍限定的 CSS 不會套用，JS 把手也不會建立。需要啟用 HTML 區塊模組（編輯器套件）。

## 13. 用戶端生命週期（JavaScript）

用 `window.refineLab.registerBlock( slugOrName, hooks )` 為區塊註冊行為。`slugOrName` 可為 `hero` 或 `theme/hero`。

```js
(function (refineLab) {
  refineLab.registerBlock('hero', {
    // 前台與編輯器：每個區塊元素就緒時呼叫一次。
    init: function (context) { /* context.el, context.name, context.slug */ },

    // 僅編輯器
    insert:    function (context) { /* 區塊剛被加入畫布 */ },
    update:    function (context) { /* context.attributes, context.prevAttributes */ },
    destroyed: function (context) { /* context.name, context.clientId */ }
  });
})(window.refineLab = window.refineLab || {});
```

| 位置 | 情境 | 說明 |
| :--- | :--- | :--- |
| 前台（`index.web.js` 或任何佈景主題指令碼） | `{ el, name, slug, isEditor: false }` | 在 `DOMContentLoaded` 執行；若 DOM 已就緒則立即執行；每個元素只初始化一次（`data-refinelab-initialized`）。根元素以 `.wp-block-theme-<slug>`、`.wp-block-<slug>` 或 `[data-block]` 比對。 |
| 編輯器（`index.editor.js`） | `{ el, name, clientId, attributes, prevAttributes, … }` | 掛鉤會對每個區塊實例執行。 |

相同的事件也會提供給其他程式碼：

- 區塊根元素上的 DOM 事件：`refinelab:init`（前台）、`refinelab:insert`、`refinelab:init`、`refinelab:update`、`refinelab:destroyed`（編輯器），皆會冒泡，情境放在 `event.detail`。
- 編輯器的 `wp.hooks` 動作：`refinelab.block.<event>`、`refinelab.block.<slug>.<event>` 與 `refinelab.block.theme.<slug>.<event>`，以 `( context, blockDef )` 呼叫。

```js
wp.hooks.addAction('refinelab.block.hero.update', 'my-theme', function (context) {
  console.log('Hero changed', context.attributes);
});
document.addEventListener('refinelab:init', function (e) { console.log(e.detail.slug); });
```

輔助函式：`refineLab.getBlockHooks( slugOrName )` 回傳已註冊的掛鉤物件。

## 14. 閱讀範例區塊

`templates/sample-blocks/` 內含可直接執行的範本，展示每項功能（見 [html-blocks-cookbook.md](html-blocks-cookbook.md)）。
