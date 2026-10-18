# Made-to-order wall art product template

A product template for ready-designed wall art that is produced after the
order is placed. No personalisation, no file upload, no proof step.

Built against this repo's theme: **Horizon 4.1.5** (not Dawn — Horizon uses
theme blocks in `blocks/` composed inside the `product-information` section,
so this is not a Dawn-style template).

## Files

| File | What it is |
| --- | --- |
| `templates/product.made-to-order.json` | The template. Assign products to it as `made-to-order`. |
| `blocks/mto-badge.liquid` | "Made to order" badge + estimated delivery range. |
| `blocks/mto-steps.liquid` | Three-step "how made to order works" strip. |
| `blocks/mto-specs.liquid` | Specs table read from product metafields. |
| `blocks/mto-tag-notes.liquid` | Notes shown only for matching `wa-` tags. |
| `blocks/mto-returns.liquid` | Returns / cancellation notice + policy link. |
| `blocks/mto-scale.liquid` | Per-product scale / in-room image from a metafield. |
| `blocks/mto-product-schema.liquid` | Product JSON-LD tuned for made-to-order. |
| `sections/product-mto-related.liquid` | Related products from the product's own `wa-` collection. |
| `snippets/mto-delivery-range.liquid` | Business-day date maths shared by the badge. |
| `sections/product-information.liquid` | **Changed**: skips its own `structured_data` script when the template suffix is `made-to-order`, so that page has exactly one Product graph. |

Everything customer-facing is a theme-editor setting. Nothing is hardcoded in
a way that needs a developer to reword it.

## Install

This repo is connected to Shopify through the GitHub integration, so
**merging to `main` deploys**. There is no `shopify theme push` step.

1. Merge to `main`. Shopify pulls the branch into the connected theme on its
   own. All ten new files sit in directories Shopify already serves, so no
   config changes are needed. (`MADE-TO-ORDER-TEMPLATE.md` is at the repo
   root, which is not a theme directory, so Shopify ignores it.)
2. Create the product metafield definitions below, if they do not exist yet.
   Without them the specs and scale blocks simply hide themselves, so the
   template is safe to deploy first and fill in after.
3. **Check the wording before assigning any product.** Open a made-to-order
   product in the theme editor, in a preview rather than the live theme, and
   read the returns notice and the day figures. Production and shipping days
   appear in **two** places that must agree — the "Made to order badge" block
   and the "Made-to-order schema" block.
4. Assign products to the template (see "Bulk assignment").

Steps 1 and 2 are invisible to customers: the template renders for nobody
until a product is assigned to it in step 4, and the one change to existing
theme code is guarded by the `made-to-order` template suffix, so current
product pages behave exactly as before. **Step 4 is the step that goes
live** — the moment a product is assigned, whatever copy is in the blocks is
what customers read.

### Metafield definitions

All owner type **Product**, and all must be visible to the storefront.

| Key | Type | Used for |
| --- | --- | --- |
| `custom.art_size` | Single line text | Specs row "Size" |
| `custom.art_material` | Single line text | Specs row "Material" |
| `custom.art_frame` | Single line text | Specs row "Frame" |
| `custom.ready_to_hang` | True or false | Specs row "Hanging hardware" — shows "Included" when true, hides the row when false |
| `custom.scale_image` | File (image) or Image | The scale / in-room visual |

The keys are settings, not hardcoded. If your definitions are already named
something else, retype the key in the block rather than renaming metafields.
A row also falls back to a variant option: row 1 falls back to an option
named "Size" when `custom.art_size` is empty.

A `list.single_line_text` metafield is joined with commas. Rich text keeps its
markup. Any row with an empty metafield is dropped, and the whole specs block
hides when every row is empty.

### Bulk assignment

This is the step customers see, so do it after the copy has been checked.
It needs Admin access, so it runs through Codex — see the handoff block in
the chat thread, or do it by hand:

- **Admin UI, small batches:** Products → filter by product type "Wall Art"
  (or a `wa-` tag) → select → Edit products → add the Theme template column →
  set `made-to-order`.
- **Programmatically:** `productUpdate` with
  `templateSuffix: "made-to-order"`, batched over the matching product IDs.
  Fetch current state first and log every changed product.

## Assumptions made

- **Theme.** Horizon 4.1.5, read from `config/settings_schema.json`. The
  template reuses this repo's existing section and block types:
  `product-information` with the static `_product-media-gallery` and
  `_product-details` blocks, and the `group`, `product-title`, `price`,
  `_divider`, `variant-picker`, `buy-buttons`, `text`, `accordion` and
  `_accordion-row` blocks — the same set `templates/product.json` already
  uses, with the same setting values, so it renders consistently with the
  current product page.
- **Production and shipping times.** Your brief left these blank. The
  template ships with 3-7 business days production and 5-10 business days
  shipping, as editor settings. Change them in the badge block *and* the
  schema block.
- **Returns policy.** Also blank in the brief, so the notice ships with
  conservative made-to-order wording: no returns or exchanges once production
  starts, free cancellation within 24 hours, damage reported within 48 hours
  with photos. **Read this before going live** and match it to your published
  policy. The link falls back to the shop's refund policy page.
- **Ships from.** Uses the existing `[[origin]]` market token rather than a
  fixed country, so US and Canadian markets read correctly — consistent with
  how the rest of this theme now handles country-specific copy.
- **Heading levels.** Horizon renders the product title as
  `<p role="heading" aria-level="3">`, so in-column block headings are set to
  `h4` and the related-products heading to `h3`. No level is skipped.
- **Tags.** `wa-hand-painted`, `wa-framed`, `wa-sets`, `wa-large`, matched
  exactly — `wa-large` will not match `wa-large-set`. Tag and copy are both
  editable, so a new tag needs no code.

## Things worth knowing

- **Date accuracy.** The delivery range is rendered in Liquid (so it works
  without JavaScript and crawlers see it) and then recomputed on load by a
  small inline script, because a cached page can otherwise serve yesterday's
  range. The script uses the visitor's timezone while Liquid uses the shop's,
  so the two can differ by a day near midnight; switch "Refresh the range in
  the browser" off to keep the shop-time value only. **Public holidays are
  not modelled** — only weekends. Pad the shipping days if that matters.
- **Schema availability.** Defaults to `PreOrder`, which is what Google
  documents for goods ordered now and made later. `MadeToOrder` is valid
  schema.org but is not on Google's supported list, so it may be ignored;
  it is offered in the dropdown anyway. A product Shopify reports as
  unavailable is marked `OutOfStock` regardless of the setting.
- **Merchant Center.** This markup is independent of your product feed. If
  you sync to Google Shopping, the feed's availability comes from Shopify
  inventory, so a feed saying "in stock" against a page saying "preorder"
  can raise a mismatch warning. Worth checking if you run Shopping ads.
- **Do not add this template's schema block to other templates.** The
  product section only steps aside for the `made-to-order` suffix, so the
  block elsewhere would produce two Product graphs on one page.
- No new external libraries. Images are lazy-loaded and responsive. The only
  JavaScript is the ~30-line inline date refresh.

## Verified / not verified

- Verified locally: `shopify theme check` reports **0 offences** across all
  ten files, every `{% schema %}` and the JSON template parse, and Liquid
  control tags balance.
- Not verified: nothing has been rendered against live product data. The
  specs, scale and related-products blocks depend on metafields, tags and
  `wa-` collection membership that only a real product exercises. Preview one
  product of each kind (hand-painted, framed, set, oversized) before assigning
  the template in bulk.
