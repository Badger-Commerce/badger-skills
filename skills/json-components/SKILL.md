---
name: json-components
description: Build and edit Badger Commerce JSON components using the MCP tools — covers the full component catalog, semantic style tokens, data binding, and data sources
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# JSON Component Builder

You are building JSON component definitions for Badger Commerce's `jsonComponent` extension. This skill contains all the reference material you need — **do not call `extensions` getSchema or getPromptSection** for the jsonComponent extension.

## Stay on-brand: load the design brief first

**Before generating components, call `getDesignBrief` via MCP.** Use the returned `palette` hex values rather than hardcoding colours, and match the `voice.tone` for any copy. Nova's CSS custom properties (e.g. `var(--color-primary)`) already inherit the brief's palette at runtime — but when you need an explicit hex value (background images, SVG fills, inline gradients), pull it from `getDesignBrief` rather than reusing colours from this skill's examples. The brief's `shop.theme` tells you which token family applies (see Semantic Style Tokens) and `shop.productImagePath` is the base for media URLs (see Media).

When the brief sets `palette.gradient`, prefer `background: var(--gradient-brand)` over reassembling a `linear-gradient(...)` from the palette. For card/surface backgrounds use `var(--color-surface)` and for heading-weight text use `var(--color-text-strong)` — both follow the tenant's brief (light-mode defaults when unset, brand-specific values when set, e.g. dark cards for dark-mode brands).

## MCP Tools to Use

| Action | Tool |
|--------|------|
| Find existing extensions on an item | `extensionConfig` (itemType, itemId) |
| Add a new jsonComponent extension | `manageExtensionConfig` add (extensionName: "jsonComponent", pageLocation: a slot from listPageLocations; it is required) |
| Update the JSON definition | `manageExtensionConfig` update (extensionConfigId) — set the `jsonComponent` property |
| Find images for use in components | `media` list (`manageMedia` import adds one the merchant gives you) |
| List available page slots | `extensions` listPageLocations (itemType, itemId) |

The `jsonComponent` config property must contain a JSON document with `"version": "1.0"` and a `component` field defining the component tree (a JSON string or an object; it is validated on save, see Validation).

## Decision Tree: Which Extension to Use?

1. **Need a hero section?** → Use the `hero` extension (5 curated presets, form-based editing)
2. **Standard pattern, no purpose-built extension?** → Use `jsonComponent` with a starter template
3. **Truly custom layout?** → Use `jsonComponent` freeform
4. **Appeal band, call-to-action strip, sidebar panel or stat card?** → Use `callToAction` (form-based: `variant` band/strip/panel/stat, `eyebrow`, `heading`, `text`, `value`, `primaryLabel`/`primaryUrl`, `secondaryLabel`/`secondaryUrl`). Themes such as Ridge give it their own design, which a jsonComponent band won't get.
5. **Long-form prose (article/blog body, policy text, anything with lists or sub-headings)?** → Use `markdownFragment`. jsonComponent `text` is a plain, escaped `<p>`: no markdown, lists, links or line breaks. Article bodies go in `markdownFragment`; jsonComponent is for the layout pieces around them (hero, feature grid, CTA band, product strip).

---

## Component Catalog

`(b)` = bindable (accepts `{"$ref": ...}`). Style keys listed are the ones the validator expects for that type; others draw a warning.

### Layout Components (have children)

| Type | Props | Style keys |
|------|-------|-----------|
| container | — | spacing, padding, background, backgroundImage, backgroundOverlay, backgroundPosition, backgroundSize, radius, shadow, maxWidth, border, animation |
| row | stackOnMobile (bool, default true) | gap, align, justify, animation |
| column | width [1/4, 1/3, 1/2, 2/3, 3/4, full, auto] (default auto), mobileWidth [1/2, full, auto] (default full) | gap, align, maxWidth, animation |
| tabs | children should be `tab` (not validated) | variant |
| tab | label (b), active (bool) | — |
| accordion | allowMultiple (bool); children should be `accordionItem` (not validated) | variant |
| accordionItem | title (b), expanded (bool) | — |

