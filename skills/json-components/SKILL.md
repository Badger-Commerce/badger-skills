---
name: json-components
description: Build and edit Badger Commerce JSON components using the MCP tools — covers the full component catalog, semantic style tokens, data binding, and data sources
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# JSON Component Builder

You are building JSON component definitions for Badger Commerce's `jsonComponent` extension. This skill contains all the reference material you need — **do not call `getExtensionConfigSchema` or `getExtensionPromptSection`** for the jsonComponent extension.

## Stay on-brand: load the design brief first

**Before generating components, call `getDesignBrief` via MCP.** Use the returned `palette` hex values rather than hardcoding colours, and match the `voice.tone` for any copy. Nova's CSS custom properties (e.g. `var(--color-primary)`) already inherit the brief's palette at runtime — but when you need an explicit hex value (background images, SVG fills, inline gradients), pull it from `getDesignBrief` rather than reusing colours from this skill's examples.

When the brief sets `palette.gradient`, prefer `background: var(--gradient-brand)` over reassembling a `linear-gradient(...)` from the palette. For card/surface backgrounds use `var(--color-surface)` and for heading-weight text use `var(--color-text-strong)` — both follow the tenant's brief (light-mode defaults when unset, brand-specific values when set, e.g. dark cards for dark-mode brands).

## MCP Tools to Use

| Action | Tool |
|--------|------|
| Find existing extensions on an item | `listExtensionsOnItem` |
| Add a new jsonComponent extension | `addExtensionToItem` (extensionName: "jsonComponent") |
| Update the JSON definition | `updateExtensionConfig` — set the `jsonComponent` property |
| Find images for use in components | `listMedia` |
| List available page slots | `listAvailablePageLocations` |

The `jsonComponent` config property must contain a JSON document with `"version": "1.0"` and a `component` field defining the component tree.

## Decision Tree: Which Extension to Use?

1. **Need a hero section?** → Use the `hero` extension (5 curated presets, form-based editing)
2. **Standard pattern, no purpose-built extension?** → Use `jsonComponent` with a starter template
3. **Truly custom layout?** → Use `jsonComponent` freeform

---

## Component Catalog

### Layout Components (have children)

**container** — Layout wrapper with spacing/padding/background. Style: spacing, padding, background, backgroundImage, backgroundOverlay, backgroundPosition, backgroundSize, radius, shadow, maxWidth

**row** — Horizontal flex container. Props: stackOnMobile (bool, default true). Style: gap, align, justify

**column** — Vertical flex, optional width. Props: width [1/4, 1/3, 1/2, 2/3, 3/4, full, auto], mobileWidth [1/2, full, auto]. Style: gap, align, maxWidth

**tabs** — Tabbed content container. Style: variant. Children must be `tab` components.

**tab** — Individual tab panel. Props: label (bindable), active (bool)

**accordion** — Collapsible sections. Props: allowMultiple (bool). Style: variant. Children must be `accordionItem`.

**accordionItem** — Accordion section. Props: title (bindable), expanded (bool)

### Content Components

**heading** — h1-h6 heading. Props: level (1-6, default 2), text (bindable). Style: variant, color, align, transform, tracking

**text** — Paragraph text. Props: text (bindable). Style: variant, color, align, transform, tracking

**link** — Navigation link. Props: text (bindable), href (bindable), action. Style: variant

**button** — Clickable button. Props: text (bindable), icon [arrow-right, arrow-left, chevron-right, chevron-left, shopping-cart, shopping-bag, heart, star, check, plus, minus, search, download, upload, external-link, mail, phone, user, log-in, log-out], iconPosition [left, right], action. Style: variant, color

**image** — Image element. Props: src (bindable), alt (bindable). Style: radius, shadow, width, height

**icon** — Lucide icon. Props: name [arrow-right, arrow-left, arrow-up, arrow-down, chevron-right, chevron-left, chevron-up, chevron-down, menu, x, check, plus, minus, search, settings, edit, trash, copy, download, upload, external-link, shopping-cart, shopping-bag, credit-card, tag, gift, percent, truck, package, mail, phone, message-circle, share, heart, star, thumbs-up, info, alert-circle, check-circle, x-circle, help-circle, user, users, log-in, log-out, home, calendar, clock, map-pin, eye, lock, shield, award, zap, sparkles, rocket], size [xs, sm, md, lg, xl]. Style: color

**blockquote** — Quote with attribution. Props: quote (bindable), author (bindable), role (bindable), avatar (bindable). Style: variant, color, background

### Utility Components

**spacer** — Vertical/horizontal gap. Props: size [xs, sm, md, lg, xl]

**divider** — Horizontal line. Props: thickness [thin, md, thick], lineStyle [solid, dashed, dotted]. Style: color, spacing

**carousel** — Image carousel. Props: images (array), autoPlay (bool), interval (1000-30000ms, default 5000), showIndicators (bool), showControls (bool). Style: radius, shadow

### Product Components

**productGrid** — Responsive product card grid from data source. Props: dataSource, columns (2-6, default 4), mobileColumns (1-3, default 2), showPrice (bool), showDescription (bool), cardVariant [standard, compact, featured]. Style: gap, padding, background, radius

