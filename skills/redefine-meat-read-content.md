---
name: Read Redefine Meat editorial content
description: >-
  Pull Redefine Meat's public editorial content through the WordPress REST API - blog posts,
  pages, media, taxonomies, and the custom post types behind its press releases, news, recipes
  and product pages.
api: openapi/redefine-meat-content-openapi.json
base_url: https://www.redefinemeat.com/wp-json/wp/v2
auth: none (reads)
operations:
- getWpV2Posts
- getWpV2PostsById
- getWpV2Pages
- getWpV2Media
- getWpV2Categories
- getWpV2Tags
- getWpV2Pr
- getWpV2Product
- getWpV2Search
- getWpV2Types
generated: '2026-08-05'
method: generated
source: openapi/redefine-meat-content-openapi.json
---

# Read Redefine Meat editorial content

Collection reads on `wp/v2` are anonymous — `GET /wp/v2/posts?per_page=1` returned HTTP 200 with
`x-wp-total: 4` on 2026-08-05.

## 1. Find out what post types exist

Call `getWpV2Types` (`GET /wp/v2/types`). Redefine Meat's editorial content mostly lives in
custom post types rather than core `posts`: `product`, `pr` (press releases), plus news, recipe
and event types visible in the site's sitemaps.

## 2. Read a collection

`getWpV2Posts`, `getWpV2Pages`, `getWpV2Pr`, `getWpV2Product`, `getWpV2Media`.

- Paginate with `page` / `per_page` (max 100) and read `X-WP-Total` / `X-WP-TotalPages`.
- Filter with `search`, `slug`, `after`, `before`, `categories`, `tags`, `order`, `orderby`.
- Project fields with `_fields` to keep responses small, and `context=view` (`edit` needs auth).

## 3. Read one item

`getWpV2PostsById` (`GET /wp/v2/posts/{id}`). Follow `featured_media` into
`GET /wp/v2/media/{id}` for imagery, and `author` into `/wp/v2/users/{id}`.

## 4. Search across everything

`getWpV2Search` (`GET /wp/v2/search`) returns `{id, title, url, type, subtype}` across post
types — the cheapest way to locate content without knowing the type up front.

## Rules

- **Reads only.** Every write on `wp/v2` requires an Application Password or a cookie plus
  `X-WP-Nonce`. Do not attempt writes; you have no authorization to modify this company's site.
- **This is a marketing site, not an API product.** There is no changelog, no deprecation policy
  and no SLA. Cache what you fetch and do not build a dependency on route stability.
- **Respect robots.txt.** Redefine Meat explicitly allow-lists 24 named AI/agent crawlers with
  `Allow: /`, but also disallows `/wp-admin/` and several WooCommerce upload paths.
