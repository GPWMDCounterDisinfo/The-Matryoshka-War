# Embedding this page (WordPress mobile fix)

This page is embedded at https://gpwmdcounterdisinfo.com/the-matryoshka-model/
inside an `<iframe>` with a fixed height (currently `height="800px"`). A fixed
height causes a **nested scrollbar** on mobile: the visitor scrolls the
WordPress page, then their scroll gets trapped inside the small iframe box
before they can continue scrolling the rest of the page.

As of this change, the page itself reports its real content height to its
parent window via `postMessage` (see the "IFRAME AUTO-HEIGHT" block at the
bottom of `script.js`). It does nothing when the page isn't embedded, so it's
safe either way.

To make WordPress actually use that height, add a small listener script to
the **same page** the iframe is embedded on (e.g. right after the iframe in
the same "Custom HTML" block, or via a header/footer code-injection plugin
scoped to that page):

```html
<script>
(function () {
  // Match this to however the iframe is identified on the page —
  // an id is most reliable if you can add one to the embed code.
  var IFRAME_SELECTOR = 'iframe[src*="gpwmdcounterdisinfo.github.io/The-Matryoshka-War"]';

  window.addEventListener('message', function (e) {
    if (!e.data || e.data.type !== 'matryoshka:resize') return;
    var iframe = document.querySelector(IFRAME_SELECTOR);
    if (iframe) iframe.style.height = e.data.height + 'px';
  });
})();
</script>
```

Notes:
- Keep the iframe's `height="800"` (or similar) attribute as a fallback in
  case the listener script isn't present or JS is blocked — the page will
  still work, just with the old nested-scroll behavior in that case.
- The message is sent with `postMessage(..., '*')` so it works regardless of
  which domain embeds the page. The listener only acts on messages shaped
  like `{ type: 'matryoshka:resize', height: <number> }`, so it's safe to
  add without extra origin checks.
- If you'd rather not touch WordPress code at all, a lower-effort partial
  fix is to just raise the fixed `height` attribute (e.g. to `1000`) — this
  page's mobile layout is close to a single constant height (no long
  in-flow content; case detail and section modals are full-screen overlays
  that don't add to page height), so a generous static value mostly avoids
  the nested-scroll issue without any script changes. It's just not
  future-proof the way the postMessage-based resize is.
