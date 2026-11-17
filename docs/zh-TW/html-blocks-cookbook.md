# HTML 區塊範例集

[English](../html-blocks-cookbook.md)

可直接使用的做法，每一個都是完整範本，放進 `<theme>/blocks/<slug>.html` 即可。語法說明見 [html-blocks.md](html-blocks.md)；下列檔案也隨外掛附在 `templates/sample-blocks/`（Refine Lab → 區塊 → 安裝範例區塊）。

## 1. 最小可用的區塊

```html
<div class="card" data-title="Feature Card" data-icon="star-filled">
  <h3 data-bind="title">Lightning fast</h3>
  <p data-bind="text">Describe the feature in a sentence.</p>
</div>
```

結果：插入器中出現名為 *Feature Card* 的區塊。標題與段落可直接在畫布上編輯；示範文字就是預設內容。儲存的屬性：`title`、`title_align`、`title_tag`、`text`、`text_align`。

## 2. 含圖片、選項與巢狀區塊的主視覺

```html
<section class="hero is-align-{{alignStyle}}" data-title="Hero Banner" data-category="theme" data-icon="cover-image">
  <inspector-controls>
    <panel-body title="Hero Settings">
      <select-control data-bind="alignStyle" label="Alignment">
        <select-control-option value="center">Center</select-control-option>
        <select-control-option value="left">Left</select-control-option>
      </select-control>
      <toggle-control data-bind="showCta" label="Display call to action"></toggle-control>
    </panel-body>
  </inspector-controls>

  <h1 data-bind="heroTitle">Supercharge your site</h1>
  <p data-bind="heroSubtitle">Build blocks from plain HTML.</p>
  <img data-bind="heroImage" src="https://example.com/sample.jpg" alt="Hero" />

  <inner-blocks allowedBlocks="core/buttons,core/paragraph,core/heading"></inner-blocks>
</section>
```

- class 中的 `{{alignStyle}}` 把下拉選單變成版面切換（`is-align-center`、`is-align-left`）。
- `<img data-bind>` 會開啟媒體庫；網址、替代文字與 `srcset` 都會儲存。
- `<inner-blocks>` 讓編輯者可以在裡面放入清單中的任何區塊。

只在開關開啟時才顯示某個部分：

```html
<a data-if="showCta" class="btn" data-bind="ctaLabel">Get started</a>
```

## 3. 重複器：客戶見證、標誌牆、團隊

```html
<section class="team" data-title="Team">
  <div class="team__member" data-repeater="people" data-label="Person" data-repeater-item-label="name">
    <img data-bind="photo" src="https://example.com/a.jpg" alt="" />
    <h3 data-bind="name">Sarah Jenkins</h3>
    <p data-bind="role">Lead architect</p>
  </div>
  <div class="team__member" data-repeater="people" data-label="Person" data-repeater-item-label="name">
    <img data-bind="photo" src="https://example.com/b.jpg" alt="" />
    <h3 data-bind="name">Minh Tran</h3>
    <p data-bind="role">Designer</p>
  </div>
</section>
```

兩個示範元素會成為兩個預設項目。編輯者可以在畫布上或「編輯全部」對話框中移動、複製、刪除與新增項目。`data-repeater-item-label="name"` 讓每一列以人名作為標題。

## 4. 含巢狀重複器的價格表

```html
<section class="pricing" data-title="Pricing Table" data-icon="money-alt">
  <inspector-controls>
    <panel-body title="Layout" initialOpen="true">
      <select-control data-bind="columns" label="Columns">
        <select-control-option value="3">3 columns</select-control-option>
        <select-control-option value="2">2 columns</select-control-option>
      </select-control>
    </panel-body>
  </inspector-controls>

  <div class="pricing__grid columns-{{columns}}">
    <div class="plan" data-repeater="plans" data-label="Plan" data-repeater-item-label="planTitle">
      <h3 data-bind="planTitle">Basic</h3>
      <p data-bind="planDesc" data-bind-control="textarea">For small projects.</p>
      <strong data-bind="planPrice">19</strong><span data-bind="planPeriod">/mo</span>
      <ul>
        <li data-repeater="features" data-label="Feature" data-repeater-item-label="featureText" data-bind="featureText">1 website</li>
        <li data-repeater="features" data-label="Feature" data-repeater-item-label="featureText" data-bind="featureText">Standard support</li>
      </ul>
      <a href="#" data-bind="planCta">Choose Basic</a>
    </div>
  </div>
</section>
```

## 5. 導覽樹（含子選單的選單）

```html
<nav class="menu" data-title="Navigation Tree Menu" data-icon="menu">
  <ul data-tree="navItems" data-label="Menu item" data-tree-item-label="menuLink">
    <li data-tree-item><a href="#" data-bind="menuLink">Home</a></li>
    <li data-tree-item>
      <a href="#" data-bind="menuLink">Services</a>
      <ul data-tree-children>
        <li data-tree-item><a href="#" data-bind="menuLink">Web design</a></li>
        <li data-tree-item><a href="#" data-bind="menuLink">Development</a></li>
      </ul>
    </li>
  </ul>
</nav>
```

