---
name: hero
description: Add and configure Hero section extensions on Badger Commerce pages with 5 preset layouts (Centered, Split Image, Video Background, Gradient Overlay, Minimal) via MCP tools
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Hero Section Extension

The Hero extension provides curated hero section layouts with form-based editing and full JSON customisation. It leverages the existing JSON component rendering engine for content slots.

## Stay on-brand: load the design brief first

**Before generating hero copy or picking colours, call `getDesignBrief` via MCP.** Use the returned `palette` hex values for button/accent colours and the `voice.tone` to shape headline and subtitle copy. Any colours you see in the examples below are illustrative — don't propagate them into output without checking the brief.

When the tenant has set `palette.gradient`, the compiled CSS exposes `--gradient-brand` — prefer `background: var(--gradient-brand)` on hero backgrounds over recreating your own linear-gradient from the individual palette colours. For dark-mode-leaning brands the brief may also set `palette.surface` (card backgrounds) and `palette.textStrong` (heading colour); the CSS compiler maps these to `--color-surface` and `--color-text-strong` automatically, so sticking to those Nova tokens is enough — no custom overrides needed.

## MCP Tools to Use

| Action | Tool |
|--------|------|
| Find existing extensions on an item | `extensionConfig` (itemType, itemId) |
| List the slots the item renders | `extensions` listPageLocations (itemType, itemId) |
| Add a hero extension | `manageExtensionConfig` add (extensionName: "hero", pageLocation: "bannerSection") |
| Update hero configuration | `manageExtensionConfig` update (extensionConfigId) — set the `heroConfig` property |
| Add a photo the merchant gives you | `manageMedia` import (url or base64 content) |
| Find images for backgrounds | `media` list |

## 5 Preset Layouts

| Preset Key | Description | Slots |
|------------|-------------|-------|
| `centered` | Full-width BG image with centered text + CTA overlay | title, subtitle, cta |
| `split-image` | Text left, image right (CSS grid two-column) | title, subtitle, cta, heroImage |
| `video-bg` | Centered text over autoplay muted looping video | title, subtitle, cta |
| `gradient` | BG image with angled CSS gradient, left-aligned text | title, subtitle, cta |
| `minimal` | Clean white surface with centered large typography | title, subtitle, cta |

Every preset also takes the optional slots `eyebrow`, `secondaryCta` and `stats` (see Slot Component Types). Hero templates exist only for the Nova family of themes (Nova, Brock, Ridge, Atelier, ...); the legacy `bootstrap` theme has none.

## Configuration Format

The `heroConfig` property stores a JSON string (send a string or an object). Unlike jsonComponent it is only checked for JSON syntax: a slot that doesn't parse is dropped silently, so check the page after saving.

```json
{
  "preset": "centered",
  "backgroundImage": "//images.bdgr.co.uk/acme/media/hero-bg.jpg",
  "backgroundOverlay": "dark",
  "slots": {
    "title": {"type": "heading", "props": {"text": "Welcome", "level": 1, "style": {"variant": "heading-xl", "color": "white"}}},
    "subtitle": {"type": "text", "props": {"text": "Discover our collection", "style": {"variant": "body-lg", "color": "white"}}},
    "cta": {"type": "button", "props": {"text": "Shop Now", "action": {"type": "navigate", "href": "/collection/new"}, "style": {"variant": "button-primary"}}}
  }
}
```

## Top-Level Properties

