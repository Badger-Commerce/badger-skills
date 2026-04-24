---
name: animation-scene
description: Build and edit Badger Commerce animation scenes — SaaS-style GSAP-powered reveals, mock-browser product demos and scroll-scrub effects, composed declaratively from a JSON spec with server-resolved bindings to products, collections and site config
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Animation Scene Extension

The `animationScene` extension renders a declarative GSAP animation on the page. Authors describe a canvas, a set of elements, a timeline of tweens, and (optionally) bindings to live commerce data. The server resolves bindings, expands repeats, substitutes tokens, and emits a flat spec that a tiny runtime turns into a GSAP timeline.

**This skill contains everything you need — do not call `getExtensionConfigSchema` for animationScene.**

## Stay on-brand: load the design brief first

**Before generating any scene, call `getDesignBrief` via MCP.** The returned object contains the tenant's palette (`palette.primary/secondary/accent`), voice, typography and shape language. Use the palette hex values in the scene instead of any colours you see in the examples below — **the example colours in this skill are illustrative only**. If the tenant has no brief yet, `getDesignBrief` returns sensible defaults and the scene still renders. Match the `voice.tone` when writing any copy inside the scene.

If the brief has `palette.gradient` set, the CSS compiler exposes it as `--gradient-brand` — use `background: var(--gradient-brand)` for canvas backgrounds or hero blocks instead of crafting your own `linear-gradient(...)`. Dark-leaning brands may also provide `palette.surface` + `palette.textStrong`, compiled to `--color-surface` and `--color-text-strong`; prefer those over hardcoded greys.

## When to use it

- **Homepage/landing hero accents** — animated titles, orbiting shapes, looping product reveals
- **SaaS-style product demos** — Stripe-homepage-style mini-browser showing a fake checkout, basket filling, cards flipping
- **Scroll-scrubbed feature sections** — elements morph/translate as the user scrolls
- **Not for** ordinary static hero sections — use the `hero` extension for those

## MCP Tools to Use

| Action | Tool |
|--------|------|
| Find existing extensions on an item | `listExtensionsOnItem` |
| Add a new animation scene | `addExtensionToItem` (extensionName: `animationScene`) |
| Update the scene spec | `updateExtensionConfig` — set the `sceneConfig` property (JSON string) |
| Find images for static elements | `listMedia` |
| Find a collection for product binding | `listCollections` (or equivalent) |

The `sceneConfig` property stores a JSON **string**. When updating, pass the JSON stringified — Badger will compact it on save.

## Scene Spec Structure

```json
{
  "version": "1",
  "canvas": { "height": "500px", "background": "var(--color-gray-50)" },
  "bindings": {
    "featured": { "source": "collection.products", "collectionSeoName": "featured", "limit": 3 }
  },
  "elements": [ /* ... */ ],
  "timeline": { "trigger": "onEnter", "steps": [ /* ... */ ] }
}
```

| Top-level key | Required | Description |
|---|---|---|
| `version` | yes | Always `"1"` |
| `canvas` | no | Container dims + background. `height`, `width`, `background`, `padding`. Pinned scroll scenes auto-size to `100vh` if `height` is omitted |
| `bindings` | no | Named data lookups, resolved server-side. Each maps a name → source def |
| `elements` | yes | Flat array of elements to place in the canvas |
| `timeline` | no | GSAP timeline with trigger + steps. Omit for static scenes |

## Element Types

Every element has `type`, optional `id`, optional `style` (CSS properties as camelCase map), and optional `content`. Elements are absolutely positioned inside the canvas — set `top`, `left`, and a `transform` if you want centering. The runtime parses `transform: translate()/translateX()/translateY()/scale()/rotate()` and applies them via GSAP's native transform props, so static centering like `translate(-50%, -50%)` composes cleanly with later `x`/`y`/`scale`/`rotation` tweens.

| Type | Purpose | Content / extras |
|---|---|---|
| `text` | Text node | `content`: string |
| `image` | `<img>` | `content` (or `src`): url; `alt`: string |
| `shape` | Decorative block | `shape`: `"rect"` (default) or `"circle"` |
| `mockBrowser` | Chrome-style frame with traffic lights + URL bar; body is `--color-gray-50` so default white product cards stand out | `url`: string; `children`: element array rendered inside (children can use `repeat`) |
| `productCard` | Compact card (image / title / price) | `content`: `{title, image, price}` |
| `group` | Container for nesting | `children`: element array |

