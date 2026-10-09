---
name: badger-mcp
description: Workflows for running a Badger Commerce shop through its MCP tools — setting up products, collections, category trees, pages and extensions, variants, taxonomy, and reviewing performance
user-invocable: true
allowed-tools: Read, Grep, Glob
---

# Badger Commerce MCP Workflows

The Badger MCP server sends its own instructions when you connect: the domain model, how tools are
named, money units, identifiers, paging and error handling. This skill doesn't repeat them. It
adds step-by-step workflows for common jobs.

Every read tool has a `manage*` twin for writes (`products` / `manageProducts`,
`collections` / `manageCollections`, ...). A read-only connection can use only the read tools.

## Set up a new product
1. `manageProducts` create: name, seoName, description, price (minor units). Products are
   created disabled; pass `enabled: true` to publish immediately.
2. `collections` list: find the target collection.
3. `manageProducts` addToCollection: skuId plus the collection's seoName or ID.
4. If the site has a taxonomy (`taxonomies` getActive): `manageProductTaxonomy` assign, then
   `manageProductTaxonomy` updateAttributes, then check the result with `productTaxonomy`.
   On a category-tree site this is usually what puts the product in its categories (see
   Category trees below); `products` get shows its `homeCategory`.
5. `inventory` get / `manageInventory` set: stock is held per SKU, not on the product.
6. Content: `manageExtensionConfig` add, then update (see below).

## Add content to a page, product or collection
1. `pages` get (or `products` / `collections` get) to find the item.
2. `extensions` listPageLocations with itemType and itemId: the slots that item really renders.
   A block in any other slot is refused, because it would never show.
3. `extensions` getPromptSection for guidance on the extension you're adding. Skip this for
   jsonComponent, hero and animationScene: use the `json-components`, `hero` and
   `animation-scene` skills instead.
4. `manageExtensionConfig` add (extensionName, pageLocation, properties). Send jsonComponent,
   heroConfig and sceneConfig as JSON; jsonComponent is validated and the error names the fault.
5. `manageExtensionConfig` update (extensionConfigId from step 4 or from `extensionConfig`):
   properties, pageLocation to move it, enabled false to hide it. Order within a slot is
   `properties.displayPriority`, lowest first (default 1000).
6. Call `getDesignBrief` first if you're producing anything visual.

## Make the shop ready to trade
A new shop starts with the platform's name, colours and a £1.00 placeholder delivery option.
1. Identity: `manageBranding` setIdentity with the shop's real name (it shows in the header and
   page titles). The domain and going live stay with the merchant in the admin UI.
