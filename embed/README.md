# Embedding KTV CTAs

CTAs in `ctas/` are served from GitHub Pages at **`https://cta.ktvdigital.com`**
and embedded in ktvdigital.com blog posts as iframes inside a
`.ktv-iframe-wrapper` box. The box has a fixed height per screen size, set in
Shopify. Each CTA must fit inside that box without a scrollbar.

Always embed with `cta.ktvdigital.com` URLs. Older posts that use
`kevdogtx.github.io/KTV-Marketing-Assets/...` still work, because GitHub
redirects them to the custom domain, but update them when you edit those posts.

The domain comes from the `CNAME` file at the repo root plus a DNS record at
Directnic (`cta` → CNAME → `kevdogtx.github.io`). Don't delete, rename or edit
either one, or every embedded CTA stops loading. Don't rename or move files in
`ctas/` either, since each file's URL is embedded in live posts.

Setup has two parts, and they go in **two different places**:

| What | Where it goes |
|---|---|
| 1. CSS | EComposer → **Custom CSS** (shared by every blog post) |
| 2. Embed code | The blog article's HTML view (`<>` button), once per post |

## 1. CSS (EComposer → Custom CSS)

All three rules are required. If the last rule is missing, the iframe shrinks to
the browser default of 300×150px inside an empty box.

```css
.ktv-iframe-wrapper {
  position: relative;
  height: 530px;
  overflow: hidden;
  border-radius: 8px;
  margin: 40px 0;
}

@media (max-width: 600px) {
  .ktv-iframe-wrapper { height: 600px; }
}

@media (max-width: 420px) {
  .ktv-iframe-wrapper { height: 700px; }
}

.ktv-iframe-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 0;
}
```

## 2. Embed code (article HTML, per post)

Paste this where the CTA should appear. Never put CSS inside this `<div>`.

```html
<div class="ktv-iframe-wrapper"><iframe src="https://cta.ktvdigital.com/ctas/search-score-tool.html" loading="lazy" title="AI SearchScore: Free AI Visibility Score"></iframe></div>
```

## Box heights (what each CTA must fit)

The box height depends on the browser window width. The iframe is about 40px
narrower than the window on phones and tablets, and caps at about 812px wide on
desktop.

| Window width | Box height |
|---|---|
| 601px and wider | 530px |
| 421–600px | 600px |
| 420px and narrower | 700px |

`search-score-tool.html` has been checked at every iframe width from 260px to
1000px. Its tallest fit leaves 23px to spare. If you change a CTA's copy or
layout, check it again at those widths, especially 280px, 560px and 728px, which
are the tightest.

## Slim mid-article CTA (`search-score-inline.html`)

A short strip version of the AI SearchScore CTA, meant to sit about 40% of the way
into a post. It needs a shorter box, so it adds a second class,
`ktv-iframe-slim`, to the same wrapper. All the Part 1 CSS still applies; the
rules below only change the height and spacing.

Add this to EComposer → **Custom CSS**, below the Part 1 rules:

```css
.ktv-iframe-wrapper.ktv-iframe-slim {
  height: 190px;
  margin: 32px 0;
}

@media (max-width: 600px) {
  .ktv-iframe-wrapper.ktv-iframe-slim { height: 220px; }
}

@media (max-width: 520px) {
  .ktv-iframe-wrapper.ktv-iframe-slim { height: 250px; }
}
```

Embed code (paste between two paragraphs, about 40% of the way down the post):

```html
<div class="ktv-iframe-wrapper ktv-iframe-slim"><iframe src="https://cta.ktvdigital.com/ctas/search-score-inline.html" loading="lazy" title="AI SearchScore: Free AI Visibility Score"></iframe></div>
```

| Window width | Box height | Tallest CTA | Spare |
|---|---|---|---|
| 601px and wider | 190px | 164px | 26px |
| 521–600px | 220px | 197px | 23px |
| 520px and narrower | 250px | 228px | 22px |

These were checked at every iframe width from 260px to 1000px. The layout sits
side by side above 520px iframe width and stacks below it. Its link uses
`utm_campaign=search-score-cta-inline`, so its clicks can be told apart from
the end-of-post CTA.

## Troubleshooting

| Symptom | Cause | Fix |
|---|---|---|
| Scrollbar inside the CTA | Box shorter than the CTA at that width | Check the CSS above is in Custom CSS; tighten the CTA's layout at that width |
| Small 300×150 CTA in a big empty box | `.ktv-iframe-wrapper iframe` rule missing | Add the last rule from Part 1 back to Custom CSS |
| CSS text shows on the page where the CTA should be | CSS was pasted into the article instead of Custom CSS | Replace the article's wrapper with the Part 2 embed code |
| Text and an extra CTA appear above the site header | Non-JavaScript content in EComposer Custom JS | Empty the Custom JS field |

## Optional: auto-height (not enabled)

Each CTA also reports its own height to the page it is embedded in, so the box
could resize to fit exactly instead of using fixed heights. This does nothing
unless the page has a listener, and right now ktvdigital.com does not.

To turn it on, paste the code below into EComposer → **Custom JS**. Paste only
this code block: no `<script>` tags, no headings, and none of this README.
EComposer adds its own `<script>` tag around the field, and any other text
breaks every blog page.

```js
(function () {
  var ORIGIN = 'https://cta.ktvdigital.com';

  function frames() {
    return document.querySelectorAll('.ktv-iframe-wrapper iframe');
  }

  window.addEventListener('message', function (e) {
    if (e.origin !== ORIGIN || !e.data || e.data.type !== 'ktv-cta-height') return;
    var h = Number(e.data.height);
    if (!isFinite(h) || h < 150 || h > 3000) return;
    Array.prototype.forEach.call(frames(), function (f) {
      if (f.contentWindow === e.source) {
        f.parentElement.style.height = Math.ceil(h) + 'px';
      }
    });
  });

  function ask() {
    Array.prototype.forEach.call(frames(), function (f) {
      try { f.contentWindow.postMessage({ type: 'ktv-cta-height-request' }, ORIGIN); } catch (err) {}
    });
  }
  if (document.readyState === 'loading') {
    document.addEventListener('DOMContentLoaded', ask);
  } else {
    ask();
  }
})();
```

After saving, open a blog post and confirm nothing appears above the header. If
anything does, empty the Custom JS field right away.

The listener only accepts messages from `https://cta.ktvdigital.com` and only
resizes the iframe that sent the message. To make another CTA report its height,
copy the auto-height `<script>` block from the bottom of
`ctas/search-score-tool.html` into it.
