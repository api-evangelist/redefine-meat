---
name: Build a Redefine Meat cart
description: >-
  Assemble a WooCommerce Store API cart on redefinemeat.com - add, update and remove items,
  apply and remove coupons, set the customer address and select a shipping rate - and understand
  the Nonce and Cart-Token requirements that gate every write.
api: openapi/redefine-meat-store-openapi.json
base_url: https://www.redefinemeat.com/wp-json/wc/store/v1
auth: nonce + cart-token
operations:
- getWcStoreV1Cart
- postWcStoreV1CartAddItem
- postWcStoreV1CartUpdateItem
- postWcStoreV1CartRemoveItem
- getWcStoreV1CartItems
- getWcStoreV1CartItemsByKey
- postWcStoreV1CartApplyCoupon
- postWcStoreV1CartRemoveCoupon
- getWcStoreV1CartCoupons
- postWcStoreV1CartUpdateCustomer
- postWcStoreV1CartSelectShippingRate
generated: '2026-08-05'
method: generated
source: openapi/redefine-meat-store-openapi.json
---

# Build a Redefine Meat cart

## 0. Get a session first — this is the step that fails

Reading the cart is anonymous: `getWcStoreV1Cart` (`GET /wc/store/v1/cart`) returns 200 with an
empty cart for an unauthenticated caller.

**Every write is not.** Posting to `/wc/store/v1/cart/add-item` without a `Nonce` header returns:

```
HTTP 401
{"code":"woocommerce_rest_missing_nonce","message":"Missing the Nonce header. This endpoint requires a valid nonce."}
```

The session is carried by two headers, both declared in this host's CORS policy:

- `Nonce` — required on writes.
- `Cart-Token` — returned in the response headers (it is listed in
  `access-control-expose-headers`) and must be echoed back to keep the same cart.

There is no documented way to mint a `Nonce` outside a browser session on this site — Redefine
Meat publishes no developer documentation. Treat cart writes as a browser-context flow, not a
server-to-server one.

## 1. Add an item

`postWcStoreV1CartAddItem` (`POST /wc/store/v1/cart/add-item`) with `{"id": <product id>,
"quantity": <number>}` and, for variable products, `variation: [{attribute, value}]`.

Get the product id from `getWcStoreV1Products` — see the catalog skill.

## 2. Adjust the cart

- `getWcStoreV1CartItems` lists items; each item's `key` is a **32-character token**, not the
  product id. `getWcStoreV1CartItemsByKey` reads one.
- `postWcStoreV1CartUpdateItem` changes quantity; `postWcStoreV1CartRemoveItem` removes one.
- `postWcStoreV1CartApplyCoupon` / `postWcStoreV1CartRemoveCoupon`, with
  `getWcStoreV1CartCoupons` to read what is applied.

## 3. Address and shipping

`postWcStoreV1CartUpdateCustomer` sets billing/shipping address, which is what makes
`shipping_rates` calculable. Then `postWcStoreV1CartSelectShippingRate` picks one.

Read `needs_shipping`, `has_calculated_shipping` and `totals` off the cart object to know where
you are.

## Rules

- **No idempotency.** There is no idempotency key on any of the 845 routes this host publishes.
  A retried `add-item` adds the item **again**. Never blind-retry a cart write — re-read the cart
  with `getWcStoreV1Cart` and reconcile.
- **Do not proceed to checkout autonomously.** `postWcStoreV1Checkout` submits a real order
  against a real company. Treat it as requiring human confirmation.
- **Watch for the soft-404.** Any unmatched path returns HTTP 200 with the theme homepage.
  Validate `content-type: application/json` before parsing.
