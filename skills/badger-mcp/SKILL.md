---
name: badger-mcp
description: Workflows for running a Badger Commerce shop through its MCP tools — setting up products, collections, category trees, pages and extensions, variants, taxonomy, email campaigns and customer messaging, and reviewing performance
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
1. `manageProducts` create: name, seoName, description, price (minor units), stereotypeId
   `defaultProduct`. Without a stereotype the product page gets none of the stereotype's blocks
   (nav, footer); a product that gains variants moves to `variantProduct` by itself. Products are
   created disabled; pass `enabled: true` to publish immediately.
2. `collections` list: find the target collection.
3. `manageProducts` addToCollection: skuId plus the collection's seoName or ID.
4. If the site has a taxonomy (`taxonomies` getActive): `manageProductTaxonomy` assign, then
   `manageProductTaxonomy` updateAttributes, then check the result with `productTaxonomy`.
   On a category-tree site this is usually what puts the product in its categories (see
   Category trees below); `products` get shows its `homeCategory`.
5. `inventory` get / `manageInventory` set: stock is held per SKU, not on the product.
6. Content: `manageExtensionConfig` add, then update (see below).

Donations: `productTypeDecorators` `["donationProduct"]`. It defaults to no shipping and
quantityType PRICE_AS_QUANTITY (the shopper enters the amount). For a fixed-price gift (e.g. £10)
set quantityType `FIXED`, or the storefront shows "Give any amount". A monthly gift needs both
`subscriptionProduct` and `donationProduct`, quantityType `FIXED`, and a
`donationFormExtension` on it for the Gift Aid tick; with `subscriptionProduct` alone it recurs
but records no donations. Gift Aid on recurring gifts needs the Charity plan or Enterprise.
Each tier then needs a Stripe price: see Subscription prices below.

## Subscription prices (monthly giving, memberships)
A subscription SKU (a variant, or a product without variants, with type `subscriptionProduct`)
renews at a Stripe recurring price. Never ask the owner for a Stripe price ID; these tools make
and change prices in Stripe for them. Stripe must be connected (test mode is fine to set up).
1. `subscriptionPrices` list: existing prices, and `unpricedSkus` that still need one.
2. `manageSubscriptionPrices` create per SKU: `sku` (the variant SKU), `amount` (minor units),
   `interval` (default month). It also sets the SKU's own price to match, so the first payment
   (taken at checkout) and the renewals agree. Pass an `idempotencyKey` so a retry makes no
   second price.
3. To change an amount: `manageSubscriptionPrices` change with `priceId` and the new `amount`,
   `dryRun: true` first. Existing subscribers stay on what they pay unless `subscribers: "move"`
   (new amount from their next renewal; ask the owner first, and tell them to warn their
   supporters, since nobody is emailed).
4. `subscriptionPrices` check after going live, or when something looks wrong; then
   `manageSubscriptionPrices` sync recreates prices missing from live mode or archived.
A price made in the Stripe dashboard is adopted with `manageSubscriptionPrices` link.

## Events and tickets
An event is one product with type `eventTicket`, one product per date. Its variants are the ticket
types (Adult, Child, Family), and each variant's stock is how many of that type can be sold. Every
unit sold is one ticket with its own QR code, which staff scan at the door with the Badger app.
1. `manageProducts` create with `productTypeDecorators` `["eventTicket"]` and an `event` object:
   `start` (required, ISO-8601 such as `2026-12-05T19:00`, in the shop's time zone unless it carries
   an offset), and optionally `end`, `venue`, `checkInOpensMinutesBefore` (default 120),
   `salesCloseMinutesBefore` (leave it out to sell until the event ends, so the door can sell) and
   `admitsPerTicket` (default 1). It defaults to no shipping. Change the details later with
   `manageProducts` update and only the `event` fields that change; `""` clears one.
2. Ticket types: `manageVariants` create per type (e.g. `variantAttributes {"ticket": "Adult"}`).
   A variant that admits several people (a family ticket) needs its own `admitsPerTicket`
   attribute. Without variants the event has a single ticket type.
