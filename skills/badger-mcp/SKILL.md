---
name: badger-mcp
description: Guide to Badger Commerce MCP tools — domain model, common workflows, and progressive tool reference for managing products, collections, pages, extensions, and more
user-invocable: true
allowed-tools: Read, Grep, Glob
---

# Badger Commerce MCP Tools Guide

This skill is your entry point for working with the Badger Commerce platform via MCP tools. It explains how the domain model fits together and provides workflow sequences for common operations.

## Domain Model

### Products, Collections, and Pages

- **Products** are the core catalogue items (name, SKU, price, images, description). Products belong to **Collections** (many-to-many) — a product can appear in multiple collections.
- **Collections** organize products into browsable categories. They form a hierarchy via parent collections (e.g., Clothing → Tops → T-Shirts). Collections have their own SEO name for URL routing.
- **Pages** are standalone CMS content (about, contact, terms, blog posts). Blog posts are child pages of a blog page. Pages have SEO names and can carry extensions just like products and collections.

### Extensions

Extensions are the **content building blocks** of the platform. They attach to products, collections, pages, or stereotypes and render in named **slots** on the page (e.g., `body`, `sidebar`, `bannerSection`, `footer`).

- Each extension targets a specific slot and has its own configuration properties
- Multiple extensions can render in the same slot — they are ordered by render priority
- Use `extensions` listAvailable to see all registered extensions
- Use `extensions` getSchema to inspect an extension's configurable fields
- Use `extensions` getPromptSection to fetch detailed AI guidance for specific extensions
- Use `extensionConfig` to add, remove, update, or list extensions on any item

**Key extensions with dedicated skills:**
- `jsonComponent` — freeform JSON component builder (see `json-components` skill)
- `hero` — curated hero section with 5 preset layouts (see `hero` skill)

### Stereotypes

Stereotypes are **reusable templates** that pre-configure a bundle of extensions. Assign a stereotype to a product, collection, or page and it inherits all the stereotype's extensions. Item-level extensions override stereotype defaults for the same extension in the same slot.

Use stereotypes when you want consistent layouts across multiple items (e.g., all products in a category share the same extension setup).

### Taxonomy

A taxonomy provides **structured product attributes** via a hierarchy of levels and typed attributes. A single active taxonomy per site defines the classification scheme.

- **Levels**: form a tree (e.g., Electronics → Computers → Laptops)
- **Attributes**: defined per level (e.g., screen size, RAM, processor) with types: STRING, NUMBER, BOOLEAN, SINGLE_SELECT, MULTI_SELECT
- **Assignment**: products are assigned to a taxonomy level and populate its attributes
- **Completeness**: the system scores how many mandatory attributes are filled (0-100%)

**All but the simplest shops should define a taxonomy** — it drives structured product pages, faceted search/filtering, and data completeness tracking. See the `taxonomy` skill for design patterns.

---

## Tool Map

### Content Management (most common)
| Tool | Key Actions |
|------|-------------|
| `products` | list, get, create, update, delete, addToCollection, removeFromCollection |
| `collections` | list, get, create, update, delete, listChildren |
| `pages` | list, get, create, update, delete, listChildren |
| `media` | list, get, update |
| `menus` | list, get, create, delete |

### Extensions
| Tool | Key Actions |
|------|-------------|
| `extensions` | listAvailable, getSchema, getPromptSection, listPageLocations |
| `extensionConfig` | listOnItem, add, remove, update |

### Commerce Operations
| Tool | Key Actions |
|------|-------------|
| `orders` | list, get, countNew *(read-only)* |
| `inventory` | get, getMulti, set, adjust |
| `variants` | listOnProduct, get, create, update, delete |
| `variantGroups` | list, create, delete, getStrategy |
| `deliveryOptions` | list, get, create, update, delete |
| `promotions` | list, get, create, update, delete |

### Analytics (read-only)
| Tool | Key Actions |
|------|-------------|
| `statistics` | salesSummary, topProducts, customerStats, revenueBreakdown, giftAid, dashboard |
| `siteTraffic` | timeSeries, topPages |

### Advanced (use when needed)
| Tool | When to use |
|------|-------------|
| `stereotypes` | Setting up reusable product/page templates with pre-configured extensions |
| `taxonomies` | Viewing/managing taxonomy hierarchies |
| `productTaxonomy` | Assigning products to taxonomy levels, populating structured attributes |
| `subscriptions` | Managing recurring payments (list, pause, resume, cancel) |
| `giftCards` | Looking up gift cards and checking balances |
| `pipelines` | Debugging order processing flow, toggling processors *(admin only)* |
| `siteConfig` | Changing site-level settings like currency, feature flags *(admin only)* |

---

## Common Workflows

### Set up a new product
1. `products` create — name, seoName, description, price (in minor units e.g. pence)
2. `collections` list — find the target collection
3. `products` addToCollection — assign product to collection
4. `productTaxonomy` assign — assign to a taxonomy level (if taxonomy exists)
5. `productTaxonomy` updateAttributes — populate structured attributes
6. `extensionConfig` add — add content extensions (hero, jsonComponent, etc.)
7. `extensionConfig` update — configure extension properties

### Add content to a page
1. `pages` get — fetch the page by seoName
2. `extensions` listPageLocations — see available slots (body, sidebar, bannerSection, etc.)
3. `extensionConfig` add — add extension to a slot (extensionName + pageLocation)
4. `extensionConfig` update — set configuration properties

*For jsonComponent, the `json-components` skill has the full component catalog and style tokens.*
*For hero sections, the `hero` skill has all 5 preset layouts.*

### Set up a collection page
1. `collections` create — name, seoName, optional parentCollectionId
2. `products` addToCollection — add products to the collection
3. `extensionConfig` add — add a hero banner (pageLocation: "bannerSection")
4. `extensionConfig` add — add product grid or other body content

### Configure structured product data
1. `taxonomies` getActive — check if a taxonomy exists
2. If none: design and create a taxonomy (see `taxonomy` skill for patterns)
3. `productTaxonomy` assign — assign each product to its taxonomy level
4. `productTaxonomy` updateAttributes — populate attribute values
5. `productTaxonomy` getAttributes — check completeness score

### Set up product variants
1. `variantGroups` list — check existing groups
2. `variantGroups` create — create groups (e.g., code: "colour", displayName: "Colour")
3. `variants` create — add variants with attribute values (e.g., `{"colour": "Blue", "size": "Large"}`)

### Review business performance
1. `statistics` dashboard — current month overview in one call
2. `statistics` topProducts / salesSummary — drill into specifics
3. `siteTraffic` timeSeries — traffic trends over time

---

## Key Conventions

- **Prices** are in minor currency units (pence/cents). £19.99 = `1999`
- **SEO names** are URL slugs (e.g., `summer-collection`). Most tools accept either SEO name or MongoDB ID for lookups.
- **Pagination**: list actions support `page` (0-based) and `size` (default 20)
- **Extension guidance**: use `extensions` getPromptSection for on-demand guidance on any extension, or install `badger-skills` for pre-loaded skills on common extensions
