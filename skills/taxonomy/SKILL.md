---
name: taxonomy
description: Design and construct product taxonomies for Badger Commerce — common patterns for apparel, food, retail, and charity shops with attribute best practices, and building category trees from them
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Product Taxonomy Design

This skill provides guidance on designing and constructing product taxonomies for typical e-commerce use cases.

## When to Use Taxonomy

Any shop with more than a handful of products benefits from a taxonomy. It provides:

- **Structured product data** — consistent attributes across similar products
- **Faceted search/filtering** — customers can filter by size, colour, material, etc.
- **Completeness tracking** — the system scores how many mandatory attributes are filled (0-100%), helping identify products missing key data
- **Structured product pages** — attributes display consistently on product detail pages

Start simple — you can always add levels and attributes as the catalogue grows.

## Common Taxonomy Patterns

### Apparel Store

```
Clothing
├── Tops (material, fit, neckline, sleeve-length, care-instructions)
├── Bottoms (material, fit, leg-style, waist-type, care-instructions)
├── Outerwear (material, fill-type, water-resistance, warmth-rating)
├── Dresses (material, fit, length, occasion)
└── Accessories (material, closure-type)
```

**Key attributes:**
- `material` — SINGLE_SELECT: Cotton, Polyester, Wool, Silk, Linen, Blend
- `fit` — SINGLE_SELECT: Slim, Regular, Relaxed, Oversized
- `care-instructions` — STRING: free-text washing/drying instructions
- `season` — MULTI_SELECT: Spring, Summer, Autumn, Winter

### Food & Drink

```
Products
├── Hot Drinks (origin, roast-level, caffeine-content)
├── Cold Drinks (volume, carbonated, sugar-content)
├── Snacks (weight, serving-size)
├── Fresh (shelf-life, storage-requirements)
└── Pantry (weight, shelf-life)
```

**Key attributes:**
- `allergens` — MULTI_SELECT: Gluten, Dairy, Nuts, Soy, Eggs, Shellfish, Sesame
- `dietary` — MULTI_SELECT: Vegan, Vegetarian, Gluten-Free, Organic, Halal, Kosher
- `weight` — STRING: e.g., "250g", "1kg"
- `origin` — STRING: country or region of origin

### General Retail

```
Department
├── Electronics
│   ├── Computers (processor, ram, storage, screen-size)
│   ├── Phones (screen-size, storage, battery-capacity)
│   └── Accessories (compatibility, connector-type)
├── Home & Garden
│   ├── Furniture (material, dimensions, assembly-required)
│   └── Decor (material, dimensions, colour)
└── Sports
    ├── Equipment (sport, skill-level, material)
    └── Clothing (sport, material, fit)
```

**Key attributes:**
- `brand` — STRING: manufacturer/brand name
- `warranty` — STRING: warranty period and terms
- `dimensions` — STRING: L × W × H format
- `weight` — STRING: product weight

### Charity / Non-Profit Shop

```
Items
├── Donated Goods (condition, donor-type)
├── New Stock (supplier, rrp)
├── Crafted Items (maker, materials-used, production-time)
└── Digital (format, access-period)
```

**Key attributes:**
- `condition` — SINGLE_SELECT: New, Like New, Good, Fair
- `gift-aid-eligible` — BOOLEAN: whether Gift Aid can be claimed
- `donor-type` — SINGLE_SELECT: Individual, Corporate, Estate

## Attribute Best Practices

### Choosing Attribute Types
- **SINGLE_SELECT** for controlled vocabularies where exactly one value applies (sizes, colours, conditions)
- **MULTI_SELECT** for tags where multiple values can apply (allergens, dietary, features, seasons)
- **STRING** for free-text values (descriptions, care instructions, dimensions)
- **NUMBER** for measurable quantities (weight in grams, battery capacity in mAh)
- **BOOLEAN** for yes/no flags (assembly required, gift aid eligible, waterproof)

### Validation Rules
- Mark critical attributes as `mandatory` — this drives completeness scoring
- Set `minLength`/`maxLength` for STRING attributes to enforce consistency
- Set `minValue`/`maxValue` for NUMBER attributes (e.g., weight > 0)
- Use `pattern` for structured formats (e.g., dimensions: `^\d+\s*[x×]\s*\d+\s*[x×]\s*\d+\s*(mm|cm|m)$`)

### Display Settings
- Set `displayOnProductPage: true` for attributes customers care about
- Use clear `displayLabel` values (e.g., "Care Instructions" not "care-instructions")
- Use `helpText` to guide admin users filling in values (e.g., "Enter dimensions as L × W × H in cm")
- Attributes appear on product pages in the order they sit on their level: add them in the order you want them shown

### Design Principles
- **Start broad, refine later** — begin with 2-3 levels and expand as needed
- **Keep attributes at the right level** — put shared attributes on parent levels, specific ones on leaf levels
- **Don't over-attribute** — 5-8 attributes per level is usually sufficient
- **Use SELECT types over STRING where possible** — they enable filtering and ensure data consistency

## Workflow