3. Capacity: `manageInventory` set on each ticket type's SKU. Checkout refuses an order once a type
   is sold out, so the stock is the hard limit.
4. Attendance: `tickets` list for every event with tickets issued, checked in and void; `tickets`
   get with the event's `skuId` for the tally per type and a page of tickets (filter `status`
   ISSUED, CHECKED_IN or VOID).
Door staff need the POS Agent role and **Check in event tickets** under Settings, POS agents in the
admin; the MCP tools can't grant either. Manual check-in, undo and voiding are in the admin's
attendance page (Orders, Event tickets), not in MCP. Tickets can't be returned: refund the order,
which voids them.

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
   `properties.displayPriority`, lowest first (default 1000, but some blocks default lower, e.g.
   callToAction at 500, so set it whenever a slot holds more than one block).
6. Call `getDesignBrief` first if you're producing anything visual.

## Make the shop ready to trade
A new shop starts with the platform's name, colours and a £1.00 placeholder delivery option.
1. Identity: `manageBranding` setIdentity with the shop's real name (it shows in the header and
   page titles). The domain and going live stay with the merchant in the admin UI.
2. Brand: if `getDesignBrief` is still the defaults (indigo #4F46E5, empty voice), propose a
   palette and voice that fit the shop and save them with `manageBranding` setDesignBrief
   (only the sections you pass change; colours as hex). The storefront restyles to match, so do
   this before building content. `manageBranding` listThemes / setTheme switches the storefront
   theme, and setCustomCss replaces the hand-written stylesheet (use `var(--color-*)` tokens);
   `getDesignBrief` includeCss shows the current one.
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
   Photos the merchant hands you (a public https URL, or a file as base64) go into the library
   with `manageMedia` import (title, altText); use the returned id and url.
5. Hand over real links: every result carries `url`, the shopper-facing address. Quote those
   (collections are /collection/..., tree categories /c/..., pages /p/..., products
   /product/...); never guess. On a tree site a collection's `url` may be its category, or null
   when it has no page.

## Blogs and blog posts
A blog is two kinds of page, and both must be shaped the way the admin's Blog screen makes them.
A blog page that's just a page with the right template still renders, but the admin's blog
list and the RSS feed (`/rss/<blog seoName>`) can't find it.
- **The blog** (the list page): stereotypeId `blogPage`, parentPageIds `["blogs"]` (the fixed
  parent every blog sits under, not a real page), template `blog/blogList`. `managePages` create
  with stereotypeId `blogPage` sets the parent and template for you.
- **A post**: stereotypeId `blogPost`, parentPageIds `[<blog seoName>]`. The template
  (`blog/blogPost`) is picked for you, and the seoName is prefixed with the blog's
  (`blog/my-post`).

1. Find the blog: `pages` get with its seoName (new shops come with `blog`). Check it has
   stereotypeId `blogPage` and parentPageIds `["blogs"]`. If either is missing, fix it with
   `managePages` update (pageId, stereotypeId `blogPage`, parentPageIds `["blogs"]`) before
   adding posts: update doesn't fill them in. No blog yet: `managePages` create (title,
   seoName, stereotypeId `blogPage`).
2. `pages` listChildren parentSeoName <blog seoName> lists the posts already there.
3. Write a post: `managePages` create (title, seoName, description, stereotypeId `blogPost`,
   parentPageIds [<blog seoName>], publishedDate). The description is the standfirst under the
   masthead title and on cards. publishedDate is the date the blog shows and sorts by; without
   it the post is dated today.
4. Image: `managePages` setImages with one `media` id. It fills the inherited
   `blogMastheadExtension` in `bannerSection`, which is the post's banner (so no hero), and is
   also the blog card and the link preview. The masthead's byline is the page's `author`
   attribute, which managePages can't set (admin post editor).
