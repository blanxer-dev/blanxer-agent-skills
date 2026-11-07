# Store work through the Blanxer MCP

The site's code only reads Blanxer. Everything you change in the store while building, you change through the Blanxer MCP. Its tools act as the signed-in person, with that person's role.

How to connect:
- **Claude Code:** comes with this plugin, or `claude mcp add --transport http blanxer https://mcp.blanxer.com/mcp`.
- **Cursor / Codex / claude.ai:** add `https://mcp.blanxer.com/mcp` as a remote MCP server.

The merchant signs in to Blanxer in the browser. You never see a key.

**Connect it first, before any other work.** Claude Code only loads a newly added MCP server when it starts. If you have to add it, ask the merchant to restart right away: type `/exit`, then run `claude -c` from the same folder Claude Code was started in. That reopens this conversation with its history; a plain `claude` would start an empty one. Signing in to an already-added server through `/mcp` needs no restart.

| While building the site you… | MCP tool | The site reads it from |
|---|---|---|
| Check SEO gaps | `blanxer_audit_seo` (see [Search engines](#search-engines-the-seo-plugin)) | — |
| Write product titles and descriptions for search | `blanxer_update_product_status_seo`, `blanxer_bulk_update_products`, or BlanxerAI: `blanxer_start_seo_ai_job` | product SEO fields → page metadata |
| Write category and brand SEO | `blanxer_bulk_update_categories_seo`, `blanxer_update_category`, `blanxer_update_brand` | `check_domain.categories`, brand details |
| Store-wide search settings (homepage title, business info, verification codes) | `blanxer_get_seo_settings`, `blanxer_update_seo_settings` | `check_domain.seo` |
| Hide a page from Google, or set its canonical | `no_index` / `canonical_url` on the product, category, brand, page and blog update tools | the entity's `no_index` / `canonical_url` |
| Photo alt text | `blanxer_update_product { image_alts }`, or BlanxerAI `target: 'image_alts'` | product `image_alts` (aligned with `image_urls`) |
| Redirect old addresses | `blanxer_add_redirect`, `blanxer_import_redirects`, `blanxer_list_redirects`, `blanxer_remove_redirect` | `GET /public/redirect/:store_id?path=` |
| Verify Google Search Console, add GA/GTM | `blanxer_update_seo_settings { verification }` (newer Blanxer) or `blanxer_set_site_verification`; GA/GTM always `blanxer_set_site_verification` | `check_domain.seo.verification`, store plugins → `verificationMetadata` (meta tags) and `<Pixel>` (GA, GTM, Clarity, Meta Pixel) |
| Add products the merchant describes or sends photos of | `blanxer_create_product`, `blanxer_bulk_create_products`, `blanxer_bulk_import_products`, `blanxer_upload_product_image`, `blanxer_request_image_upload` | product endpoints |
| Create categories and brands for the menus | `blanxer_create_category`, `blanxer_create_brand`, `blanxer_reorder_categories` | categories / brands |
| Set up launch offers and codes | Offers stores: `blanxer_create_offer` (below). Other stores: `blanxer_create_coupon` | `/public/promotions` (Offers) or coupons |
| Fix delivery places, charges and COD | `blanxer_get_delivery_charges`, `blanxer_update_delivery_charges` (pooled stores: per shop too, see [Selling from several stores](#selling-from-several-stores-pooled-stock)) | delivery table (pooled stores: the checkout plan) |
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

## Search engines (the SEO plugin)

Newer Blanxer has an **SEO plugin** (free on every plan; BlanxerAI work uses the store's BlanxerAI allowance). On an older Blanxer these tools answer "needs the next backend release" and `blanxer_audit_seo` runs its older checks. `blanxer_how_to { operation: 'improve_seo' }` gives the whole recipe.

**Audit.** `blanxer_audit_seo` returns a 0–100 score, parts (products, categories, brands, pages, blog, site), issues by severity (high first) with a count, examples and the fix, and a 30-day score history. Each fix names its tool: a BlanxerAI job, `blanxer_fix_slugs`, an SEO setting, or a dashboard link. Passing `scope` runs the older local checks instead.

**Settings** (`blanxer_get_seo_settings` → `blanxer_update_seo_settings`, owner/manager; pass only what changes, nested fields are merged; `dry_run: true` shows old → new):

| Setting | What it does |
|---|---|
| `title`, `description`, `share_image` | The homepage's search title (≤60), meta description (~120–160) and share picture |
| `title_suffix` | Adds " \| Store" to every page title (default on) |
| `language` | `en`, `ne` or `en-ne` |
| `business` | `type` (schema.org, e.g. `ClothingStore`), `phone`, `email`, `address { street, city, region, postal, country }`, `hours`, `price_range`, `map_url` — Google's business info. Facts from the merchant only |
| `verification` | `google`, `bing`, `yandex`, `pinterest`: the `content="…"` value from each service's HTML-tag method (a pasted `<meta>` tag works too). Then the merchant presses Verify there and submits `/sitemap.xml` |
| `indexnow` | Tells Bing / Yandex about new and changed pages (default on; production only) |
| `ai_training` | `false` blocks AI-training crawlers; AI search crawlers stay allowed |

A custom site reads these from `check_domain.seo` (the IndexNow key there is public by design). Blanxer has no Google sign-in: Search Console is set up by the merchant, with the verification code saved here.

**Per page.** `no_index: true` on `blanxer_update_product`, `blanxer_update_category`, `blanxer_update_brand`, `blanxer_update_page` or `blanxer_update_blog_post` asks search engines not to index it (`noindex, follow`) and drops it from the sitemap — for thin, test or private pages, never for pages the merchant wants found. `canonical_url` (an absolute URL; `'none'` clears) is advanced: only when the merchant asks. A custom site should honour both: `noindex` meta and leave the page out of its sitemap; use `canonical_url` as the canonical when set.

**Photo alt text.** `blanxer_update_product { image_alts: [...] }` — one per photo, in `image_urls` order; a shorter list keeps the stored alts for the rest. Describe what the photo shows. A custom site uses `image_alts[i]` and falls back to the product name.

**Readable addresses.** `blanxer_fix_slugs` (dry run first, then `{ dry_run: false, confirm_token }`) gives products, categories, brands, pages and blog posts with random or Nepali-script slugs an English-letter slug from their name (राम्रो चिया → `ramro-chiya`). New Nepali names get such slugs automatically.

**Redirects.** A slug change in Blanxer makes a 301 from the old address by itself (and rewrites older redirects so there are no chains) — don't add those by hand. For a store moving from another site: `blanxer_import_redirects { rows: [{ from, to }] }` (up to 2,000, dry run first) or `blanxer_add_redirect { from, to, code }` (301 default, 302 temporary); `from` is a path (a full URL is cut to its path, lower-cased), `to` a path or a full URL. `blanxer_list_redirects` shows them with hits; `blanxer_remove_redirect` takes the id or the `from`. A custom site should ask `GET /public/redirect/:store_id?path=<path>` before answering 404 and redirect to `{ to, code }` when it gets one.

**BlanxerAI SEO text.** `blanxer_start_seo_ai_job { target, ids?, only_missing, language, instructions? }`:
- `target`: `products`, `categories`, `brands`, `pages` (title ≤60 + description ≤155), `image_alts`, or `store` (the homepage settings).
- It only **proposes**. `blanxer_get_seo_ai_job { job_id }` until `ready`, show the merchant the proposals (current → proposed), and only on their yes `blanxer_apply_seo_ai_job { job_id, item_ids? }` (dry run, then `{ dry_run: false, confirm_token }`).
- Fields the merchant wrote are never overwritten unless you pass `ids` AND `only_missing: false`. A field changed after the proposal is skipped ("changed since").
- `blanxer_undo_seo_ai_job { job_id }` puts the old values back.
- Refused (not set up, plan, allowance used up): tell the merchant as it is and don't retry; the template tools above still work.

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

## Selling from several stores (pooled stock)

Some stores sell online from several of their shops: the website's stock is the shops' total, the server picks which shop sends each order, and a cart no single shop can fill becomes one checkout with several orders. It is opt-in. **On every other store none of this applies and nothing changes**, so don't bring it up unless the store has it or the merchant asks.

- **Who switches it on:** Blanxer support allows it per store (Platinum, advanced inventory, at least two shops that can ship). Then the owner switches it on, picks the pool shops, the routing per city and the website display in the dashboard: **Setting → Checkout → Store pickup & fulfilment**. Those are dashboard-only; the MCP can't change them.
- **The site's code doesn't change** for it: the package's checkout reads `online_fulfilment` and uses the server's checkout plan (`references/checkout.md`).
- `blanxer_how_to { operation }` with `set_up_pooled_stock`, `location_delivery_charges` or `split_orders` gives the exact steps for this store.

**Is it on?** `blanxer_get_store_context` → `online_fulfilment { pooled, allowed, eligible, blockers[], warnings[], pool_outlets, display }`.

- `allowed: false` → tell the merchant Blanxer support switches it on.
- `blockers` say why it can't be switched on yet, and `warnings` what to fix (for example a shop with no Tigg location). Show them as they are.

**Shops** (owner / manager):

| Tool | What |
|---|---|
| `blanxer_list_outlets` | Each shop with its city, online buffer, delivery label and pool status |
| `blanxer_update_outlet { outlet, city, online_buffer, courier_bindings, delivery_mode, delivery_adjust }` | Only what changes. `city` must be one of the store's delivery places (routing uses it). `online_buffer` = units always kept for walk-in customers, never sold online. `courier_bindings` = which courier pickup the shop ships with. `delivery_mode` `store` (the store's rates), `store_plus` (the store's rates + `delivery_adjust { basis: 'amount' \| 'percent', value }`, e.g. +Rs 50 or +20%), `own` (its own rates) or `free` |

**Delivery charge per shop.** A checkout pays one charge for each shop its items come from, each at that shop's rates. Cash on delivery per place stays store-wide.

- `blanxer_get_delivery_charges { outlet }` → that shop's effective rates; `outlet: 'all'` → every shop's label ("Same as store", "Store +Rs 50", "Own rates · 12 cities changed", "Free").
- `blanxer_update_delivery_charges { outlet, mode, adjust, defaults, place_updates, reset_places, copy_from, apply_to }`:
  - `mode` / `adjust` as on the outlet: `{ outlet, mode: 'store_plus', adjust: { basis: 'amount', value: 50 } }` = the store's rates + Rs 50 from that shop;
  - `own` rates are changes on top of the store's: `defaults` and `place_updates` set cells, `reset_places` puts cities back to the store's. Cities the shop never changed keep following the store;
  - `copy_from: 'store' | <outlet>` starts this shop from the store's or another shop's setup; `apply_to: [<outlets>]` copies this shop's setup to others in one go;
  - always **preview → the merchant's yes → apply with the `confirm_token`**. The answer has an `undo_token` to put it back. The preview says when it would replace a shop's own changes.

**Orders** (manual orders and the order list):

- `blanxer_quote_cart` on a pooled store returns `parcels[]` (the shop each ships from, its lines and delivery) and a `checkout_plan_id`. Show the merchant the parcels and the total.
- `blanxer_create_order { …, checkout_plan_id }` charges what that quote said; it returns `orders[]` when the cart split. `outlet` forces one shop (it fails when that shop is short).
- `blanxer_list_orders { split, checkout_group, outlet, to_pack }`: split checkouts, one checkout's orders, one shop's orders, what a shop still has to pack. `blanxer_get_order` → `checkout_group` ("Parcel 1 of 2") and `linked_orders`.
- `blanxer_update_order_status { group_scope: 'order' | 'group' }`: cancel only this parcel, or every parcel of the checkout that isn't sent yet. Ask the merchant which; the default is one.

**Between locations:** `blanxer_get_between_locations { period | from/to, location?, received_by?, view?: 'between' | 'shipped_for_online' }`. A read-only record of which shop sent goods for orders paid somewhere else (online or at another shop): units, selling value, cost, delivery, courier fees, by shop and as a "received by × sent by" grid. `shipped_for_online` = what each shop sent for online orders. There is no settle status and Blanxer posts nothing; the merchant settles it their own way. Only pooled stores have the between-locations record.
