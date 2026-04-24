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
| Find existing extensions on an item | `listExtensionsOnItem` |
| Add a hero extension | `addExtensionToItem` (extensionName: "hero") |
| Update hero configuration | `updateExtensionConfig` — set the `heroConfig` property |
| Find images for backgrounds | `listMedia` |

## 5 Preset Layouts

| Preset Key | Description | Slots |
|------------|-------------|-------|
| `centered` | Full-width BG image with centered text + CTA overlay | title, subtitle, cta |
| `split-image` | Text left, image right (CSS grid two-column) | title, subtitle, cta, heroImage |
| `video-bg` | Centered text over autoplay muted looping video | title, subtitle, cta |
| `gradient` | BG image with angled CSS gradient, left-aligned text | title, subtitle, cta |
| `minimal` | Clean white surface with centered large typography | title, subtitle, cta |

## Configuration Format

The `heroConfig` property stores a JSON string:

```json
{
  "preset": "centered",
  "backgroundImage": "/media/hero-bg.jpg",
  "backgroundOverlay": "dark",
  "slots": {
    "title": {"type": "heading", "props": {"text": "Welcome", "level": 1, "style": {"variant": "heading-xl", "color": "white"}}},
    "subtitle": {"type": "text", "props": {"text": "Discover our collection", "style": {"variant": "body-lg", "color": "white"}}},
    "cta": {"type": "button", "props": {"text": "Shop Now", "action": {"type": "navigate", "href": "/collections"}, "style": {"variant": "button-primary"}}}
  }
}
```

## Top-Level Properties

| Property | Type | Presets | Description |
|----------|------|---------|-------------|
| `preset` | string | All | One of: `centered`, `split-image`, `video-bg`, `gradient`, `minimal` |
| `backgroundImage` | string | centered, gradient | URL to background image |
| `backgroundOverlay` | string | centered, video-bg | `"dark"` (60%), `"medium"` (40%), `"light"` (20%) |
| `videoUrl` | string | video-bg | URL to MP4 video file |
| `slots` | object | All | Map of slot name → JSON component definition |

## Slot Component Types

Each slot value is a standard JSON component definition — the same types and style tokens used by the `jsonComponent` extension:

| Slot | Component Type | Key Props |
|------|---------------|-----------|
| `title` | `heading` | `text`, `level` (1-6), `style.variant`, `style.color` |
| `subtitle` | `text` | `text`, `style.variant`, `style.color` |
| `cta` | `button` | `text`, `action.type` ("navigate"), `action.href`, `style.variant` |
| `heroImage` | `image` | `src`, `alt` |

### Style Tokens for Slots

**Typography variants:** `heading-xl`, `heading-lg`, `heading-md`, `heading-sm`, `body-lg`, `body-md`, `body-sm`

**Colors:** `white`, `primary`, `muted`, `black`, `secondary`, `gray`, `gray-light`, `gray-dark`

**Button variants:** `button-primary`, `button-secondary`

## Default Page Location

`bannerSection` — heroes render above the main content area by default.

## Examples

### Centered hero with background image
```json
{
  "preset": "centered",
  "backgroundImage": "media/hero-banner.jpg",
  "backgroundOverlay": "dark",
  "slots": {
    "title": {"type": "heading", "props": {"text": "Summer Sale", "level": 1, "style": {"variant": "heading-xl", "color": "white"}}},
    "subtitle": {"type": "text", "props": {"text": "Up to 50% off selected items", "style": {"variant": "body-lg", "color": "white"}}},
    "cta": {"type": "button", "props": {"text": "Shop Sale", "action": {"type": "navigate", "href": "/collections/sale"}, "style": {"variant": "button-primary"}}}
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
    "cta": {"type": "button", "props": {"text": "Browse", "action": {"type": "navigate", "href": "/collections/new"}, "style": {"variant": "button-primary"}}},
    "heroImage": {"type": "image", "props": {"src": "media/new-arrivals.jpg", "alt": "New arrivals collection"}}
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
    "cta": {"type": "button", "props": {"text": "Explore", "action": {"type": "navigate", "href": "/collections"}, "style": {"variant": "button-secondary"}}}
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
    "cta": {"type": "button", "props": {"text": "Learn More", "action": {"type": "navigate", "href": "/about"}, "style": {"variant": "button-primary"}}}
  }
}
```

## Decision Tree

1. **Need a hero section?** → Use this extension (not freeform JSONComponent)
2. **Need a layout beyond the 5 presets?** → Use `jsonComponent` with the hero as inspiration
3. **Slots use the same JSON component syntax** as `jsonComponent` — same types, same style tokens