# liquid-tab-bar

A bottom navigation bar whose surface **sags under a finger-following ball** — like a mattress under a bowling ball. The ball tracks your pointer, dents the bar downward as it goes, and settles onto the tab you release over. No images, no SVG goo-blur hacks, no framework.

**[▶ Live demo](https://everyoneneedsasamwise.github.io/liquid-tab-bar/)** · zero dependencies · ~4KB · MIT

![liquid-tab-bar demo](demo.gif)

---

## Why the physics are real

The bar is an SVG `<path>` recomputed every frame. The surface height at any x is a **Gaussian dip** centred on the ball:

```
edgeY(x) = topline + DIP·e^(-(x-cx)²/2σ²) − LIP·e^(-(x-cx)²/2σ_lip²)
```

A blur filter (the usual "gooey" trick) can only make a shape bulge **outward** — it can't press the surface **inward**, which is what a weight resting on a membrane actually does. So the dent is drawn, not filtered. The ball eases toward your pointer with a `requestAnimationFrame` lerp, which gives it that heavy, liquid lag.

## Install

**Drop-in (no build step):** copy `liquid-tab-bar.js` and `liquid-tab-bar.css` into your project.

**npm:**

```bash
npm install liquid-tab-bar
```

## Use it

Write plain, accessible links and let the script enhance them in place:

```html
<nav class="liquid-tab-bar" data-liquid-tab-bar>
  <a href="#home"    data-icon="home">Home</a>
  <a href="#explore" data-icon="compass">Explore</a>
  <a href="#create"  data-icon="grid">Create</a>
  <a href="#saved"   data-icon="heart">Saved</a>
  <a href="#profile" data-icon="user">Profile</a>
</nav>

<link rel="stylesheet" href="liquid-tab-bar.css" />
<script src="liquid-tab-bar.js"></script>
```

Any element with `data-liquid-tab-bar` auto-initialises on load. Also add the clearance so your page content isn't hidden behind the fixed bar:

```html
<body class="has-liquid-tab-bar">  <!-- or your own padding-bottom: ~96px -->
```

### Navigation modes

- **Hash links** (`href="#home"`) → single-page mode. Clicks and drag-releases update `location.hash`; the active tab follows `hashchange`. Nothing reloads.
- **Real paths/URLs** (`href="/about"`) → the active tab is matched from `location.pathname` on load, and navigating a tab loads that page normally.

### Control it from JavaScript (optional)

```js
const bar = LiquidTabBar.mount(".liquid-tab-bar", {
  active: 0,                       // starting tab index
  onNavigate(i, tab, { viaDrag }) {
    // return false to take over navigation yourself (e.g. a SPA router)
    router.push(tab.href);
    return false;
  },
  onActiveChange(i, tab) { /* ... */ },
});

bar.setActive(2);   // move the ball programmatically
bar.destroy();      // remove listeners
```

You can also skip the markup and pass tabs directly:

```js
LiquidTabBar.mount("#nav", {
  tabs: [
    { href: "/",        label: "Home",    icon: "home",    color: "#e09f5c" },
    { href: "/projects", label: "Work",   icon: "grid",    color: "#8aa0e8" },
  ],
});
```

## Theming

Every colour is a CSS custom property on `.liquid-tab-bar`. Override them in your own stylesheet:

```css
.liquid-tab-bar {
  --ltb-max-width: 520px;
  --ltb-bar-fill: #161c2b;
  --ltb-bar-stroke: rgba(238, 189, 138, 0.18);
  --ltb-label: #66728b;          /* inactive icon + label */
  --ltb-label-active: #0b0f1a;   /* active icon on the ball */
  --ltb-ball-highlight: rgba(255, 255, 255, 0.18);
  --ltb-lift: -28px;             /* how high the active icon rides the ball */
}
```

Per-tab accent colour comes from `data-color="#rrggbb"` on each link (otherwise a built-in palette cycles). For a light bar, add `data-theme="light"` to the `<nav>`.

## Icons

14 icons are built in: `home`, `grid` (`work`), `user` (`about`), `mail` (`contact`), `search`, `settings`, `heart`, `bell`, `compass`, `play`, `star`, `chat`. Reference one with `data-icon="name"`.

Bring your own by putting an inline `<svg>` inside the link (it's drawn in a `0 0 24 24` box, stroked with `currentColor`):

```html
<a href="#more" aria-label="More">
  <svg viewBox="0 0 24 24"><circle cx="5" cy="12" r="1.6"/><circle cx="12" cy="12" r="1.6"/><circle cx="19" cy="12" r="1.6"/></svg>
  More
</a>
```

> **Note:** built-in and custom icon markup is treated as author-trusted HTML (it's your own SVG, set via `innerHTML`). Don't feed it untrusted, user-supplied strings.

## Accessibility

- Renders real `<a>` links, so it works without JS and reads correctly to screen readers.
- Sets `aria-current="page"` on the active tab and shows a visible `:focus-visible` ring.
- Honours `prefers-reduced-motion`: the ball jumps to the target tab instead of easing.

## Browser support

Any browser with [Pointer Events](https://caniuse.com/pointer) — all current Chrome, Safari, Firefox and Edge. Mouse, touch and pen all work.

## License

[MIT](LICENSE) © Everyone Needs a Samwise
