# Store work through the Blanxer MCP

The site's code only reads Blanxer. Everything you change in the store while building, you change through the Blanxer MCP. Its tools act as the signed-in person, with that person's role.

How to connect:
- **Claude Code:** comes with this plugin, or `claude mcp add --transport http blanxer https://mcp.blanxer.com/mcp`.
- **Cursor / Codex / claude.ai:** add `https://mcp.blanxer.com/mcp` as a remote MCP server.

The merchant signs in to Blanxer in the browser. You never see a key.

**Connect it first, before any other work.** Claude Code only loads a newly added MCP server when it starts. If you have to add it, ask the merchant to restart right away: type `/exit`, then run `claude -c` from the same folder Claude Code was started in. That reopens this conversation with its history; a plain `claude` would start an empty one. Signing in to an already-added server through `/mcp` needs no restart.

| While building the site you… | MCP tool | The site reads it from |
|---|---|---|
| Check SEO gaps | `blanxer_audit_seo` | — |
| Write product titles and descriptions for search | `blanxer_update_product_status_seo`, `blanxer_bulk_update_products` | product SEO fields → page metadata |
| Write category and brand SEO | `blanxer_bulk_update_categories_seo`, `blanxer_update_category`, `blanxer_update_brand` | `check_domain.categories`, brand details |
| Verify Google Search Console, add GA/GTM | `blanxer_set_site_verification` | store plugins → `verificationMetadata` (meta tags) and `<Pixel>` (GA, GTM, Clarity, Meta Pixel) |
| Add products the merchant describes or sends photos of | `blanxer_create_product`, `blanxer_bulk_create_products`, `blanxer_bulk_import_products`, `blanxer_upload_product_image`, `blanxer_request_image_upload` | product endpoints |
| Create categories and brands for the menus | `blanxer_create_category`, `blanxer_create_brand`, `blanxer_reorder_categories` | categories / brands |
| Set up launch offers and codes | Offers stores: `blanxer_create_offer` (below). Other stores: `blanxer_create_coupon` | `/public/promotions` (Offers) or coupons |
| Fix delivery places, charges and COD | `blanxer_get_delivery_charges`, `blanxer_update_delivery_charges` | delivery table |
| Change the store name or description (shared with the Blanxer site) | `blanxer_update_store_profile` | `check_domain.name`, `description` |
| Change logo, favicon, brand name or colours (shared with the Blanxer site) | `blanxer_update_appearance` | `check_domain.customization` |
| Write blog posts | `blanxer_create_blog_post` (empty post), then its body with `blanxer_add_section` / `blanxer_update_section` using `{ post_id }`, then `blanxer_set_blog_post_status` | blog |
| **Save the website link** | `blanxer_set_storefront_url { url }` → preview with `confirm_token` → after the merchant's yes `{ dry_run: false, confirm_token }`. A bare `mystore.com` becomes `https://mystore.com`; `*.blanxer.io` addresses are refused | — |
| Get the feed and sitemap links | `blanxer_get_store_context` (only once a website link is saved) | — |
| Retire the old subdomain site | `blanxer_update_appearance { store_visibility: 'hidden_on_subdomain' }` | — |

Don't use these to design a custom site, because they change the Blanxer-hosted site:
- page tools (create, update, duplicate and delete page);
- `blanxer_add_section` and `blanxer_update_section` (except with `{ post_id }` for blog post bodies);
- `blanxer_apply_page_template`;
- navigation and footer editors.

For the whole flow, `blanxer_how_to { operation: 'custom_website' }`.

## Launch offers

Discounts, codes, free delivery over an amount, buy more save more, buy X get Y, free gifts and bundles are **offers**, on stores with the **Offers** plugin.

- `blanxer_get_store_context` says whether the store is on Offers (`offers.enabled`). Every plan can have it. The owner or a manager turns it on in the Blanxer dashboard (Offers in the menu); it can't be turned on through the MCP. Turning it on moves the store's coupons into Offers with the same codes and counts.
- Not on Offers: a code is a coupon (`blanxer_create_coupon`, website checkout only). Everything else needs Offers.
- On Offers: `blanxer_how_to { operation }` with `create_discount_code`, `create_automatic_offer`, `buy_more_save_more`, `buy_x_get_y`, `free_gift`, `bundle_offer` or `offer_performance` gives the exact call.

| Tool | What |
|---|---|
| `blanxer_list_offers` | Offers with state, code and numbers. Check a code isn't taken first |
| `blanxer_get_offer` | One offer's rules in plain words, codes and recent orders |
| `blanxer_create_offer` | `amount_off`, `free_delivery`, `tiers`, `bxgy`, `bundle`, `gift`; automatic or with a `code` |
| `blanxer_update_offer` | Only what changes (the tool merges it) |
| `blanxer_set_offer_status` | Pause / resume; archive is preview-first and final |
| `blanxer_quote_cart` | A manual or POS cart priced with the offers (not the website) |
| `blanxer_get_offer_performance` | Orders, revenue, discount and gifts per offer |

- Show the merchant the offer (what the shopper gets, products, dates, limits) and create it after their yes.
- For the website, the offer's `channels` must include `web` (the default is all three) and `display.public` stays on for offers the site should list. New offers show on the site without a redeploy.
- Check it in the site's cart: the totals come from the quote. Never compute a discount for the site (rule 13).

Confirm before bulk changes (rule 14).