編輯者以拖放建立階層（側邊欄「在視窗中管理」）；層數不限。用 `data-tree-grid` 取得卡片／網格版面，用 `data-tree-part="left|right"` 把一棵樹分到兩個容器（兩欄的大型選單），用 `data-tree-slice="0,3"` 只渲染前三個項目。

## 6. 從網站取得內容

在側邊欄選擇文章，再對它們迴圈：

```html
<section class="news" data-title="Latest news">
  <inspector-controls>
    <panel-body title="Source" initialOpen="true">
      <post-picker data-bind="articles" post-type="post" multiple="true" max_items="3" label="Articles"></post-picker>
    </panel-body>
  </inspector-controls>

  <article data-for="item in articles">
    <a href="{{ item.url }}"><h3>{{ item.title }}</h3></a>
    <p>{{ item.excerpt }}</p>
  </article>
</section>
```

或由詞彙決定要顯示哪些文章：

```html
<term-picker data-bind="topic" taxonomy="category" posts_bind="topic_posts" posts_per_page="6" orderby="date" order="DESC" context="auto"></term-picker>
<div data-for="post in topic_posts"><a href="{{ post.url }}">{{ post.title }}</a></div>
```

選擇器在頁面渲染時於伺服器端解析；編輯器則顯示範本的示範標記。

## 7. 條件與站點資料迴圈

```html
<!-- 麵包屑路徑 -->
<nav data-breadcrumb data-separator="/">
  <a href="#">Home</a><span>Current page</span>
</nav>

<!-- 語言連結 -->
<ul data-lang-switcher>
  <li><a href="#" data-lang="en">English</a></li>
</ul>

<!-- 依頁面類型顯示不同標記 -->
<h1 data-if="context.is_front_page">Welcome to {{ site.name }}</h1>
<h1 data-elseif="context.is_single">{{ post.title }}</h1>
<h1 data-else>{{ context.title }}</h1>

<!-- 含 first／last 輔助的自訂迴圈 -->
<template data-for="(crumb, i) in breadcrumbs">
  <a data-if="!crumb.is_last" href="{{ crumb.url }}">{{ crumb.title }}</a>
  <span data-else>{{ crumb.title }}</span>
</template>
```

`data-fe-if`／`data-fe-for` 做同樣的事，但在訪客的瀏覽器中計算（編輯器保留示範內容）。

## 8. 不需外掛程式碼的表單

```html
<form class="contact" data-form="contact" data-form-label="Contact form" data-redirect="/thank-you/">
  <input data-form-field="name" type="text" placeholder="Your name" required />
  <input data-form-field="email" type="email" placeholder="Email" required />
  <textarea data-form-field="message" placeholder="How can we help?"></textarea>
  <button type="submit">Send</button>
</form>
```

提交內容會儲存為項目（Entries）並以郵件通知；見 [forms.md](forms.md)。

## 9. 含行為的多檔案區塊

資料夾 `blocks/interactive-box/`：

```
index.html        含 data-bind 控制項的範本
index.css         共用樣式
index.web.js      前台行為
index.editor.js   編輯器掛鉤
index.php         選用的伺服器端渲染
```

`index.web.js`：

```js
(function (refineLab) {
  refineLab.registerBlock('interactive-box', {
    init: function (context) {
      var btn = context.el.querySelector('.interactive-box-toggle');
      var box = context.el.querySelector('.interactive-box-details');
      btn.addEventListener('click', function () {
        var open = box.classList.toggle('is-open');
        btn.setAttribute('aria-expanded', open ? 'true' : 'false');
      });
    }
  });
})(window.refineLab = window.refineLab || {});
```

`index.php`（取代範本渲染）：

```php
<?php
/** @var array $attributes @var string $content @var WP_Block $block */
$title = ! empty( $attributes['title'] ) ? $attributes['title'] : 'Hello';
$wrapper = get_block_wrapper_attributes( array( 'class' => 'interactive-box-wrapper' ) );
?>
<div <?php echo $wrapper; ?>>
  <h3><?php echo esc_html( $title ); ?></h3>
  <?php echo $content; // 已渲染的 InnerBlocks。 ?>
</div>
```

## 10. 常見錯誤

| 症狀 | 原因 | 解法 |
| :--- | :--- | :--- |
| 重新命名欄位後內容消失 | `data-bind` 的 key 就是儲存用的鍵 | 頁面已使用該區塊後絕不要重新命名 key；請新增一個新的 key。 |
| 區塊沒有出現在插入器 | 檔案不在掃描的資料夾，或快取過期 | 放進 `blocks/`、`template-parts/blocks/` 或 `html-blocks/`；Refine Lab → 區塊 → 清除快取。 |
| 編輯器中連結無法點擊 | 畫布上一般的 `href` 會停用 | 以 `data-bind` 綁定連結，並在連結彈出視窗中編輯。 |
| 編輯後圖示消失 | 有指令碼在每次 DOM 變動時重新執行圖示函式庫 | 只執行一次對 `[data-lucide]:not(svg)` 的取代，或改用 CSS 圖示。 |
| 樣式影響到編輯器 | 佈景主題 CSS 是全域的 | 將編輯器規則限定在 `.editor-styles-wrapper`。 |