5. Body, in `mainSection` (itemId is the post's full seoName, `blog/my-post`): prose (see the
   shared blocks below), with optional quote and images jsonComponents between the prose, then
   `blogRelatedArticlesExtension` (`excludeCurrentPost` defaults to true) last.
6. Optional: a cta variant `panel` in `rightSection`. Anything there puts the post beside a
   sidebar; without it the post is one centred column.
7. Link the blog (not each post) from the navigation, and check the post's `url`.

## Content page recipes
Standard page shapes. First: `getDesignBrief` for the voice, `media list` for real photos (never
made-up URLs), and `extensions listPageLocations` (itemType `page`, itemId the page's full
seoName) for its slots. Each block is a `manageExtensionConfig` add on itemType `page`; give
every block in a slot a `displayPriority` (100, 200, ...) so they keep your order. Several
hundred words in all.

Blocks the recipes share:
- **prose**: `markdownFragment` with `html` (the markdown), `cssStyle: "blog"`, `staticMode: true`.
  Long-form text only, one block for a run of prose, no raw HTML.
- **quote / images / facts**: `jsonComponent` with `"layout": "contained"` (`json-components`
  skill). Quote: a `blockquote` (quote, author, role). Images: one `image`; 2-3 as a `row` of
  `column`s with an `image` each; 4+ as a `carousel`; alt text from the media. Facts: a `row` of
  `column`s, each an `icon` (Lucide name, e.g. `calendar`, `clock`, `map-pin`, `mail`, `user`,
  `award`) + `heading` + `text`.
- **cta**: `callToAction` with `variant` (`band`, `strip`, `panel`, `stat`; default `band`),
  `eyebrow`, `heading`, `text`, `value` (the figure, for `stat`), `primaryLabel` + `primaryUrl`,
  optional `secondaryLabel` + `secondaryUrl`. Links are site paths (`/p/donate`), anchors, https,
  `mailto:` or `tel:`; a button missing its label or link is dropped.
- **hero**: `hero` with a `heroConfig` JSON string (`hero` skill).

**News or blog post**: see Blogs and blog posts above.

**Profile page** (staff, trustee or volunteer bio)
1. `managePages` create: title = the person's name, seoName, description (stereotype defaults to
   `defaultPage`).
2. `bannerSection`: hero preset `split-image`: `heroImage` = their portrait (library url, alt = the
   name), `title` = name, `subtitle` = role. No portrait in the library: ask for one rather than
   leave the placeholder.
3. `mainSection`: prose biography, a facts row (expertise, qualifications, contact), a quote.
4. cta variant `strip` after them.

**Event page**
1. `managePages` create (stereotype `defaultPage`).
2. `bannerSection`: hero preset `centered`, `eyebrow` = the date, `title`, `subtitle`, `cta` = the
   book or register button.
3. `mainSection`: facts row (`calendar` date, `clock` time, `map-pin` place), prose description
   and agenda, then `faqExtension`: `title`, and `questions` and `answers` as two lists of equal
   length matched by index (answers may hold HTML); optional `style` `default`, `compact` or
   `bordered`.
4. Optional `openStreetMapExtension` (`latitude`, `longitude`, `markerTitle`, `zoomLevel`) only
   with the venue's real coordinates: without them it centres on London.
5. cta variant `band` in `bottomBanner`.

## Set up a collection page
1. `manageCollections` create: name, seoName, stereotypeId `defaultCollection` (it supplies the
   nav menu and the product grid; without it the page has neither). Nesting under a parent is
   admin-UI only for now.
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

Searches that find nothing, "why doesn't X show when I search for Y", or putting products in
order for a search term: use the `search-tuning` skill (`synonyms` explain and gaps,
`manageSynonyms`, `productRanking` / `manageProductRanking`).