### Style properties

Use any CSS property (camelCase). Position is `absolute` by default. For layout inside groups/mockBrowser bodies, override `position: "relative"` on children.

**Design-token-friendly values** — Nova CSS custom properties are available:
- Colors: `var(--color-primary)`, `var(--color-gray-50)` → `var(--color-gray-900)`, `var(--color-success)`, `var(--color-error)`
- Spacing: `var(--space-1)` (4px) → `var(--space-20)` (80px)
- Typography: `var(--font-size-xs)` → `var(--font-size-5xl)`, `var(--font-weight-medium)` / `--semibold` / `--bold`
- Radii: `var(--radius-sm)` → `var(--radius-2xl)`, `var(--radius-full)`
- Shadows: `var(--shadow-xs)` → `var(--shadow-xl)`

Prefer tokens over hardcoded values so scenes inherit tenant theming.

### Repeating elements

An element with a `repeat` block is expanded into one copy per item in the named binding. The element's `id` becomes a **CSS class** on every instance; each instance also gets a unique `id` (`{id}-{index}`). Target all copies from the timeline with `.id` (class selector).

`repeat` works at any nesting depth — including inside `mockBrowser.children` and `group.children`. Children without their own `repeat` inherit the parent's `{{item.*}}` context, so a non-repeating child of a repeated parent can still reference the parent's bound item.

```json
{
  "id": "card",
  "type": "productCard",
  "repeat": { "binding": "featured" },
  "style": { "top": "55%", "left": "calc(20% + 120px * {{index}})", "width": "160px" },
  "content": { "title": "{{item.name}}", "image": "{{item.primaryImage}}", "price": "{{item.price}}" }
}
```

### Links / interactivity

Any element becomes clickable by adding `href`. The runtime wraps the element in an `<a>` so GSAP selectors (`#id`, `.class`) still resolve to the positioned node. Pairs naturally with bindings — `href: "{{item.url}}"` on a repeating `productCard` turns a passive product reveal into a clickable grid. **Always link product cards bound to `collection.products` so visitors can navigate to the product page** — otherwise the scene is just a slideshow.

| Field | Purpose |
|---|---|
| `href` | URL to navigate to. Tokens substituted — use `{{item.url}}` which resolves to `/product/{seoName}` for products and `/collection/{seoName}` for collections |
| `target` | `_blank` opens in a new tab. Runtime auto-adds `rel="noopener noreferrer"` |
| `rel` | Explicit rel value; overrides the `_blank` default |
| `ariaLabel` | Accessible label for links without descriptive inner text (e.g. shape- or image-only links) |

```json
{
  "id": "card", "type": "productCard", "href": "{{item.url}}",
  "repeat": { "binding": "featured" },
  "style": { "top": "40%", "left": "calc(5% + {{index}} * 32%)", "width": "28%" },
  "content": { "title": "{{item.name}}", "image": "{{item.primaryImage}}", "price": "{{item.price}}" }
}
```

Linked elements get a subtle hover effect (brightness + shadow lift) that composes with GSAP transforms, so hover works even during scroll-scrubbed animations.

## Bindings

Bindings are resolved server-side before the spec reaches the browser. Each binding is a named data source; use the name in `repeat.binding` and in `{{item.*}}` tokens inside that element.

| Source | Required params | Returns |
|---|---|---|
| `product.current` | — (only works on a product page) | 1 item |
| `collection.current` | — (only works on a collection page) | 1 item |
| `collection.products` | `collectionSeoName` OR `collectionId`; optional `limit` (default 12, max 48) | N items |

### Product item fields

`{{item.id}}`, `{{item.name}}`, `{{item.seoName}}`, `{{item.skuId}}`, `{{item.price}}` (formatted, e.g. `£19.99`), `{{item.comparePrice}}`, `{{item.priceRaw}}` (integer pence), `{{item.url}}`, `{{item.primaryImage}}`, `{{item.thumbnailUrl}}`

### Collection item fields

`{{item.id}}`, `{{item.name}}`, `{{item.seoName}}`, `{{item.url}}`

## Tokens

Tokens are substituted in every string value — `content`, `style`, `url`, `alt`, `href`, `ariaLabel`, inside `children`.

| Token | Resolves to |
|---|---|
| `{{item.field}}` | Field on the current repeat item |
| `{{index}}` | 0-based index inside a repeat |
| `{{count}}` | Total items in the repeat |
| `{{config.key}}` | Site config value by key (scalar) |