2. Brand: if `getDesignBrief` is still the defaults (indigo #4F46E5, empty voice), propose a
   palette and voice that fit the shop and save them with `manageBranding` setDesignBrief
   (only the sections you pass change; colours as hex). The storefront restyles to match, so do
   this before building content.
3. Delivery: `deliveryOptions` list, then `manageDeliveryOptions` update the placeholder and
   create the rest, with real names, prices (minor units) and descriptions.
4. Policies: a delivery and returns page (`managePages` create) stating the terms the merchant
   gave you, linked from the navigation.
5. Legal lines and wording: `manageSiteText` set overrides the platform's text for a key, e.g.
   `footer.disclaimer` (charity and company numbers, registered office) and `footer.footerText`
   (which otherwise credits the platform). `siteText` get shows a key's default first.
6. Say what you invented. If the merchant didn't give you a founder's name, a start date or a
   returns window, don't present a guess as fact: flag it in your summary.

## Make the shop ready for visitors
A shop isn't launched until a shopper landing on the home page can get everywhere and has a
reason to stay. After creating the catalogue and pages:
1. Navigation: `menus` list finds the main menu (main=true), `menus` get shows what shoppers
   see in `rendered`. `manageMenus` setItems replaces it: link every collection and page that
   matters (type COLLECTION or PAGE, target = seoName; on a category-tree site, categories are
   type MERCHANDISING_NODE with the /c/ path as target). Keep it to about six top-level items and
   group the rest as children. Don't add a Home item: the logo already links home. Check
   `rendered` afterwards: an item with no url is dead.
2. Home page: what `/` shows. On new shops it's the page `home` (`pages get home` finds it);
   older shops use the collection `home`.
   `extensionConfig` on it shows its blocks and the ones it inherits. Replace the placeholder
   welcome text, then build it top to bottom in the slots it renders:
   - `bannerSection`: a hero (`hero` skill) with the shop's promise and a button to the main
     collection;
   - `topBanner` / `mainSection`: a short intro and the brand story, as markdownFragment or a
     jsonComponent layout (`json-components` skill), with scroll reveal;
   - `bottomBanner`: reassurance (delivery, returns, how it's made) and an emailCaptureExtension.
   Aim for something worth reading, not a line of filler: several hundred words across blocks.
3. Featured products: on a home page, a jsonComponent productGrid or productCarousel bound to a
   collection (`collection-products` data source) shows them; the new-shop home page already has
   one bound to `featured`. Add your best sellers to that collection (`manageProducts`
   addToCollection) and take the placeholder `sample-product` out of it, or disable it. On an
   older shop's home collection, the built-in grid shows the collection's own products.
4. Images: use only the shop's own photos. `media list` shows the library; put them on products
   with `manageProducts` setImages (first is the main image) and use their `url` in hero and
   jsonComponent blocks. Never use stock, placeholder or made-up image URLs. If the library is
   empty or has nothing suitable, leave images out and tell the merchant which photos are needed.
5. Hand over real links: every result carries `url`, the shopper-facing address. Quote those
   (collections are /collection/..., tree categories /c/..., pages /p/..., products
   /product/...); never guess. On a tree site a collection's `url` may be its category, or null
   when it has no page.

## Set up a collection page
1. `manageCollections` create: name, seoName. Nesting under a parent is admin-UI only for now.
2. `manageProducts` addToCollection for each product.
3. `manageExtensionConfig` add a hero in `bannerSection`, then content in `topBanner` (above
   the product grid) or `bottomBanner` (below it).
4. Add it to the main navigation (`manageMenus` setItems, see above).

On a category-tree site, a collection is a product set or a landing page, not a category: add a
category to the tree instead (below), or the new collection's page may be hidden.

## Category trees (/c/ pages)
A site's categories come from one of two places, set by `category-navigation-source`
(`siteConfig` get): COLLECTIONS (the default; categories are collections and their parents,
at /collection/...) or MERCH_TREE (the primary merchandising tree, at /c/...). On a tree site,
collections still feed product grids, featured blocks and the home page, but the tree owns the
category links, the menus' category items and the product breadcrumb.
1. `merchandisingTrees` outline: every category of the primary tree, with its path and backing.
2. `products` get: `homeCategory` is where the product's breadcrumb leads. `manageProducts`
   update with mainCategory (a category's id, path or name; `none` clears) overrides it.

Adding and arranging categories, cross-cutting pages, replacing collections, and moving a
collection-based shop onto a tree: use the `category-trees` skill (`merchandisingTrees`,
`manageMerchandisingTrees`).

## Structured product data
1. `taxonomies` getActive (or list: `active` marks the one in use). If there is none, design one
   with the `taxonomy` skill: levels, attributes, types and allowed values.
2. Build it: `manageTaxonomies` create (name, description) gives the taxonomyId. Then addLevel
   for each level (parentLevelId for a sub-level, left out for a top-level one) and addAttribute
   for each attribute (levelId, `attribute` {name, type, mandatory, allowedValues, ...}). Put
   shared attributes on the parent level. Results carry the new levelId and attributeId;
   `taxonomies` get shows the whole tree.
3. `manageTaxonomies` setActive. The first activation changes no product. Switching from another
   active taxonomy clears every product's level and attributes: `dryRun: true` first, tell the
   user how many products it clears, and only then repeat with `confirm: true` (needs mcp:admin).
4. `manageProductTaxonomy` assign each product to a level, then updateAttributes.
5. `productTaxonomy` (skuId) to find missing mandatory attributes.
6. Later edits (updateLevel, moveLevel, removeLevel, updateAttribute, removeAttribute): on the
   active taxonomy, run the edit with `dryRun: true` first. Its `impact` lists products that
   would lose a level, an attribute or a value (or hold a value of the old type). If it isn't
   empty, tell the user and repeat with `confirm: true` only once they agree. Adding values,
   levels and attributes is always safe.

## Search

Searches that find nothing, or "why doesn't X show when I search for Y": use the `search-tuning`
skill (`synonyms` explain and gaps, `manageSynonyms`).

## Variants
1. `variantGroups` list to see existing groups (e.g. colour, size).
2. `manageVariants` create: parentSkuId, name,
   variantAttributes `{"colour": "Blue", "size": "Large"}`. Missing groups are created for you.

## Review the business
1. `statistics` dashboard: this month against last month in one call.
2. `statistics` topProducts / salesSummary / revenueBreakdown to drill in.
3. `siteTraffic` timeSeries / topPages.
4. `orders` countNew / list (status SUBMITTED) for orders waiting to be processed.

## Fulfil, cancel and take back orders
1. `opsInbox` for what's waiting: orders to pack and dispatch, returns to receive and refund.
2. Pack: `manageFulfilment` pack (SUBMITTED orders).
3. Dispatch: `manageFulfilment` dispatch with `carrier`, `trackingNumber` and, if the carrier
   has one, a `trackingUrl`. The customer's dispatch email includes them. If the customer says
   the email never arrived, `manageOrderEmails` resendDispatch.
4. Cancel (SUBMITTED or PACKING only): `manageCancellations` with `dryRun: true` first, and tell
   the user what will be refunded or released and restocked. Then run it with a `reason` and a
   new `idempotencyKey`. Orders with subscriptions, gift cards or donations can't be cancelled.
5. Customer return requests: `returns list state PENDING_APPROVAL`, `returns get` to read the
   customer's reason and comments, then `manageReturns` approve (emails them the return
   instructions) or reject with a reason they'll see.
6. Return (DISPATCHED or DELIVERED): `orders get` for the line ids, then `manageReturns` create
   with `lines {lineId: quantity}` and a reason. When the parcel arrives: `manageReturns`
   receive (restocks unless `restock: false`). Then `manageReturns` refund with `dryRun: true`,
   confirm the amount with the user, and refund with an `idempotencyKey`. Or reject with a reason
   the customer will see.
