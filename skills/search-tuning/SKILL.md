---
name: search-tuning
description: Diagnose and fix Badger Commerce site search through its MCP tools — "why doesn't X show when I search for Y", searches that find nothing, adding synonyms, and reviewing AI-suggested synonyms
user-invocable: false
allowed-tools: Read, Grep, Glob
---

# Search Tuning

Search matches the words a shopper types against each product's name, description and search
terms, with typo tolerance, prefix matching and stemming ("boot" finds "boots"). It does not know
that two different words mean the same thing: "shoes" finds nothing in a shop whose products say
"footwear". This skill is how you find those gaps and close them.

## When to use it

- The merchant asks "why don't I see X when I search for Y?"
- Reviewing the shop: popular searches that find nothing or very little.
- Before a launch or a range change: words customers use that the catalogue doesn't.

## The tools

| Tool | Actions |
| ---- | ------- |
| `synonyms` | `list` (optional `status` LIVE, SUGGESTED or REJECTED), `explain` (`query`, optional `product` SKU or seoName), `gaps` (zero-result searches in the last 30 days, with any synonym already covering each) |
| `manageSynonyms` | `create`, `update`, `delete`, `accept`, `reject`, `suggest`, `resync` |
| `siteTraffic` | `topSearches`, `zeroResultSearches` for wider context |

## Workflow: "why doesn't X show when I search for Y?"

1. `synonyms` explain with the `query` and the `product`. Read what it reports, in order:
   - **Not in the index** (disabled, missing, waiting for a reindex): this is not a search
     problem. Fix the product (`manageProducts`), or wait for the reindex.
   - **Indexed, but missing the query's words**: the product's name, description and search terms
     don't contain them. Pick one fix below.
   - **A synonym already applies but the product still doesn't match**: check the synonym's terms
     against the words in the product's name.
2. Choose the fix.
   - **The word is wrong or missing on the product**: the product is called "Rigger boot" and
     nothing says it is a boot for groundwork. Improve the name or description with
     `manageProducts`. This helps that one product.
   - **Customers and the catalogue use different words for the same thing**: "shoes" vs
     "footwear", "hi vis" vs "high visibility". Add a synonym. This helps every product.
   - **The product is in the wrong place**: a taxonomy or category problem, not a search one (see
     the `taxonomy` skill).
3. Run `explain` again to confirm the query now finds it.

## Workflow: close the gaps

1. `synonyms` gaps: the searches finding nothing, most frequent first.
2. Skip the ones you can't fix with a synonym: products the shop doesn't sell (tell the merchant;
   it may be a range gap), misspellings Typesense already tolerates, and terms that look like
   order numbers or names.
3. For the rest, either create synonyms yourself or `manageSynonyms` suggest. Suggest asks the
   shop's AI to propose groups from the zero- and low-result searches and the catalogue's own words.
   They arrive as SUGGESTED and are **not live**.
4. Review each suggestion: `synonyms` list with `status` SUGGESTED, `explain` the motivating query
   if unsure, then `accept` or `reject`. Rejected ones are remembered and not proposed again.

## Multi-way or one-way

- **MULTI_WAY**: all the terms mean the same thing. `shoes, footwear` means each finds the other.
  Use it for true equivalents: spelling variants (`hi-vis, hi vis, high visibility`), trade and
  consumer names (`sds drill, hammer drill`).
- **ONE_WAY**: searching the `root` also finds the terms, never the reverse. Root `boots` with
  `wellies, riggers, dealers` means "boots" shows every kind, while "wellies" stays specific. Use
  it for a broad word that should include narrower ones.

## Worked example: a builders' merchant

- `shoes`, `footwear`, `safety shoes` → MULTI_WAY `shoes, footwear`. Searching "safety shoes" then
  matches "safety footwear" too.
- `wellies` → their wellingtons are named "Safety Wellington": MULTI_WAY `wellies, wellington`.
- `boots` should show riggers and dealers too → ONE_WAY root `boots`, terms `rigger, dealer,
  wellington`.
- `ppe` → nothing is named PPE: ONE_WAY root `ppe`, terms `gloves, goggles, ear defenders, hard
  hat, mask, hi-vis`.
- `4x40 screws` → the products say "4.0 x 40mm": better fixed in the product names than with a
  synonym per size.

## Pitfalls

- **Over-broad synonyms**: `tools, hammer` makes every "tools" search a hammer search and every
  "hammer" search a tools search. If one word is broader, use ONE_WAY.
- **Synonyms are not stemming**: plurals already match, so don't add `boot, boots`.
- **Don't mirror the taxonomy**: a synonym for every category name adds noise. Synonyms are for
  the words customers type that the catalogue doesn't use.
- **Accepting blindly**: AI suggestions come from real failed searches, but check that each one
  connects to products the shop actually sells.
- **Out of sync**: if a synonym shows a sync error, or search ignores live synonyms after an
  outage, run `manageSynonyms` resync.