## Timeline

```json
{
  "trigger": "onEnter",
  "threshold": 0.3,
  "steps": [
    { "target": "#title", "from": {"y": 40, "opacity": 0}, "to": {"duration": 0.6, "ease": "power2.out"} },
    { "target": ".card",  "from": {"y": 30, "opacity": 0}, "to": {"duration": 0.5, "stagger": 0.15, "ease": "power2.out"}, "delay": 0.3 }
  ]
}
```

### Triggers

| Trigger | Behaviour |
|---|---|
| `immediate` | Runs as soon as the scene is built |
| `onEnter` (default) | Runs once when the scene enters the viewport. `threshold` is the IntersectionObserver threshold (default 0.3) |
| `scroll` | ScrollTrigger scrub — timeline progress bound to scroll position. Optional `start`, `end` (GSAP strings), `scrub` (true or number). Set `pin: true` for Apple-style multi-phase scroll stories: canvas auto-sizes to `100vh`, pin distance auto-computes from timeline duration at ≈200px scroll per second. Override `end: "+=Npx"` for explicit pin length |
| `loop` | Runs in a loop forever. Optional `yoyo` (bool), `repeatDelay` (seconds) |

### Steps

Each step is one GSAP tween. `target` is a CSS selector scoped to the scene container.

| Field | Required | Notes |
|---|---|---|
| `target` | yes | `#id` for single, `.class` for repeats, or any CSS selector |
| `from` | no | Starting state for properties (e.g. `y`, `opacity`, `scale`). See below for what gets used |
| `to` | yes | Tween vars: `duration`, `ease`, `stagger` — and optionally destination property values |
| `delay` | no | Absolute position on the timeline (seconds). Pass `"+=0.5"` for relative positioning — GSAP accepts both |

**`from` vs `fromTo` — how it's resolved:**

| Step shape | GSAP call | Meaning |
|---|---|---|
| `from` + `to` with only metadata (duration/ease/stagger/…) | `gsap.from()` | Tween FROM `from` values TO the element's current CSS state. **This is the common case** — elements settle into their natural styled state |
| `from` + `to` that also has property values (`y`, `opacity`, etc.) | `gsap.fromTo()` | Tween FROM `from` TO the exact values in `to`. Use when you need both endpoints explicit (scroll-scrub is a common case) |
| `to` only (no `from`) | `gsap.to()` | Tween FROM current CSS TO `to` values |

### Easing

Common easing strings: `none`, `power1.out` / `power2.out` / `power3.out` / `power4.out`, `power1.in` / `in.out`, `back.out(1.7)`, `elastic.out(1, 0.3)`, `bounce.out`, `expo.out`, `sine.inOut`.

## Preset Recipes

### 1. Product reveal — cards fly in from below

```json
{
  "version": "1",
  "canvas": { "height": "520px", "background": "var(--color-gray-50)" },
  "bindings": {
    "featured": { "source": "collection.products", "collectionSeoName": "featured", "limit": 3 }
  },
  "elements": [
    { "id": "title", "type": "text", "content": "This season's picks",
      "style": { "top": "15%", "left": "50%", "transform": "translateX(-50%)",
                 "fontSize": "var(--font-size-4xl)", "fontWeight": "var(--font-weight-bold)",
                 "color": "var(--color-gray-900)" } },
    { "id": "card", "type": "productCard", "href": "{{item.url}}",
      "repeat": { "binding": "featured" },
      "style": { "top": "45%", "left": "calc(18% + {{index}} * 22%)", "width": "200px" },
      "content": { "title": "{{item.name}}", "image": "{{item.primaryImage}}", "price": "{{item.price}}" } }
  ],
  "timeline": {
    "trigger": "onEnter", "threshold": 0.3,
    "steps": [
      { "target": "#title", "from": {"y": 30, "opacity": 0}, "to": {"duration": 0.5, "ease": "power2.out"} },
      { "target": ".card",  "from": {"y": 40, "opacity": 0}, "to": {"duration": 0.6, "stagger": 0.15, "ease": "power3.out"}, "delay": 0.2 }
    ]
  }
}
```

### 2. Stripe-style mock browser — product demo frame

