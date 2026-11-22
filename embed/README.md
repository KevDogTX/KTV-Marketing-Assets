# Embedding KTV CTAs

CTAs in `ctas/` are served from GitHub Pages and embedded in ktvdigital.com blog
posts as iframes. Each CTA reports its own height to the parent page, so the
iframe grows or shrinks to fit the content at any screen width, with no inner
scrollbar.

Setup has three parts. Parts 1 and 2 go in EComposer's **custom CSS / custom JS**
for the blog template (they apply to every post). Part 3 goes in each post.

## 1. CSS (fallback height)

Used before the CTA reports its height, or if JavaScript is blocked.

```css
.ktv-iframe-wrapper {
  position: relative;
  height: 530px;
  overflow: hidden;
  border-radius: 8px;
  margin: 40px 0;
}

.ktv-iframe-wrapper iframe {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  border: 0;
}

@media (max-width: 600px) {
  .ktv-iframe-wrapper { height: 600px; }
}

@media (max-width: 420px) {
  .ktv-iframe-wrapper { height: 700px; }
}
```

## 2. JavaScript (auto-height listener)

If the field expects plain JavaScript, paste only what is between the
`<script>` tags.

```html
<script>
(function () {
  var ORIGIN = 'https://kevdogtx.github.io';

  function frames() {
    return document.querySelectorAll('.ktv-iframe-wrapper iframe');
  }

  window.addEventListener('message', function (e) {
    if (e.origin !== ORIGIN || !e.data || e.data.type !== 'ktv-cta-height') return;
    var h = Number(e.data.height);
    if (!isFinite(h) || h < 150 || h > 3000) return;
    Array.prototype.forEach.call(frames(), function (f) {
      if (f.contentWindow === e.source) {
        var wrapper = f.parentElement;
        wrapper.style.paddingBottom = '0';
        wrapper.style.height = Math.ceil(h) + 'px';
      }
    });
  });

  // Ask CTAs that loaded before this listener was ready to report again.
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
</script>
```

## 3. Embed code (per post)

```html
<div class="ktv-iframe-wrapper">
  <iframe src="https://kevdogtx.github.io/KTV-Marketing-Assets/ctas/search-score-tool.html"
          loading="lazy" title="AI Visibility SearchScore Tool"></iframe>
</div>
```

## Notes

- The listener only accepts messages from `https://kevdogtx.github.io` and only
  resizes the iframe that sent the message.
- It also sets `padding-bottom: 0` on the wrapper, so it works even if the old
  16:9 CSS (`padding-bottom: 56.25%`) is still in place.
- Any CTA can opt in by including the same auto-height `<script>` block found at
  the bottom of `ctas/search-score-tool.html`.