**productCarousel** — Scrollable product strip from data source. Props: dataSource, autoScroll (bool), scrollInterval (ms), showPrice (bool), cardVariant [standard, compact, featured]. Style: gap, padding, background, radius

**productCard** — Single product card from data source at index. Props: dataSource, index (0-100), showImage (bool), showPrice (bool), showDescription (bool), variant [standard, compact, featured]. Style: padding, background, radius, shadow

---

## Semantic Style Tokens

Use these in the `"style"` prop. Never use raw CSS values.

### Typography (variant)
| Token | Use |
|-------|-----|
| heading-xl | Display/hero headings |
| heading-lg | h1 equivalent |
| heading-md | h2 equivalent |
| heading-sm | h3 equivalent |
| body-lg | Lead paragraphs |
| body-md | Standard body text |
| body-sm | Captions, fine print |
| button-primary | Primary button |
| button-secondary | Secondary button |
| link | Link text |

### Colors (color)
Base: primary, secondary, muted, success, error, warning
Light/dark variants: primary-light, primary-dark, secondary-light, secondary-dark, success-light, success-dark, error-light, error-dark, warning-light, warning-dark
Grays: gray-light, gray, gray-dark
Utility: white, black

### Background (background)
Surfaces: surface, surface-light, surface-dark, white, transparent
Primary: primary, primary-light, primary-dark
Status: success, success-light, error, error-light, warning, warning-light
Grays: gray-light, gray, gray-dark

### Spacing/Padding/Gap (spacing, padding, gap)
none (0), xs (4px), sm (8px), md (16px), lg (24px), xl (32px)

### Alignment (align) / Justify (justify)
align: left, center, right, start, end
justify: start, center, end, between, around

### Border Radius (radius)
none, sm, md, lg, full (pill/circle)

### Shadow (shadow)
none, sm, md, lg, xl

### Width/Height (width, height)
full (100%), auto

### Max Width (maxWidth)
prose (65ch), sm (640px), md (768px), lg (1024px), xl (1280px), full, none

### Text Transform (transform)
uppercase, lowercase, capitalize, none

### Letter Spacing (tracking)
tight (-0.025em), normal, wide (0.025em), wider (0.05em), widest (0.1em)

### Background Image (on container)
backgroundImage: URL string (e.g., "media/hero.jpg")
backgroundPosition: center, top, bottom, left, right
backgroundSize: cover, contain, auto
backgroundOverlay: none, light (30%), medium (50%), dark (70%), heavy (85%)

---

## Data Binding

Use `{"$ref": "/context/path", "fallback": "default"}` for dynamic values. Optional `"format"`: capitalize, uppercase, lowercase, trim.

### Context Paths

| Path Prefix | Examples |
|------------|----------|
| /user/* | /user/loginName, /user/personalDetails/firstName, /user/personalDetails/lastName, /user/personalDetails/emailAddress, /user/attributes/{key} |
| /order/* | /order/number, /order/state, /order/price/total (cents), /order/price/currencyCode, /order/lineItems, /order/personalDetails/firstName |
| /siteContext/* | /siteContext/siteName, /siteContext/logoURL, /siteContext/productImagePath, /siteContext/socialAccounts/{platform} |
| /item/* (product) | /item/productName, /item/price (cents), /item/comparePrice, /item/description, /item/miniDescription, /item/manufacturer, /item/seoName, /item/images/0/url, /item/images/0/altText, /item/quantityInStock, /item/attributes/{key} |
| /item/* (collection) | /item/name, /item/description, /item/seoName |
| /item/* (page) | /item/name, /item/attributes/{key} |
| /lastOrder/* | Same as /order/* but for last completed order (order confirmation pages) |

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
| collection-products | collection (seoName, required), limit (int, default 12), offset (int, default 0), inStockOnly (bool, default false) | Products from a collection |
| recently-viewed | limit (int, default 10) | User's recently viewed products (requires login) |

Reference in components via `"dataSource": "mySource"` prop. Also accessible via `$ref`: `/data/{sourceName}/0/productName`.

---

## Media

For media library images, use relative paths like `"media/hero-banner.jpg"` — the system prepends the site's productImagePath automatically.

For product images via data binding: `{"$ref": "/item/images/0/url"}`

---

## Layout Modes

Set `"layout"` at root level:
- `"fluid"` (default) — Full-width, edge-to-edge. Use for hero sections.
- `"contained"` — Constrained to page container. Use for standard content.

For full-width background with constrained content: use `"layout": "fluid"` with a container child that has `"maxWidth": "prose"` or similar.

## Rules

1. Wrap in `{"version": "1.0", "component": {...}}`
2. Every component needs `"type"` and optionally `"key"`, `"props"`, `"children"`
3. Use ONLY semantic tokens — never raw CSS values
4. For navigation: `"action": {"type": "navigate", "href": "/path"}`
5. Output valid JSON only — no markdown, no explanations
6. Prices are in cents
7. Use meaningful fallback values for data bindings

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
