# HTML blocks cookbook

**Language:** English · [繁體中文](zh-TW/html-blocks-cookbook.md)

Working patterns, each one a complete template you can drop into `<theme>/blocks/<slug>.html`. The syntax is explained in [html-blocks.md](html-blocks.md); the files below also ship with the plugin in `templates/sample-blocks/` (Refine Lab → Blocks → install sample blocks).

## 1. Smallest useful block

```html
<div class="card" data-title="Feature Card" data-icon="star-filled">
  <h3 data-bind="title">Lightning fast</h3>
  <p data-bind="text">Describe the feature in a sentence.</p>
</div>
```

Result: a block called *Feature Card* in the inserter. The heading and the paragraph are editable on the canvas; the sample text is the default content. Stored attributes: `title`, `title_align`, `title_tag`, `text`, `text_align`.

## 2. Hero with image, options and nested blocks

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

- `{{alignStyle}}` in the class turns the select into a layout switch (`is-align-center`, `is-align-left`).
- `<img data-bind>` opens the Media Library; the URL, alt text and `srcset` are stored.
- `<inner-blocks>` lets editors drop any of the listed blocks inside.

Show a part only when a toggle is on:

```html
<a data-if="showCta" class="btn" data-bind="ctaLabel">Get started</a>
```

## 3. Repeater: testimonials, logos, team

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

Two sample elements become two default items. Editors can move, duplicate, delete and add items on the canvas or in the "Edit all" dialog. `data-repeater-item-label="name"` titles each row by the person's name.

## 4. Pricing table with nested repeaters

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

## 5. Navigation tree (menus with sub-menus)

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

Editors build the hierarchy by drag and drop (sidebar "Manage in modal"); depth is unlimited. Use `data-tree-grid` for a card/grid layout, `data-tree-part="left|right"` to split one tree across two containers (a mega menu with two columns) and `data-tree-slice="0,3"` to render only the first three items.

## 6. Pull content from the site

Choose posts in the sidebar and loop over them:

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

Or let a term decide the posts:

```html
<term-picker data-bind="topic" taxonomy="category" posts_bind="topic_posts" posts_per_page="6" orderby="date" order="DESC" context="auto"></term-picker>
<div data-for="post in topic_posts"><a href="{{ post.url }}">{{ post.title }}</a></div>
```

Pickers are resolved on the server when the page renders; the editor shows the template's sample markup.

## 7. Conditions and loops over site data

```html
<!-- Breadcrumb trail -->
<nav data-breadcrumb data-separator="/">
  <a href="#">Home</a><span>Current page</span>
</nav>

<!-- Language links -->
<ul data-lang-switcher>
  <li><a href="#" data-lang="en">English</a></li>
</ul>

<!-- Different markup per page type -->
<h1 data-if="context.is_front_page">Welcome to {{ site.name }}</h1>
<h1 data-elseif="context.is_single">{{ post.title }}</h1>
<h1 data-else>{{ context.title }}</h1>

<!-- Custom loop with first/last helpers -->
<template data-for="(crumb, i) in breadcrumbs">
  <a data-if="!crumb.is_last" href="{{ crumb.url }}">{{ crumb.title }}</a>
  <span data-else>{{ crumb.title }}</span>
</template>
```

`data-fe-if` / `data-fe-for` do the same but are evaluated in the visitor's browser (the editor keeps the sample).

## 8. A form with no plugin code

```html
<form class="contact" data-form="contact" data-form-label="Contact form" data-redirect="/thank-you/">
  <input data-form-field="name" type="text" placeholder="Your name" required />
  <input data-form-field="email" type="email" placeholder="Email" required />
  <textarea data-form-field="message" placeholder="How can we help?"></textarea>
  <button type="submit">Send</button>
</form>
```

Submissions are stored as entries and e-mailed; see [forms.md](forms.md).

## 9. Multi-file block with behaviour

Folder `blocks/interactive-box/`:

```
index.html        template with data-bind controls
index.css         shared styles
index.web.js      front-end behaviour
index.editor.js   editor hooks
index.php         optional server-side render
```

`index.web.js`:

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

`index.php` (replaces template rendering):

```php
<?php
/** @var array $attributes @var string $content @var WP_Block $block */
$title = ! empty( $attributes['title'] ) ? $attributes['title'] : 'Hello';
$wrapper = get_block_wrapper_attributes( array( 'class' => 'interactive-box-wrapper' ) );
?>
<div <?php echo $wrapper; ?>>
  <h3><?php echo esc_html( $title ); ?></h3>
  <?php echo $content; // Rendered InnerBlocks. ?>
</div>
```

## 10. Common mistakes

| Symptom | Cause | Fix |
| :--- | :--- | :--- |
| Content disappears after renaming a field | `data-bind` keys are the storage keys | Never rename a key once pages use the block; add a new key instead. |
| Block missing from the inserter | File not in a scanned folder, or stale cache | Put it in `blocks/`, `template-parts/blocks/` or `html-blocks/`; Refine Lab → Blocks → Clear cache. |
| Links cannot be clicked in the editor | Plain `href` is disabled on the canvas | Bind the link with `data-bind` and edit it in the link popover. |
| Icons vanish after editing | A script re-runs an icon library on every DOM change | Replace only `[data-lucide]:not(svg)` once, or use CSS icons. |
| Styling leaks into the editor | Theme CSS is global | Scope editor rules to `.editor-styles-wrapper`. |
