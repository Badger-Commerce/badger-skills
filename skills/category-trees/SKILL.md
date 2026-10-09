---
name: category-trees
description: Build and run Badger Commerce category trees (merchandising trees, the /c/ pages) through MCP — reading the outline, adding categories with each kind of backing, cross-cutting pages, product breadcrumbs and main categories, collections replaced by categories, and moving a collection-based shop onto a tree
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Category Trees

A site's categories come from one of two places, set by `category-navigation-source`
(`siteConfig` get):

| Value | Categories are | At |
|-------|----------------|----|
| COLLECTIONS (default) | collections and their parent collections | /collection/... |
| MERCH_TREE | the primary merchandising tree | /c/... |

A tree category is backed by a rule (a taxonomy level and attribute values, a query, a
collection, a list of products) rather than by products added to it one by one, and its pages
have filters. On a tree site the tree owns the category links, the menus' category items and the
product breadcrumb. Collections stay, but as product sets (grids, featured blocks, the home page)
and landing pages, not as categories. Building a tree doesn't change the shop's links, menus or
breadcrumbs until the setting says MERCH_TREE, though the primary tree's /c/ pages open at their
address. On a site already on MERCH_TREE, the primary tree is live: build a replacement as a
second tree, and make it primary (`updateTree` primary: true) when it's ready.

Rankings and category landing pages stay with `productRanking` / `manageProductRanking` (see
Order a category, below).
Designing the taxonomy a tree is built from: the `taxonomy` skill.

## Read before you write
1. `merchandisingTrees` trees: every tree, its slug and whether it's primary (the one the
   storefront uses).
2. `merchandisingTrees` outline (tree: id or slug; the primary one if you leave it out): every
   category, depth first, with path, name, backing (`collection:seoName`,
   `taxonomy:levelId +N filters`, `query`, `list:N`, `group`) and the `hidden`,
   `notProductHome` and `replaces` flags. Stops at 300 categories (`truncated`).
3. `merchandisingTrees` node (category: id, /c/ path or name): the full backing, children and
   `url`. A backing you write takes the same JSON.
4. `productRanking` preview (category) shows the products a category lists, as shoppers see them.