```json
{
  "version": "1",
  "canvas": { "height": "560px", "background": "var(--color-gray-100)" },
  "bindings": {
    "featured": { "source": "collection.products", "collectionSeoName": "best-sellers", "limit": 3 }
  },
  "elements": [
    { "id": "browser", "type": "mockBrowser", "url": "yourstore.com",
      "style": { "top": "50%", "left": "50%", "transform": "translate(-50%, -50%)",
                 "width": "680px", "height": "440px" },
      "children": [
        { "id": "card", "type": "productCard", "href": "{{item.url}}",
          "repeat": { "binding": "featured" },
          "style": { "position": "absolute", "top": "30%", "left": "calc(6% + {{index}} * 32%)", "width": "30%" },
          "content": { "title": "{{item.name}}", "image": "{{item.primaryImage}}", "price": "{{item.price}}" } }
      ] }
  ],
  "timeline": {
    "trigger": "onEnter",
    "steps": [
      { "target": "#browser", "from": {"scale": 0.92, "opacity": 0}, "to": {"duration": 0.7, "ease": "power3.out"} },
      { "target": ".card",    "from": {"y": 20, "opacity": 0},       "to": {"duration": 0.5, "stagger": 0.18, "ease": "power2.out"}, "delay": 0.4 }
    ]
  }
}
```

### 3. Scroll-scrub feature — text translates as you scroll

```json
{
  "version": "1",
  "canvas": { "height": "800px" },
  "elements": [
    { "id": "tagline", "type": "text", "content": "Ship faster. Every time.",
      "style": { "top": "50%", "left": "50%", "transform": "translate(-50%, -50%)",
                 "fontSize": "var(--font-size-5xl)", "fontWeight": "var(--font-weight-bold)" } }
  ],
  "timeline": {
    "trigger": "scroll",
    "start": "top bottom",
    "end": "bottom top",
    "scrub": true,
    "steps": [
      { "target": "#tagline", "from": {"x": -200, "opacity": 0.2},
        "to": {"x": 200, "opacity": 1} }
    ]
  }
}
```

### 4. Apple-style scroll-pin story — multi-phase reveal over a long scroll

Pinned canvas stays in view while the user scrolls through several phases — intro title → hero image + feature pills → shrinking hero → outro tagline → CTA. Because `pin: true`, the canvas auto-sizes to `100vh` and the pin distance is auto-computed from the timeline's total duration. No explicit `canvas.height` or `timeline.end` needed. Place elements using `vh`-based `top` values (or percentages — same thing since the canvas is viewport-sized) so phases land inside the visible area.

