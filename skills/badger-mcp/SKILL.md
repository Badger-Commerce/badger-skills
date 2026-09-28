---
name: badger-mcp
description: Workflows for running a Badger Commerce shop through its MCP tools — setting up products, collections, pages and extensions, variants, taxonomy, and reviewing performance
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
5. `inventory` get / `manageInventory` set: stock is held per SKU, not on the product.
6. Content: `manageExtensionConfig` add, then update (see below).

## Add content to a page, product or collection
1. `pages` get (or `products` / `collections` get) to find the item.
2. `extensions` listPageLocations to see the slots (body, sidebar, bannerSection, ...).
3. `extensions` getPromptSection for guidance on the extension you're adding. Skip this for
   jsonComponent, hero and animationScene: use the `json-components`, `hero` and
   `animation-scene` skills instead.
4. `manageExtensionConfig` add (extensionName, pageLocation).
5. `manageExtensionConfig` update (extensionConfigId from step 4 or from `extensionConfig`).
6. Call `getDesignBrief` first if you're producing anything visual.

## Set up a collection page
1. `manageCollections` create: name, seoName. Nesting under a parent is admin-UI only for now.
2. `manageProducts` addToCollection for each product.
3. `manageExtensionConfig` add a hero in `bannerSection`, then body content.

## Structured product data
1. `taxonomies` getActive. If there is none, design one with the `taxonomy` skill and create it
   in the admin UI (MCP can't create taxonomies yet).
2. `manageProductTaxonomy` assign each product to a level, then updateAttributes.
3. `productTaxonomy` (skuId) to find missing mandatory attributes.

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
5. Return (DISPATCHED or DELIVERED): `orders get` for the line ids, then `manageReturns` create
   with `lines {lineId: quantity}` and a reason. When the parcel arrives: `manageReturns`
   receive (restocks unless `restock: false`). Then `manageReturns` refund with `dryRun: true`,
   confirm the amount with the user, and refund with an `idempotencyKey`. Or reject with a reason
   the customer will see.