## Share a "buy now" or "donate" link
For a QR code, an email, a social post or a button on another website: a direct checkout link
puts one product in an order of its own and opens checkout. The shopper's basket isn't touched,
and on checkout they can still change the amount (donations) or quantity, and add Gift Aid.
1. `products` get: `buyUrl` is the bare link (`https://{shop}/buy/{seoName}`).
2. Add parameters as needed:
   - `amount=N` for an open-amount product (a donation or other PRICE_AS_QUANTITY product), in
     whole pounds (`amount=2` is £2); `qty=N` for anything else, including a fixed-price
     donation (quantityType FIXED or SELECTABLE). Values outside the product's min/max are
     clamped.
   - `v=variantSku` for a product with variants. Without one, the shopper lands on the product
     page to choose.
   - `src=label` (e.g. `src=summer-fair-2026`) to see which link raised what: it's saved on every
     order made through it.
3. The product must be enabled. Don't open the link to test it: every visit creates an order.

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

## Customer messaging and email campaigns
> Added with badger-commerce MR !550. Until that is released, a shop's MCP server won't have the
> `campaigns`, `manageCampaigns`, `contacts`, `contactRequests` or `legalDocuments` tools.

Shoppers' consent is recorded per contact and per mailing list; campaigns go only to the people who
agreed. The consent rules are the platform's, not yours to bend, so these workflows are about
using them well.

### Send an email campaign
A campaign is written once, in the shop's default language, and goes to one mailing list. Work
through it in order and stop whenever a step fails.
1. `campaigns topics`: the shop's mailing lists with key, name and confirmed subscribers. Pick the
   list with the owner (`general` is the built-in "News and offers"). A list with no confirmed
   subscribers can't be sent to. Archived lists are flagged.
2. Sending domain: `campaigns sendingDomain`. Campaigns go out from the shop's own domain, never a
   shared Badger one, and can't be scheduled until it is verified. `{configured: false}` means none
   yet. If it isn't verified, say so and don't carry on to scheduling:
   - none, or DNS records failing: the shop owner must publish the DNS records it lists at their DNS
     provider (where they bought the domain). Suggest a subdomain such as `news.theirshop.co.uk`,
     so marketing mail builds its own reputation and can't affect order emails. `manageCampaigns`
     `setSendingDomain` (`domainName`) adds it and returns the records; `checkSendingDomain` asks
     again once they're published. Both, and `removeSendingDomain`, need mcp:admin. DNS can take a
     few minutes to a day, so hand the records over and come back later.
   - company details missing: the owner fills in **Legal entity name** and **Registered address**
     in Settings; the email provider won't approve a sender without them, and the footer needs the
     address.
   - under review, not approved, suspended: wait, or tell the owner to contact support. A suspended
     domain pauses any campaign that was sending.
   Test sends work before the domain is verified, so you can build and test in the meantime.
3. `getDesignBrief` for the voice, `media list` for real photos (never made-up URLs).
   `manageCampaigns` create (name, subject, `topic` = the list key, `preheader`, `blocks`). `blocks`
   is the whole body (up to 40), each `{type, params}`:
   - `HEADING`: `text` (up to 200), `level` 1 or 2;
   - `TEXT`: `markdown` (up to 10,000; links `https://`, `http://` or `mailto:`; raw HTML is shown as
     text);
   - `IMAGE`: `mediaId` from the media library, `alt`, optional `linkUrl`;
   - `BUTTON`: `label` (up to 60) and `url`;
   - `PRODUCT_GRID`: `collectionSeoName`, `limit` 2 to 12 (default 6), `columns` 2 or 3 (default 3),
     `showPrices` (default true). The products are chosen when sending starts, in the collection's
     order, and anything disabled or unbuyable is left out;
   - `DIVIDER`: no params.
   Links are `https://` addresses or paths on the shop (`/sale`). `{{first_name}}` or
   `{{first_name|friend}}` greets by name in the subject, a heading or text; nothing else merges.
   The footer (unsubscribe, list name, business address) is added by the platform and can't be
   edited. Write one campaign per language, each to its own list.