1. `taxonomies` getActive — check if a taxonomy already exists (`taxonomies` list shows them all; `active` marks the one in use)
2. Design your hierarchy on paper/notes first — levels, attributes, and allowed values
3. Build it with `manageTaxonomies`:
   - `create` (name, description) — returns the taxonomyId
   - `addLevel` (taxonomyId, name, parentLevelId; leave parentLevelId out for a top-level level; optional `position`, 0-based, default last) — returns the new levelId
   - `addAttribute` (taxonomyId, levelId, `attribute`: `{name, type, mandatory, allowedValues, helpText, displayLabel, displayOnProductPage, facetDisplay, unit, swatches, minValue, maxValue, minLength, maxLength, pattern, errorMessage}`) — returns the new attributeId. Ids are made from the name; give the same `attributeId` on sibling levels to make them one filter
   - `taxonomies` get — check the whole tree
   - `setActive` — the first activation changes no product. Switching from another active taxonomy clears every product's classification: run it with `dryRun: true`, tell the user, then `confirm: true` (which also needs the `mcp:admin` scope). `delete` needs `mcp:admin` too and refuses the active taxonomy
4. `manageProductTaxonomy` assign — assign products to their appropriate levels
5. `manageProductTaxonomy` updateAttributes — populate attribute values for each product
6. `productTaxonomy` (skuId) — check completeness scores and find gaps. updateAttributes saves even invalid values, returning `valid: false` and `validationErrors`, and refuses attributes that don't apply at the product's level
7. Iterate: add attributes as new product categories emerge (`addAttribute`, or `updateAttribute` with `addValues` / `removeValues` for the allowed values)
8. Filters: `manageProductTaxonomy` setFilterDisplay (attributeId, `facetDisplay` LIST, SWATCH or RANGE, `unit` for a range, `swatches` value → #hex for values not named after a colour; no skuId) sets how an attribute filters listings. SWATCH doesn't suit NUMBER or BOOLEAN, RANGE doesn't suit BOOLEAN. Variant options (e.g. a size group) count towards a filter through `manageVariantGroups` setFilterMapping (code, attributeId, attributeValues option → filter value)

**Changing a taxonomy that's in use.** Renaming is always safe (ids never change). Moving a level to another parent, removing levels, removing attributes or allowed values, and changing an attribute's type can leave data behind on products. Run the edit with `dryRun: true` first: its `impact` lists what products would lose. If it isn't empty, check with the user, then repeat with `confirm: true`. `updateAttribute` only changes the keys you send; send a key as `null` to clear it.

## Building a Category Tree From the Taxonomy

On a site with a merchandising tree (the `/c/` category pages; see the `category-trees` skill), the taxonomy is what most categories are built from. A product assigned to a level shows up in every category projected from that level or one above it, with no collection to add it to. That changes a few design choices.

### Levels become categories
- Back each browsable category with a `taxonomyProjection`: `{"type": "taxonomyProjection", "taxonomyLevelId": "<levelId>"}`. It lists products assigned to that level **or any level beneath it**, so a parent category fills itself from its children.
- `taxonomies` get gives the `levelId`s, and each attribute's `attributeId` and `allowedValues`.
- The tree doesn't have to mirror the taxonomy one to one. A heading with no products of its own is a `group` category, and a level too broad for shoppers can be split with attribute filters (below).

### Attribute filters: subcategories and cross-cutting pages
`attributeFilters` narrow a projection to attribute values: `[{"attributeId": "application", "values": ["Drywall"]}]`. Several filters must all match; within one filter, any listed value does. Values match the stored value exactly, so filter on SINGLE_SELECT or MULTI_SELECT attributes rather than free-text STRING ones.

Two uses:
- **Subcategories that split a level.** "Wood Screws" and "Drywall Screws" are both the Screws level, filtered by `application`. Make the attribute mandatory, so that every product lands in one of them.
- **Cross-cutting pages.** "Stainless Steel Fixings" (Fixings + `finish`), "Waterproof Safety Boots" (Footwear + `features`), "18V Cordless Range" (Power Tools + `voltage` and `power-source`). They cut across the real categories rather than splitting them.

### `excludeAsProductHome`
A product's breadcrumb leads to its **home category**: the deepest visible category that matches it. A cross-cutting page can be as deep as the real category (a stainless wood screw matches both `/c/fixings/screws/wood-screws` and `/c/fixings/stainless-steel`), so it would take the breadcrumb. Mark every cross-cutting page `excludeAsProductHome` (`manageMerchandisingTrees` setExcludeAsProductHome, or `excluded: true` when you create it). Splitting subcategories stay as homes. A product that still lands in the wrong place gets a main category (`manageProducts` update, mainCategory).

### Design tips
- **Decide the cross-cutting pages while designing attributes.** Each one needs a SELECT attribute with controlled values, filled on every product it should list.
- **Facets come from the same attributes.** A category's filters are taxonomy attribute ids (plus `price`, `inStock` and the like). An attribute worth a category page is usually worth a facet too.
- **Missing values hide products.** A product without the filtered attribute isn't on the page. Check completeness with `productTaxonomy` before relying on a filtered category.