### Content Components (no children)

| Type | Props | Style keys |
|------|-------|-----------|
| heading | level (1-6, default 2), text (b) | variant, color, align, transform, tracking, textStyle |
| text | text (b) — plain text only | variant, color, align, transform, tracking, textStyle |
| link | text (b), href (b), action | variant |
| button | text (b), icon [arrow-right, arrow-left, chevron-right, chevron-left, shopping-cart, shopping-bag, heart, star, check, plus, minus, search, download, upload, external-link, mail, phone, user, log-in, log-out], iconPosition [left, right] (default right), action | variant, color |
| image | src (b), alt (b) | radius, shadow, width, height |
| icon | name (see below), size [xs, sm, md, lg, xl] | color |
| blockquote | quote (b), author (b), role (b), avatar (b) | variant, color, background |

**icon names** (anything else is rejected): arrow-right, arrow-left, arrow-up, arrow-down, chevron-right, chevron-left, chevron-up, chevron-down, menu, x, check, plus, minus, search, settings, edit, trash, copy, download, upload, external-link, shopping-cart, shopping-bag, credit-card, tag, gift, percent, truck, package, mail, phone, message-circle, share, heart, star, thumbs-up, info, alert-circle, check-circle, x-circle, help-circle, user, users, log-in, log-out, home, calendar, clock, map-pin, eye, lock, shield, award, zap, sparkles, rocket

### Utility Components (no children)

| Type | Props | Style keys |
|------|-------|-----------|
| spacer | size [xs, sm, md, lg, xl] (default md) | — |
| divider | thickness [thin, md, thick], lineStyle [solid, dashed, dotted] | color, spacing (neither has a visible effect on Nova) |
| carousel | images (array of `{src, alt, caption}`, each bindable), autoPlay (bool), interval (1000-30000ms, default 5000), showIndicators (bool), showControls (bool) | radius, shadow |

### Product Components (no children)

| Type | Props | Style keys |
|------|-------|-----------|
| productGrid | dataSource, columns (2-6, default 4), mobileColumns (1-3, default 2), showPrice, showDescription, cardVariant [standard, compact, featured] | gap, padding, background, radius |
| productCarousel | dataSource, autoScroll (bool), scrollInterval (1000-30000ms), showPrice, cardVariant | gap, padding, background, radius |
| productCard | dataSource, index (0-100), showImage, showPrice, showDescription, variant [standard, compact, featured] | padding, background, radius, shadow |

---

## Semantic Style Tokens

Use these in the `"style"` prop. Never use raw CSS values. Unknown token values are silently dropped (no error).

**Theme families:** nova, brock, depot, pop, atelier, spec and ridge use the Nova mapping; every other theme (bootstrap and anything unmapped) uses the Bootstrap mapping. Tokens marked *Nova* only render properly on the Nova family; Bootstrap gets a plain fallback (noted).

Aliases: `backgroundColor` = `background`, `textAlign` = `align`, `borderRadius` = `radius`.