4. `manageCampaigns` update to change it: pass the `version` you last read (`campaigns get`), and
   only the fields you give change (`blocks` replaces all of them). A stale version is refused:
   read again and retry. Only a draft can be edited; `duplicate` makes a draft copy of anything else.
5. `campaigns preview`: the text version for a sample recipient, plus `problems`, which lists
   everything stopping it being scheduled (missing subject or content, an image or collection that
   no longer exists, an empty product grid, no verified domain, no business address, nobody to send
   to, or an email over 100 KB). Fix these before going on.
6. `manageCampaigns` testSend with `addresses`: the owner's own addresses, at most 5 (20 test sends
   an hour for the shop). It goes through the normal order-email sender, marked "[Test]" with
   today's products. Never put a customer's address here. It returns `sent` and `refused` counts;
   an address on the do-not-email list is refused. Ask the owner to check the inbox and approve.
7. `campaigns audience` (`topic`): how many people the list reaches now. Tell the owner that
   number in plain words ("this will go to about 1,240 people on News and offers") and get their
   explicit yes to it and to the send time.
8. `manageCampaigns` schedule with `confirmAudience` equal to the number from step 7. It is refused
   if it is missing or the list has changed since: the refusal gives the new number and schedules
   nothing, so tell the owner the new figure and ask again. `sendAt` is the shop's local time
   (`2026-11-03T09:00`, in the shop's time zone) or an ISO time with an offset, up to 90 days ahead.
   **Leaving `sendAt` out sends now**, so only do that when the owner asked for it.
   Until it starts sending, `unschedule` returns it to a draft and `cancel` stops it.
9. Watch it: `campaigns get` for status, `campaigns stats` for the counts and rates. Delivery,
   bounce and complaint numbers keep arriving for hours after the status says SENT. A shop sends at
   most 5,000 emails an hour, so a big list takes several hours. `pause`, `resume` and `cancel`
   work while it sends; `cancel` is final and emails already sent can't be recalled. Read the
   numbers like this: skipped is people who no longer qualified (unsubscribed, unconfirmed,
   erased); suppressed is addresses on the do-not-email list; bounced and complained addresses come
   off future campaigns by themselves. A complaint rate that is more than a trickle, or many
   bounces, means the list needs attention: say so rather than reassure.

### When a campaign has paused
`opsInbox` has a `campaigns` line counting campaigns the system paused and a sending domain that
needs action; its `next` is `campaigns list status PAUSED`. A campaign the system paused has a
`statusReason` (a person pausing one leaves none), and it is usually one of these problems:
- the sending domain stopped being verified (DNS records changed or removed): `campaigns
  sendingDomain` shows which record fails; the owner fixes it at their DNS provider, then
  `checkSendingDomain` (mcp:admin), then `resume`;
- the email provider refused the account's details three times in a row, or kept failing for an
  hour, or a batch stopped moving: tell the owner to check the shop's Mailtrap settings (Settings,
  Email campaigns) or contact support. Nothing is lost, and `resume` sends what hadn't gone;
- the domain is suspended or not approved: contact support.
Don't `resume` repeatedly without the cause changing, and don't `cancel` unless the owner wants it
stopped for good. The error text is in `statusReason`: quote it to the owner.

### Customer questions: is this person subscribed, did they get it
`contacts` and `contactRequests` hold names, email addresses, consent history and message text, so
they need mcp:write even though they only read. Don't paste those details anywhere they aren't
needed, and don't repeat addresses back more than the question requires.
1. `contacts search`: `query` is the start of an email address or part of a name (an address
   searches from its start). Narrow with `status` (subscribed, pending, unsubscribed) on a list
   (`topic`, default `general`). `contacts summary` counts subscribers, pending and unsubscribed
   per list.
