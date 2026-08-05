---
name: Browse the Redefine Meat product catalog
description: >-
  Read Redefine Meat's live plant-based New-Meat product catalog anonymously through the
  WooCommerce Store API - list products, page through them, filter by the B2B/B2C category
  split, and fetch a single product by id or slug.
api: openapi/redefine-meat-store-openapi.json
base_url: https://www.redefinemeat.com/wp-json/wc/store/v1
auth: none
operations:
- getWcStoreV1Products
- getWcStoreV1ProductsById
- getWcStoreV1ProductsBySlug
- getWcStoreV1ProductsCategories
- getWcStoreV1ProductsCategoriesById
- getWcStoreV1ProductsCollectionData
generated: '2026-08-05'
method: generated
source: openapi/redefine-meat-store-openapi.json
---

# Browse the Redefine Meat product catalog

Redefine Meat runs no developer program, but its WooCommerce Store API is genuinely public.
No credential is required for any step below — this was verified against the live host on
2026-08-05.

## 1. Understand the shape of the catalog

Call `getWcStoreV1ProductsCategories` (`GET /wc/store/v1/products/categories`). The catalog is
split into two top-level categories: `b2b` (foodservice / PRO range) and `b2c` (consumer range).
Each category object carries a `count`, so you can size each range before fetching products.

For facet counts across the whole collection — price range, attribute counts, rating counts,
stock-status counts — call `getWcStoreV1ProductsCollectionData`.

## 2. List products

Call `getWcStoreV1Products` (`GET /wc/store/v1/products`).

- Paginate with `page` (default 1) and `per_page` (default 10, **maximum 100**). Exceeding the
  maximum returns HTTP 400 `rest_invalid_param` with a `data.details.per_page` explanation.
- Read `X-WP-Total` and `X-WP-TotalPages` from the response headers to know when to stop; the
  `Link` header carries `rel="next"`.
- Narrow with `search`, `slug`, `include`, `exclude`, `order`, `orderby`, `after`, `before`.
- Use `context=view` (default) unless you have credentials — `context=edit` requires auth.

## 3. Fetch one product

Either `getWcStoreV1ProductsById` (`GET /wc/store/v1/products/{id}`) or
`getWcStoreV1ProductsBySlug` (`GET /wc/store/v1/products/{slug}`). An unknown id returns
HTTP 404 `woocommerce_rest_product_invalid_id`.

The product schema carries 36 fields including `prices`, `images`, `categories`, `brands`,
`attributes`, `variations`, `is_in_stock` and `stock_availability`.

## Rules

- **Never treat a 200 as proof.** This host answers HTTP 200 with its full theme homepage for
  any unmatched path. Verify the response `content-type` is `application/json` and that the body
  parses before acting on it.
- **No rate-limit signalling exists.** No `RateLimit` or `Retry-After` headers are emitted and no
  policy is published. Be conservative: this is a company marketing site, not a metered API.
- **Errors** use the WordPress envelope `{code, message, data.status}`, not RFC 9457
  problem+json. See `errors/redefine-meat-problem-types.yml`.