| Property | Type | Presets | Description |
|----------|------|---------|-------------|
| `preset` | string | All | One of: `centered`, `split-image`, `video-bg`, `gradient`, `minimal` |
| `backgroundImage` | string | centered, gradient | Background image URL, used verbatim (see Images). Without one, both fall back to a darkened brand gradient |
| `backgroundOverlay` | string | centered, video-bg | `"dark"` (default), `"medium"` or `"light"`: a darkening gradient, strongest for dark. Any other value draws no overlay |
| `videoUrl` | string | video-bg | URL to MP4 video file |
| `backgroundEffect` | string | gradient, minimal, split-image | Animated canvas behind the copy: `"none"` (default), `"constellation"`, `"flowField"`, `"terrain"`, `"globe"`, `"aurora"`, `"gridGlow"`. Ignored on other presets |
| `backgroundEffectIntensity` | string | as above | The level: `"subtle"`, `"normal"`, `"present"` (default), `"bold"` or `"showpiece"` |
| `backgroundEffectTuning` | object | as above | Optional overrides of single settings of the level (see Animated Background Effects) |
| `globe` | object | as above, with `backgroundEffect` `"globe"` | `{"home": {"lat", "lon"}, "destinations": [{"lat", "lon"}, ...]}`: where the shipping arcs start and end |
| `entrance` | string | All | How the copy arrives on load: `"auto"` (default), `"none"`, `"fadeUp"`, `"rise"`, `"blur"`, `"wipe"`, `"settle"`, `"words"` (see Entrance) |
| `entranceTuning` | object | All | Optional `travel`, `duration`, `stagger` overrides of the entrance's timing |
| `height` | string | All | `"tall"` (default, about 70vh: a landing hero) or `"compact"` (as tall as its copy). Use `compact` for every hero except the home page's |
| `contentAlign` | string | gradient, split-image | `"edge"` (default: the copy starts a fixed inset from the window edge) or `"container"` (the copy lines up with the page's content column, under the logo). Wide screens (1024px+) only; the centred presets ignore it |
| `slots` | object | All | Map of slot name → JSON component definition |

## Animated Background Effects

`backgroundEffect` adds a Stripe/Accenture-style ambient canvas behind the hero copy. It takes its colours from the Design Brief tokens (`--color-primary`, `--color-secondary`, `--color-accent`, `--color-on-dark-accent`), so don't pass colours. It draws a still frame for reduced-motion visitors, pauses offscreen, and is masked behind the text so the copy stays legible.

| Effect | Look and pointer response | Good for |
|--------|------|----------|
| `terrain` | A wireframe landscape rolling towards the viewer; the ground rises into a hill under the pointer and a click sends a ripple across it | Statement home pages: outdoor, travel, adventure, tech |
| `globe` | A dotted globe with shipping arcs from the shop to where it ships; visitors drag to spin it | Shops that ship widely or internationally (set `globe`) |
| `constellation` | Drifting points joined by lines; the pointer scatters them | Consultancy, tech, "network" brands |
| `flowField` | Particle trails following a noise field; they swirl round the pointer | Creative, organic, calm brands |
| `aurora` | Large soft fields of brand colour drifting slowly; they lean towards the pointer | Calm or premium brands, and behind longer copy |
| `gridGlow` | Perspective dot grid with a travelling glow that slides sideways to the pointer | The plainest; prefer `terrain` |

Use it on at most one hero per page, usually the home page's, and pair it with `gradient` for a dark, premium look. A spinning globe on its own is a common sight on software sites; only use `globe` when the arcs say something true about the shop (it ships beyond its own town).

**Level.** `backgroundEffectIntensity` sets every setting at once. `present` (the default) is clearly visible without competing with the copy. Use `bold` or `showpiece` for a short headline with room around it; behind a long subtitle keep `present` or lower. `subtle` and `normal` are the quieter original levels, for a brief that asks for restraint.

**Tuning.** Set the level first, then override only what it doesn't give you in `backgroundEffectTuning`. Every key is optional and clamped to its range; a key left out follows the level.

| Key | Range | Effect |
|-----|-------|--------|
| `strength` | 0.2–4 | How visible the marks are (opacity multiple) |
| `speed` | 0.25–4 | How fast it moves |
| `density` | 0.4–2.5 | How many points, trails or grid lines (and globe dots) |
| `size` | 0.5–3.5 | Dot size and line weight; the globe's size |
| `pointer` | 0–4 | How strongly it reacts to the pointer; `0` ignores it |
| `behindCopy` | 0–1 | How much shows behind the text; lower it for long copy |
| `linkDistance` | 0.5–2.2 | `constellation` only: how far apart points are still joined |
| `trail` | 0.3–4 | `flowField` only: how long the trails linger |
| `relief` | 0–3 | `terrain` only: how tall the hills are |

**Globe arcs.** `globe.home` is where the shop is; each `destinations` entry is a place it ships to (up to 24, as decimal degrees). With a home and no destinations, arcs go to major cities worldwide; with no home, the globe spins without arcs. Locations carry no coordinates, so look the shop's town up yourself.

```json
{"preset": "gradient", "backgroundEffect": "terrain", "backgroundEffectIntensity": "bold",
 "backgroundEffectTuning": {"relief": 1.6}, "slots": {...}}
```

```json
{"preset": "gradient", "backgroundEffect": "globe",
 "globe": {"home": {"lat": 53.38, "lon": -1.47},
           "destinations": [{"lat": 51.51, "lon": -0.13}, {"lat": 48.86, "lon": 2.35}, {"lat": 40.71, "lon": -74.01}]},
 "slots": {...}}
```

## Entrance

`entrance` animates the copy in when the page loads, on every preset: the eyebrow, title, text, buttons and stats arrive one after another. Reduced-motion visitors see the copy at once.

| Value | Look |
|-------|------|
| `auto` (default) | `fadeUp` on `centered`, none on the other presets |
| `none` | No entrance |
| `fadeUp` | A short fade up (24px) |
| `rise` | A longer rise with a soft landing; a good default for a home page |
| `blur` | Each line sharpens into place |
| `wipe` | Each line is revealed from below |
| `settle` | Each line scales down slightly into place |
| `words` | The title arrives word by word; best with a short title |

`entranceTuning` overrides the style's timing: `travel` 0–140 (pixels each line moves), `duration` 0.2–2.5 (seconds per line), `stagger` 0–0.5 (seconds between lines). Pair a strong effect with a calm entrance (`rise`, `fadeUp`) rather than two loud ones.

```json
{"preset": "minimal", "entrance": "words", "entranceTuning": {"stagger": 0.22}, "slots": {...}}
```

## Slot Component Types

Each slot value is a standard JSON component definition — the same types and style tokens used by the `jsonComponent` extension:

| Slot | Component Type | Key Props |
|------|---------------|-----------|
| `title` | `heading` | `text`, `level` (1-6), `style.variant`, `style.color` |
| `subtitle` | `text` | `text`, `style.variant`, `style.color` |
| `cta` | `button` | `text`, `action.type` ("navigate"), `action.href`, `style.variant` |
| `heroImage` | `image` | `src`, `alt` (split-image only; a placeholder shows when absent) |
| `eyebrow` | `text` | A short line above the title. `style.variant` `overline` or `overline-pill` suits it |
| `secondaryCta` | `button` | A second button beside `cta` (same props); usually `button-secondary` |
| `stats` | any component, usually `row` | A strip of figures under the buttons, with a divider above. Convention: a `row` of `column`s, each a `heading` (the figure) plus a `text` (its label). `children` sits beside `props`, not inside it |

centered, gradient and video-bg are dark surfaces, so headings and text turn light by themselves; split-image and minimal are light. Slots get no data sources, so productGrid, productCarousel, productCard and `/data/...` bindings render empty in a hero.

Optional slots render only when present; omit a slot to hide it. `cta` and `secondaryCta` sit together in one flex row, centred or left-aligned to match the preset. Themes restyle the `nova-hero__eyebrow`, `nova-hero__actions` and `nova-hero__stats` wrappers (Ridge, for example, turns stats columns into divided cells). On Ridge the home hero is the `gradient` preset with `eyebrow`, `secondaryCta`, `stats` and `contentAlign: "container"`: it gets a purple wash, a red accent bar and a pulsing dot before the eyebrow. A title is one heading, so there are no accent words.

### Style Tokens for Slots

**Typography variants:** `heading-xl`, `heading-lg`, `heading-md`, `heading-sm`, `body-lg`, `body-md`, `body-sm`, `display-xl`, `display-lg`, `display-md`, `subheadline`, `overline`, `overline-pill`

**Colors:** `white`, `primary`, `muted`, `black`, `secondary`, `gray`, `gray-light`, `gray-dark`. On Nova `secondary` is a mid grey, not the brief's secondary colour.

**Button variants:** `button-primary`, `button-secondary`

### Button Links

Use the `url` the MCP tools return for the target (`collections`, `pages`, `products` or `merchandisingTrees` node), as a site-relative path. Links in a hero aren't rewritten for you: on a category-tree site (`category-navigation-source` MERCH_TREE), link a category as `/c/...`, not `/collection/...`. A replaced collection's old URL only redirects, and a hidden one is a 404.

### Images

`backgroundImage` and `heroImage` src are used verbatim; no base path is added. Use `shop.productImagePath` from `getDesignBrief` + `/` + the media `url` from the `media` tool (e.g. `//images.bdgr.co.uk/acme/media/hero.jpg`), or a full `https://` URL. A relative `media/...` path won't load.

## Page Location

`bannerSection`, above the main content. The admin defaults to it, but `manageExtensionConfig` add needs it passed as `pageLocation`.

## Examples

### Centered hero with background image
```json
{
  "preset": "centered",
  "backgroundImage": "//images.bdgr.co.uk/acme/media/hero-banner.jpg",
  "backgroundOverlay": "dark",
  "slots": {
    "title": {"type": "heading", "props": {"text": "Summer Sale", "level": 1, "style": {"variant": "heading-xl", "color": "white"}}},
    "subtitle": {"type": "text", "props": {"text": "Up to 50% off selected items", "style": {"variant": "body-lg", "color": "white"}}},
    "cta": {"type": "button", "props": {"text": "Shop Sale", "action": {"type": "navigate", "href": "/collection/sale"}, "style": {"variant": "button-primary"}}}
  }
}
```

### Split image hero
```json
{
  "preset": "split-image",
  "slots": {
    "title": {"type": "heading", "props": {"text": "New Arrivals", "level": 1, "style": {"variant": "heading-xl"}}},
    "subtitle": {"type": "text", "props": {"text": "Handpicked for quality and style", "style": {"variant": "body-lg", "color": "muted"}}},
    "cta": {"type": "button", "props": {"text": "Browse", "action": {"type": "navigate", "href": "/collection/new"}, "style": {"variant": "button-primary"}}},
    "heroImage": {"type": "image", "props": {"src": "//images.bdgr.co.uk/acme/media/new-arrivals.jpg", "alt": "New arrivals collection"}}
  }
}
```

### Minimal hero (no background)
```json
{
  "preset": "minimal",
  "slots": {
    "title": {"type": "heading", "props": {"text": "Less is More", "level": 1, "style": {"variant": "heading-xl"}}},
    "subtitle": {"type": "text", "props": {"text": "Simple, thoughtful design", "style": {"variant": "body-lg", "color": "muted"}}},
    "cta": {"type": "button", "props": {"text": "Explore", "action": {"type": "navigate", "href": "/collection/new"}, "style": {"variant": "button-secondary"}}}
  }
}
```

### Eyebrow, two buttons and stats
```json
{
  "preset": "gradient",
  "slots": {
    "eyebrow": {"type": "text", "props": {"text": "Volunteer-run since 1962", "style": {"variant": "overline", "color": "white"}}},
    "title": {"type": "heading", "props": {"text": "On call for the hills, every day of the year", "level": 1, "style": {"variant": "heading-xl", "color": "white"}}},
    "subtitle": {"type": "text", "props": {"text": "Your support keeps the team equipped", "style": {"variant": "body-lg", "color": "white"}}},
    "cta": {"type": "button", "props": {"text": "Donate", "action": {"type": "navigate", "href": "/p/donate"}, "style": {"variant": "button-primary"}}},
    "secondaryCta": {"type": "button", "props": {"text": "Our work", "action": {"type": "navigate", "href": "/p/about-us"}, "style": {"variant": "button-secondary"}}},
    "stats": {"type": "row", "props": {"style": {"gap": "lg"}}, "children": [
      {"type": "column", "children": [
        {"type": "heading", "props": {"text": "140+", "level": 3, "style": {"variant": "heading-md", "color": "white"}}},
        {"type": "text", "props": {"text": "Call-outs a year", "style": {"variant": "body-sm", "color": "white"}}}]},
      {"type": "column", "children": [
        {"type": "heading", "props": {"text": "24/7", "level": 3, "style": {"variant": "heading-md", "color": "white"}}},
        {"type": "text", "props": {"text": "Volunteers on call", "style": {"variant": "body-sm", "color": "white"}}}]}
    ]}
  }
}
```

### Video background hero
```json
{
  "preset": "video-bg",
  "backgroundOverlay": "dark",
  "videoUrl": "https://example.com/hero-video.mp4",
  "slots": {
    "title": {"type": "heading", "props": {"text": "Experience the Difference", "level": 1, "style": {"variant": "heading-xl", "color": "white"}}},
    "subtitle": {"type": "text", "props": {"text": "Watch how our products are crafted", "style": {"variant": "body-lg", "color": "white"}}},
    "cta": {"type": "button", "props": {"text": "Learn More", "action": {"type": "navigate", "href": "/p/about-us"}, "style": {"variant": "button-primary"}}}
  }
}
```

## Decision Tree

1. **Need a hero section?** → Use this extension (not freeform JSONComponent)
2. **An appeal band, CTA strip or sidebar panel further down the page?** → `callToAction` (form-based, no JSON; see the `badger-mcp` skill's content page recipes)
3. **Need a layout beyond the 5 presets?** → Use `jsonComponent` with the hero as inspiration
4. **Slots use the same JSON component syntax** as `jsonComponent` — same types, same style tokens