```json
{
  "version": "1",
  "canvas": {
    "background": "linear-gradient(180deg, #0a0a1f 0%, #1a0a2e 50%, #0a1a2e 100%)"
  },
  "elements": [
    { "id": "orb-a", "type": "shape", "shape": "circle",
      "style": { "top": "15vh", "left": "-10%", "width": "500px", "height": "500px",
                 "background": "radial-gradient(circle, rgba(124,58,237,0.4), rgba(124,58,237,0) 70%)",
                 "filter": "blur(30px)", "zIndex": "1" } },
    { "id": "orb-b", "type": "shape", "shape": "circle",
      "style": { "top": "50vh", "right": "-15%", "width": "600px", "height": "600px",
                 "background": "radial-gradient(circle, rgba(236,72,153,0.35), rgba(236,72,153,0) 70%)",
                 "filter": "blur(35px)", "zIndex": "1" } },

    { "id": "title-intro", "type": "text", "content": "The future of commerce.",
      "style": { "top": "42vh", "left": "5%", "right": "5%", "textAlign": "center",
                 "fontSize": "clamp(40px, 7vw, 100px)", "fontWeight": "700",
                 "color": "#fff", "letterSpacing": "-0.03em", "lineHeight": "1.05",
                 "opacity": "0", "zIndex": "3" } },

    { "id": "hero-img", "type": "image",
      "content": "https://picsum.photos/seed/bdgr-hero/1200/750", "alt": "",
      "style": { "top": "18vh", "left": "15%", "width": "70%", "maxWidth": "900px",
                 "aspectRatio": "16 / 10", "objectFit": "cover",
                 "borderRadius": "24px", "boxShadow": "0 40px 100px rgba(0,0,0,0.6)",
                 "opacity": "0", "zIndex": "2" } },

    { "id": "feat-1", "type": "text", "content": "⚡ Blazing fast",
      "style": { "top": "30vh", "left": "6%", "fontSize": "clamp(18px, 1.8vw, 28px)",
                 "fontWeight": "600", "color": "#fff",
                 "padding": "12px 22px", "background": "rgba(255,255,255,0.08)",
                 "border": "1px solid rgba(255,255,255,0.15)",
                 "borderRadius": "9999px", "backdropFilter": "blur(10px)",
                 "opacity": "0", "zIndex": "4" } },
    { "id": "feat-2", "type": "text", "content": "✨ Beautifully designed",
      "style": { "top": "44vh", "right": "6%", "fontSize": "clamp(18px, 1.8vw, 28px)",
                 "fontWeight": "600", "color": "#fff",
                 "padding": "12px 22px", "background": "rgba(255,255,255,0.08)",
                 "border": "1px solid rgba(255,255,255,0.15)",
                 "borderRadius": "9999px", "backdropFilter": "blur(10px)",
                 "opacity": "0", "zIndex": "4" } },
    { "id": "feat-3", "type": "text", "content": "🎨 Built for your brand",
      "style": { "top": "70vh", "left": "50%", "transform": "translateX(-50%)",
                 "width": "340px",
                 "fontSize": "clamp(18px, 1.8vw, 28px)",
                 "fontWeight": "600", "color": "#fff", "textAlign": "center",
                 "padding": "12px 22px", "background": "rgba(255,255,255,0.08)",
                 "border": "1px solid rgba(255,255,255,0.15)",
                 "borderRadius": "9999px", "backdropFilter": "blur(10px)",
                 "opacity": "0", "zIndex": "4" } },

    { "id": "title-outro", "type": "text", "content": "Powered by Badger.",
      "style": { "top": "38vh", "left": "5%", "right": "5%", "textAlign": "center",
                 "fontSize": "clamp(36px, 6vw, 88px)", "fontWeight": "700",
                 "letterSpacing": "-0.02em",
                 "background": "linear-gradient(90deg, #a78bfa, #f472b6)",
                 "webkitBackgroundClip": "text", "webkitTextFillColor": "transparent",
                 "backgroundClip": "text", "color": "transparent",
                 "opacity": "0", "zIndex": "5" } },

    { "id": "cta", "type": "text", "content": "Get started →",
      "style": { "top": "58vh", "left": "50%", "transform": "translateX(-50%)",
                 "width": "220px",
                 "padding": "18px 36px", "background": "#fff", "color": "#0a0a1f",
                 "borderRadius": "9999px", "fontSize": "20px", "fontWeight": "600",
                 "textAlign": "center", "boxShadow": "0 20px 50px rgba(255,255,255,0.25)",
                 "opacity": "0", "zIndex": "6" } }
  ],
  "timeline": {
    "trigger": "scroll", "pin": true, "scrub": 1,
    "steps": [
      { "target": "#orb-a", "from": {"x": -60}, "to": {"x": 80, "duration": 10, "ease": "none"}, "delay": 0 },
      { "target": "#orb-b", "from": {"x": 60}, "to": {"x": -80, "duration": 10, "ease": "none"}, "delay": 0 },

      { "target": "#title-intro", "from": {"y": 40, "opacity": 0}, "to": {"opacity": 1, "y": 0, "duration": 1, "ease": "power2.out"}, "delay": 0 },
      { "target": "#title-intro", "to": {"opacity": 0, "y": -30, "duration": 1, "ease": "power2.in"}, "delay": 1.8 },

      { "target": "#hero-img", "from": {"scale": 0.9, "y": 60, "opacity": 0}, "to": {"opacity": 1, "scale": 1, "y": 0, "duration": 1.5, "ease": "power3.out"}, "delay": 2.2 },

      { "target": "#feat-1", "from": {"x": -60, "opacity": 0}, "to": {"opacity": 1, "x": 0, "duration": 0.8, "ease": "power2.out"}, "delay": 3.8 },
      { "target": "#feat-2", "from": {"x": 60, "opacity": 0}, "to": {"opacity": 1, "x": 0, "duration": 0.8, "ease": "power2.out"}, "delay": 4.6 },
      { "target": "#feat-3", "from": {"y": 40, "opacity": 0}, "to": {"opacity": 1, "y": 0, "duration": 0.8, "ease": "power2.out"}, "delay": 5.4 },

      { "target": "#hero-img", "to": {"scale": 0.35, "y": -280, "opacity": 0.6, "duration": 1.2, "ease": "power2.inOut"}, "delay": 6.8 },
      { "target": "#feat-1", "to": {"opacity": 0, "x": -60, "duration": 0.6, "ease": "power2.in"}, "delay": 6.8 },
      { "target": "#feat-2", "to": {"opacity": 0, "x": 60, "duration": 0.6, "ease": "power2.in"}, "delay": 6.8 },
      { "target": "#feat-3", "to": {"opacity": 0, "y": 40, "duration": 0.6, "ease": "power2.in"}, "delay": 6.8 },

      { "target": "#title-outro", "from": {"y": 40, "scale": 0.95, "opacity": 0}, "to": {"opacity": 1, "y": 0, "scale": 1, "duration": 1.2, "ease": "power3.out"}, "delay": 7.8 },

      { "target": "#cta", "from": {"scale": 0.7, "opacity": 0}, "to": {"opacity": 1, "scale": 1, "duration": 0.8, "ease": "back.out(1.7)"}, "delay": 8.8 }
    ]
  }
}
```

