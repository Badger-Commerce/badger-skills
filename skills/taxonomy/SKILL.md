---
name: taxonomy
description: Design and construct product taxonomies for Badger Commerce — common patterns for apparel, food, retail, and charity shops with attribute best practices
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
- Set `displayOrder` to control the order attributes appear on product pages

### Design Principles
- **Start broad, refine later** — begin with 2-3 levels and expand as needed
- **Keep attributes at the right level** — put shared attributes on parent levels, specific ones on leaf levels
- **Don't over-attribute** — 5-8 attributes per level is usually sufficient
- **Use SELECT types over STRING where possible** — they enable filtering and ensure data consistency

## Workflow

1. `taxonomies` getActive — check if a taxonomy already exists
2. Design your hierarchy on paper/notes first — levels, attributes, and allowed values
3. Create the taxonomy and add levels with their attributes
4. `productTaxonomy` assign — assign products to their appropriate levels
5. `productTaxonomy` updateAttributes — populate attribute values for each product
6. `productTaxonomy` getAttributes — check completeness scores and find gaps
7. Iterate: add attributes as new product categories emerge

**Tip:** Collections can have a `defaultTaxonomyLevelId` — when products are added to that collection, they're automatically assigned to the matching taxonomy level.