2. `contacts get` (`contactId`): the addresses and whether each works, where they stand on each
   list now (`consent`), and the last 50 decisions in `history`, with date, source and method
   (ticked box, confirmation link, one-click unsubscribe, staff). Use it to answer "why didn't they
   get the email":
   - pending: they signed up but haven't clicked the confirmation link, so campaigns skip them;
   - unsubscribed: they did it themselves (or staff did, the history says which);
   - the address shows a warning: it bounced or they reported spam, so it is on the do-not-email
     list, and nothing but their own confirmation click undoes that;
   - erased: the contact is gone;
   - not on that list at all, or subscribed to a different list;
   - subscribed but the campaign still skipped them: with "Confirm newsletter sign-ups by email"
     on, an address must also have been proven theirs, so check `verified` on the address.
   No match at all: they never gave an email address, or it was entered differently.
3. The contact-us inbox: `contactRequests list` (status NEW by default, or IN_PROGRESS, CLOSED,
   SPAM or `all`; `query` over name, email and subject), then `contactRequests get` for the whole
   message, its order and the replies sent so far. Check `notificationFailed` and any replies. `opsInbox`
   counts new requests and the age of the oldest.
4. **You can't act on any of it over MCP.** There is no way to subscribe or unsubscribe someone,
   erase a contact, reply to a message or change a request's status, and the do-not-email list isn't
   exposed. Say what to do instead: the owner (an admin) uses Customers, Contacts (unsubscribe, erase),
   Customers, Contact requests (reply, close), Customers, Do-not-email list, or an integration over the
   REST API. Staff can only ever unsubscribe people: someone is subscribed only when they agree
   themselves. If a customer asks you to sign them up, the answer is the sign-up form, the tick box at
   checkout or their account's Preferences tab.

### Check the terms and privacy notice
Terms of sale and the privacy notice are ordinary pages, and the platform stores each text a customer
was shown under a version label. If the page is edited without a new label, two different texts share
one name, which makes the record of what someone agreed to less useful.
1. `legalDocuments current` (optionally `kind` `TERMS_OF_SALE` or `PRIVACY_NOTICE`): the page, label
   and link in use, and `drift`: `NOT_CONFIGURED` (no page set, so nothing is shown or recorded),
   `PAGE_MISSING`, `UNREADABLE`, `EDITED_UNDER_SAME_LABEL` (the words changed, the label didn't) and
   `LABEL_SHARED` (several texts carry the label).
2. For `EDITED_UNDER_SAME_LABEL` or `LABEL_SHARED`, advise the owner to bump the version label
   (Settings, Store, Terms of sale and privacy notice) whenever they change the page's wording. You
   can't change the label or the page settings from MCP.
3. `legalDocuments snapshots` (`kind`) lists the stored texts newest first with when each was first
   shown, its hash and how many acceptances carry it; `legalDocuments get` (`snapshotId`) shows one
   text word for word if it is short. A text is stored the first time a customer is shown it, so a
   freshly edited page reads as not stored yet until someone sees it. Looking never creates a record.

### Guardrails for messaging
- **Never schedule without the owner's explicit go-ahead on the audience number** (and the send time).
  The owner saying "send the newsletter" is not that: read `campaigns audience`, tell them the
  figure, and wait for yes. `confirmAudience` is a check against a mistake, not a substitute for the
  owner's approval. Don't pass a number you were given earlier without re-reading it.
- Test sends go to the owner's own addresses only, never a customer's. A campaign can't be sent to
  an address you supply, and you shouldn't look for a way around that.
- **Consent rules hold.** Only confirmed subscribers of the campaign's list get it. Someone who
  unsubscribed, complained or bounced stays off until they confirm again from their own inbox: don't
  suggest re-adding them, importing them, or ticking a box on their behalf, none of which works.
  Don't promise a customer that a staff action will re-subscribe them.
- **No shared sending domain.** Campaigns always send from the shop's own verified domain. If there
  isn't one, the answer is DNS records for the owner to publish, not a workaround. Badger's own
  domains can't be used, and a domain belongs to one shop.
- Don't use campaigns for order or account email: those are `manageOrderEmails` and the transactional
  emails. A campaign is for news and offers the person asked for.