| Key | Tokens | Notes |
|-----|--------|-------|
| variant (type) | heading-xl, heading-lg, heading-md, heading-sm, body-lg, body-md, body-sm, link | heading-xl = display, lg ≈ h1, md ≈ h2, sm ≈ h3; body-lg = lead |
| variant (marketing) | display-xl, display-lg, display-md, subheadline, overline, overline-pill, link-arrow | Big fluid headlines, lead copy, small-caps kicker (pill = badge), "Read more →" link. Bootstrap: inline approximations |
| variant (button) | button-primary, button-secondary | |
| color | primary, secondary (on Nova a mid grey, not the brief's secondary colour), muted, success, error, warning; primary-light/-dark, secondary-light/-dark, success-light/-dark, error-light/-dark, warning-light/-dark; gray-light, gray, gray-dark; white, black | |
| background | surface, surface-light, surface-dark, white, transparent, secondary, muted, primary, primary-light, primary-dark, primary-subtle, success, success-light, error, error-light, warning, warning-light, gray-light, gray, gray-dark | |
| background (dark/pattern) | dark, gradient-dark, mesh-dark, gradient, mesh, dots | Nova: themed section surfaces. Bootstrap: mesh = plain white, mesh-dark = plain dark |
| padding | none (0), xs (4px), sm (8), md (16), lg (24), xl (32), 2xl (64), 3xl (96), section (64 / 24 sides), section-lg (96 / 24 sides) | Use section / section-lg for full-width bands |
| gap | none, xs, sm, md, lg, xl, 2xl (48px), 3xl (64px) | |
| spacing | none, xs, sm, md, lg, xl | Sets the gap between a container's children |
| align | left, center, right, start, end | center also centres flex children |
| justify | start, center, end, between, around | |
| radius | none, sm, md, lg, full | |
| shadow | none, sm, md, lg, xl | |
| border | default, light, dark, none, accent-top, gradient-top | Bootstrap: gradient-top = solid accent |
| width / height | full, auto | |
| maxWidth | prose (65ch), sm (640px), md (768px), lg (1024px), xl (1280px), full, none | Constrained widths are centred |
| transform | uppercase, lowercase, capitalize, none | |
| tracking | tight, normal, wide, wider, widest | |
| textStyle | gradient | Gradient-filled text |
| animation | reveal, stagger | Scroll-in reveal; stagger animates children in turn. *Nova only* (no-op on Bootstrap) |
| backgroundImage | URL string (container) | See Media |
| backgroundPosition | center, top, bottom, left, right | |
| backgroundSize | cover, contain, auto | |
| backgroundOverlay | none, light (30%), medium (50%), dark (70%), heavy (85%) | Black overlay, only with backgroundImage |

**Dark sections (Nova):** background primary, primary-dark, dark, gradient-dark and mesh-dark make headings white and switch overlines/buttons to on-dark colours automatically, and body text turns light. Don't add `color` tokens to text inside them.

---

## Data Binding

Use `{"$ref": "/context/path", "fallback": "default"}` for dynamic values. Optional `"format"`: capitalize, uppercase, lowercase, trim. Valid contexts: user, order, siteContext, item, lastOrder, data (anything else is rejected). `lastOrder` passes validation but is never filled, so it always renders the fallback.

### Context Paths

| Path Prefix | Examples |
|------------|----------|
| /user/* | /user/loginName, /user/personalDetails/firstName, /user/personalDetails/lastName, /user/personalDetails/emailAddress, /user/attributes/{key} |
| /order/* | /order/number, /order/state, /order/price/total (cents), /order/price/currencyCode, /order/lineItems, /order/personalDetails/firstName |
| /siteContext/* | /siteContext/siteName, /siteContext/logoURL, /siteContext/productImagePath, /siteContext/socialAccounts/{platform} |
| /item/* (product) | /item/productName, /item/price (cents), /item/comparePrice, /item/description, /item/miniDescription, /item/manufacturer, /item/seoName, /item/images/0/url, /item/images/0/altText, /item/quantityInStock, /item/attributes/{key} |
| /item/* (collection) | /item/name, /item/description, /item/seoName |
| /item/* (page) | /item/name, /item/attributes/{key} |
| /data/* | /data/{sourceName}/0/productName (see Data Sources) |

---

## Data Sources

Declare at root level alongside `"component"`:

```json
{
  "version": "1.0",
  "dataSources": {
    "mySource": { "type": "source-type", "params": { ... } }
  },
  "component": { ... }
}
```

### Available Providers

| Type | Params | Description |
|------|--------|-------------|
| trending-products | limit (int, default 8), hoursToCheck (int, default 24) | Trending products by recent views |
| collection-products | collection (seoName, required), limit (int, default 12), offset (int, default 0; rounded down to a multiple of limit), inStockOnly (bool, default false; drops disabled products, not out-of-stock ones) | Products from a collection |
| merchandising-node-products | nodeId OR path (canonical, e.g. "/c/womens/coats"; nodeId wins), treeId (default primary tree), limit (int, default 12), offset (int, default 0) | Products in a category-tree (merchandising) node |
| recently-viewed | limit (int, default 10) | User's recently viewed products (requires login) |

`collection-products` reads the collection as a product set, so it works on a category-tree site too, even when the collection's own page is replaced or hidden.

Reference in components via `"dataSource": "mySource"` prop. Also accessible via `$ref`: `/data/{sourceName}/0/productName`. An unknown node or collection renders an empty list, not an error.

---

## Media

`image` src, `blockquote` avatar, `carousel` image src and `backgroundImage` are used **verbatim** — no base path is added. Use an absolute URL: `shop.productImagePath` from `getDesignBrief` + `/` + the media `url` from the `media` tool (e.g. `//images.bdgr.co.uk/acme/media/hero.jpg`), or a full `https://` URL.

Product images from a data source (productGrid/productCarousel/productCard) are prefixed for you. A bound `{"$ref": "/item/images/0/url"}` is not prefixed, so a relative image path won't load in an `image`.

---

## Layout Modes

Set `"layout"` at root level:
- `"fluid"` (default) — Full-width, edge-to-edge. Use for hero sections.
- `"contained"` — Constrained to page container. Use for standard content.

For full-width background with constrained content: use `"layout": "fluid"` with a container child that has `"maxWidth": "prose"` or similar.

## Validation

`manageExtensionConfig` rejects the save on **errors**: missing `component` or `type`; unknown component type; `children` on a leaf type or not an array; enum value outside its list (icon name, width, size, cardVariant…); wrong value type (number/boolean/string); `$ref` on a non-bindable prop, not starting with `/`, or with an unknown context; `action` without `type: "navigate"` and `href`; invalid JSON; `style` or `action` that isn't an object. Numeric ranges (columns, interval, index) aren't checked, only the type.

It saves with **warnings**, which the MCP result doesn't show: unknown prop and unknown node field (both ignored), style key not listed for that type (still applied if the renderer knows the key), unknown `format`, missing `version`. Token values are not checked: a misspelt token just renders nothing.

## Rules

1. Wrap in `{"version": "1.0", "component": {...}}`
2. Every component needs `"type"` and optionally `"key"`, `"props"`, `"children"`
3. Use ONLY semantic tokens — never raw CSS values
4. For navigation: `"action": {"type": "navigate", "href": "/path"}`. Take the path from the target's `url` in the MCP tools; hrefs aren't rewritten, so on a category-tree site (`category-navigation-source` MERCH_TREE) link categories as `/c/...`, not `/collection/...`
5. Output valid JSON only — no markdown, no explanations
6. Prices are in cents
7. Use meaningful fallback values for data bindings
8. `text` is plain text; long-form copy goes in `markdownFragment` (see Decision Tree)

---

## Example: Welcome Banner with Data Binding

```json
{
  "version": "1.0",
  "component": {
    "type": "container",
    "props": {
      "style": { "padding": "lg", "background": "surface", "radius": "md" }
    },
    "children": [
      {
        "type": "heading",
        "props": {
          "level": 2,
          "text": { "$ref": "/user/personalDetails/firstName", "fallback": "Welcome", "format": "capitalize" },
          "style": { "variant": "heading-lg", "color": "primary" }
        }
      },
      {
        "type": "text",
        "props": {
          "text": "Thanks for visiting our store!",
          "style": { "variant": "body-md", "color": "muted" }
        }
      },
      { "type": "spacer", "props": { "size": "md" } },
      {
        "type": "button",
        "props": {
          "text": "Browse Products",
          "action": { "type": "navigate", "href": "/collections" },
          "style": { "variant": "button-primary" }
        }
      }
    ]
  }
}
```