**How it works:**
- `pin: true` locks the canvas to the viewport while the user scrolls. Canvas auto-sizes to `100vh`; pin distance auto-computes as ≈200px × timeline duration (here ~9.6s → ~1920px of scroll)
- Override with `timeline.end: "+=1500"` etc. for an explicit pin length
- `scrub: 1` smooths the timeline-to-scroll mapping (1 second of catch-up)
- Elements start at `opacity: 0` so they're hidden before their tween arrives
- `delay` is an absolute timeline position in seconds; with scrub it maps proportionally to scroll progress
- Centering via `transform: translateX(-50%)` is safe — the runtime parses it into GSAP's `xPercent` so later `x`/`y`/`scale` tweens compose cleanly instead of clobbering the transform matrix

### 5. Looping hero accent — gentle floating shape

```json
{
  "version": "1",
  "canvas": { "height": "300px", "background": "var(--color-primary)" },
  "elements": [
    { "id": "orb", "type": "shape", "shape": "circle",
      "style": { "top": "50%", "left": "20%", "width": "60px", "height": "60px",
                 "background": "rgba(255,255,255,0.3)" } }
  ],
  "timeline": {
    "trigger": "loop", "yoyo": true,
    "steps": [
      { "target": "#orb", "to": { "y": -40, "duration": 2.5, "ease": "sine.inOut" } }
    ]
  }
}
```

## Default Page Location

`mainSection` — scenes render in the main content area by default. Use `bannerSection` for above-the-fold heroes.

## Decision Tree

1. **Static hero with form-based editing?** → `hero` extension (5 presets)
2. **Freeform static layout?** → `jsonComponent` extension
3. **Animated, live, data-bound scene?** → `animationScene` (this one)
4. **Long cinematic video?** → not this; use a video element in `jsonComponent`

## Accessibility

- The runtime auto-respects `prefers-reduced-motion: reduce` — tweens are skipped, but elements still render with their `to` end-state opacity so the page isn't broken.
- Don't hide critical information behind an animation — the reduced-motion path should still look good.
- Text inside scenes remains selectable and keyboard-focusable.

## Common Pitfalls

- **Hardcoded pixel positions break on mobile.** Prefer `%`-based positioning, or use `calc()` with `{{index}}` for repeated layout. For pinned scroll scenes, `vh` and `%` are equivalent (canvas auto-sizes to `100vh`).
- **Don't forget to absolute-position children of `mockBrowser` or `group`** — mock browser bodies are `position: relative`, so children need `style.position: "absolute"` and explicit `top`/`left`.
- **Selectors are CSS, not IDs** — use `#name` for singletons, `.name` for repeated elements (the `id` of a repeating template becomes the class).
- **Don't mix `repeat` with data-less sources** — `product.current` and `collection.current` return at most 1 item, so `repeat` on them is legal but wasteful; drop `repeat` and use `{{item.*}}` only inside a non-repeating element with `binding.source = "product.current"` — *except that's not supported directly*; the cleaner path is: put the element inside a `repeat` block regardless, and accept the 1-item expansion.
- **Scenes with 0 elements after binding resolution are invisible.** If a binding resolves to `[]` (e.g. the collection doesn't exist), repeated elements vanish. Always provide a sensible non-bound fallback element (e.g. a title).
- **Don't exceed the viewport on mobile.** Test at 375px width — heavy mock-browser layouts often need mobile-specific widths.

## v2 Roadmap (not yet available)

- Tenant-data bindings for analytics and order counts (opt-in, obfuscated)
- Admin preview inside the edit page (v1 is live-only)
- Scene-level dependency on `hover` / `click` triggers
- Parallax and mouse-follow effects