A category is named by id, path or name, looked up in the primary tree. To work on another tree
(a draft you're building, say), pass `tree` (id or slug) on every call, or use ids from its
outline.

## Add a category
`manageMerchandisingTrees` createNode: name, backing, parent (a path; leave it out or pass `/c`
for the top level), and optionally seoName (made from the name), position (1 = first),
description, metaTitle, metaDescription, hidden, excluded, collections (seoNames it replaces)
and facets (filter ids in order, e.g. taxonomy attribute ids, `price`, `inStock`, `onSale`,
`featured`, `manufacturer`, `rating`). facets only orders the filters: how one looks is
`manageProductTaxonomy` setFilterDisplay (`taxonomy` skill). The backing's
`type` decides what it lists:
- `{"type": "taxonomyProjection", "taxonomyLevelId": "fixings-screws", "attributeFilters":
  [{"attributeId": "application", "values": ["Drywall"]}]}`: products assigned to that taxonomy
  level or below, optionally narrowed to attribute values (exact allowed values from
  `taxonomies` get with the taxonomyId). The usual choice on a site with a taxonomy (`taxonomy` skill).
- `{"type": "manualCollection", "collectionSeoName": "gifts"}`: a collection's products. The
  category replaces that collection.
- `{"type": "manualList", "productSkuIds": ["SKU-1", "SKU-2"]}`: a fixed list in that order.
  SKUs, URL names or exact product names are accepted and stored as SKUs.
- `{"type": "virtualQuery", "query": {"op": "AND", "clauses": [{"type": "priceRange",
  "maxCents": 2000}, {"type": "inStock"}]}}`: a rule. Clauses: attributeEquals (attributeId,
  value), attributeIn (attributeId, values), attributeRange (attributeId, min, max), priceRange
  (minCents, maxCents), inStock, inCollection (collectionSeoNames), inTaxonomyLevel
  (taxonomyLevelId, includeDescendants), textMatch (text), nested (query); op is AND or OR.
- `{"type": "group"}`: a heading with no products of its own, for its subcategories.

A URL name must be unique among its siblings and can't contain `/` or spaces. Then `updateNode`
(category; only the fields you pass change, `facets: []` goes back to the default), `moveNode`
(category, parent and/or position) and `deleteNode` (takes its subcategories with it). Trees:
`createTree` (name, slug, primary), `updateTree`, `deleteTree`. Listings update after a reindex
the change queues, so give a new category a moment.

## Order a category
Rankings apply only under the default "Featured" sort, when browsing with no search term. A
`manualList` category keeps its own order and ignores them.
1. `productRanking` category (category: id, path or name) shows its ranking; `preview` (category,
   `limit` default 12, max 48) shows the first products as shoppers see them, each marked
   pinned, boosted, buried or normal.
2. `manageProductRanking` pin (products, category, `position` 1-based, default after the existing
   pins), boost, bury or unrank change one product at a time. setCategory (`pinned`, `boosted`,
   `buried`, at most 100 each) replaces the whole ranking; clearCategory removes it.
3. Brand or attribute landing pages (`/c/womens/dresses/colour/red`): `productRanking` landing
   (category, or none for the tree default), then `manageProductRanking` setLanding
   (`landingRules`, `[]` switches them off) or inheritLanding.
4. What a category optimises for when ranked by what sells: `manageProductRanking` setGoal (goal
   units, conversion, revenue or margin, with category; a blank goal inherits the shop's).

## Breadcrumbs: home and main categories
On a tree site, a product's breadcrumb leads to its home category: the deepest visible category
whose backing matches the product (a tie goes to the one first in the tree). Hidden categories,
anything under them, and virtual-query and group categories are never a home. With no match
the breadcrumb is just Home and the product.
1. Cross-cutting pages ("Stainless Steel Fixings", "Waterproof Boots", "18V Range") match as
   deeply as the real category and would take the breadcrumb. Mark them with
   `manageMerchandisingTrees` setExcludeAsProductHome (category, excluded: true), or pass
   `excluded: true` on createNode. The outline shows them as `notProductHome`.
2. `products` get: `homeCategory` is where the breadcrumb leads, `mainCategory` the override
   (null when none).
3. To override: `manageProducts` update with skuId and mainCategory (id, path or name in the
   primary tree; `none` clears). It wins only while that category is visible.

## Replace collections with categories
A collection is replaced by the first visible category that lists it in its replaces
(`replacesCollections`), or failing that, by one backed by it. On a tree site its page
301s to the category, and menu items and links built by the platform lead there.
1. `manageMerchandisingTrees` setReplacesCollections (category, collections: seoNames; `[]`
   clears) for a category built some other way, such as from the taxonomy.
2. `manageMerchandisingTrees` matchCollections (tree) does the obvious ones: each category gets
   the collection with its own URL name, where nothing replaces it yet. It returns the
   collections it matched.
3. Collections nothing replaces follow `unmapped-collection-visibility`: HIDDEN (the default:
   the page is a 404 and drops out of menus and the sitemap), NOINDEX (the page stays, out of
   search engines and the sitemap) or VISIBLE. The landing-page collection (the one `/` shows)
   always keeps its page.
4. `collections` get: `pageStatus` is OWN_PAGE, MOVED (`url` is the category), NOINDEX or HIDDEN
   (`url` is null). Until the site is on MERCH_TREE every collection reads OWN_PAGE, so work out
   the coming status from the outline's `backing` and `replaces`.

Links written into content (hero buttons, jsonComponent `href`, markdown) aren't rewritten. A
link to a replaced collection still works through the redirect, but one to a hidden collection
is a 404. On a tree site, link to the category's /c/ `url`.

## Move a collection-based shop onto a tree
Build and check while the site is still on COLLECTIONS. Switch last. Switching back is one
setting and nothing is lost, but browsers and search engines remember the 301s they've seen.
1. `siteConfig` get category-navigation-source (COLLECTIONS), `merchandisingTrees` trees, and
   `collections` list (every page). Sort the collections into categories and the rest: product
   sets and campaign pages such as `featured`, `sale` or `gifts`.
2. Build: `manageMerchandisingTrees` buildFromCollections (name, slug; primary: true to replace an
   existing primary tree). Each enabled collection becomes a category backed by it, so it
   replaces it. The landing-page collection is left out and its children become top-level, and a
   collection with two parents goes under the first. The tree is primary when the site had none.
   On a site with a taxonomy, building the categories from taxonomy levels (createNode) and then
   running matchCollections gives better categories. Either way, the setting doesn't change.
3. Tidy, from the outline: deleteNode the categories made from product sets and campaign
   collections, mark the cross-cutting pages (setExcludeAsProductHome), add `group` headings,
   moveNode into order, and setReplacesCollections where a URL name doesn't match.
4. Check:
   - the outline: every collection you sorted as a category appears as a `collection:` backing
     or in a `replaces`, and every one that doesn't is one you're happy to see HIDDEN, NOINDEX or
     VISIBLE;
   - `productRanking` preview (category) on several categories: they list products;
   - `products` get on a sample, including products in several collections: `homeCategory` is
     right; set mainCategory where it isn't;
   - `menus` get (main menu): after the switch, COLLECTION items lead to their category, but one
     to a collection nothing replaces loses its link when HIDDEN. Plan MERCHANDISING_NODE items
     (target: the /c/ path) to set when you switch;
   - content that links to /collection/... (`extensionConfig` on the home page and the main
     pages): plan /c/ links for those too;
   - featured blocks: a `featuredProductExtension` with fixedCollectionSeoName still works,
     because collections stay product sets. To feed it from a category, set `fixedCategoryPath`
     to the /c/ path instead.
5. Agree the switch with the merchant: it changes the live shop. `manageSiteConfig` update
   unmapped-collection-visibility if HIDDEN isn't right (NOINDEX keeps campaign pages), then
   `manageSiteConfig` update category-navigation-source MERCH_TREE. Then make the menu and
   content changes from step 4.
6. Afterwards: `collections` get on a few (MOVED, with a /c/ `url`), `menus` get (no item without
   a url), and the breadcrumb on a product page. Category data can take a minute to reach every
   server.
