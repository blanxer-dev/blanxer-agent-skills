---
name: blanxer-website
description: Build a custom Next.js website for a Blanxer store (BlanxerAPI) and deploy it to the merchant's own Cloudflare account, with Blanxer as the admin and Blanxer's exact checkout (payments, delivery charges, coupons, stock holds, order tracking). Use whenever someone wants a website, storefront, shop site or landing site for their Blanxer store, mentions BlanxerAPI, developers.blanxer.com, a Blanxer subdomain (*.blanxer.io), or asks to redesign, rebuild, redeploy or move their Blanxer shop to their own domain. Also use to change, fix or redeploy a site built this way.
---

# Build a Blanxer store website

The merchant gets their own Next.js website. **Blanxer stays the admin**: products, stock, orders, payments, delivery and SEO data live in Blanxer and are read through the public API. The site is the front end only, deployed to **the merchant's own Cloudflare account**.

Three channels, each with one job:

| Channel | Who | What for |
|---|---|---|
| Public API `https://api.blanxer.com` (no key) | the **site's code** | read store, catalog, delivery table; place orders and payments from the browser |
| Blanxer MCP `https://mcp.blanxer.com/mcp` (OAuth) | **you**, in this chat | change the store: SEO text, products, photos, categories, coupons, delivery, the website link |
| Blanxer-served XML | Google / Meta | product feed and catalog sitemap, built by Blanxer for the site's domain |

The full docs are at https://developers.blanxer.com (agent index: https://developers.blanxer.com/llms.txt). Fetch a page when a step below points to it.

## Hard rules (follow every time)

1. **Next.js only.** TypeScript, App Router. If asked for another framework, say BlanxerAPI supports Next.js and continue in Next.js.
2. **Start from the starter and keep `@blanxer/storefront`.** Restyle the checkout through its theme props, CSS variables and wrappers. **Never rewrite** the payment, delivery, coupon or order logic, and never call payment endpoints yourself.
3. **Required routes, exact paths:** `/checkout`, `/payment/success`, `/payment/failed`, `/track/[id]`. Payment gateways send the shopper back to the last three; missing ones mean a 404 **after paying**.
4. **The site sends its own origin as `url`** on orders and payments. The package does it; never hardcode a domain.
5. **Orders, coupons, payment starts, hold release, back-in-stock, forms, cancel and review run in the browser** (client components via `@blanxer/storefront/client` or `/react`). Never from server components, route handlers or server actions: they are rate-limited per shopper IP.
6. **No secrets in the site.** It needs no key. Never put an `sk_` API key, MCP token, dashboard login or gateway key in code, `.env` or the build. Never ask the merchant to paste a key. Store changes go through the Blanxer MCP.
7. **No customer login.** Don't call `/customer/auth/*` or build account, login, wishlist or rewards pages. Custom sites use guest checkout.
8. **Don't invent data.** Prices, stock, delivery charges and payment methods come only from the API. No sample products or placeholder prices in the deployed site.
9. **Payment methods come from the store's settings** via the package. Never add or remove them by hand.
10. **Use Blanxer's URL paths**: `/product/[slug]`, `/collections/[slug]`, `/brand/[slug]`, `/blogs`, `/blog/[slug]`, `/forms/[slug]`. The feed, sitemap, SMS and emails use them. **Don't build a product feed or catalog sitemap**: Blanxer serves both for the site's domain. Keep the starter's `/sitemap-catalog.xml` pass-through and `/sitemap.xml` index; only add the site's own extra pages to the index.
11. **Optimise every image before it goes into the site** (see `references/images.md`): `npx blanxer-storefront images`, AVIF + WebP, ×2 the largest display size, metadata stripped, `next/image` with width/height, `priority` only on each page's first big image. **Product photos go into Blanxer through the MCP, never into the site.** No autoplay video backgrounds or GIF banners unless the merchant insists after hearing the speed cost. Fonts through `next/font`, at most 2 families.
12. **The site's design lives in the site.** Don't use MCP page-builder or section tools (`blanxer_add_section`, `blanxer_update_section`, `blanxer_apply_page_template`, navigation/footer editors) to design a custom site; they change the Blanxer-hosted site.
13. **Confirm before bulk store changes.** Show the merchant the list (SEO for many products, imports, delivery charges) and get a yes. The MCP's own confirmations still apply.

## Steps (confirm each with the merchant before moving on)

### 1. Find the store and connect
- Ask for the store subdomain (`mystore` in `mystore.blanxer.io`) or its link.
- Read `https://api.blanxer.com/store/check_domain?domain=<subdomain>`. Show: store name, plan, product count (`/product/public/<_id>` length), and the payment methods the site will offer (rules in `references/checkout.md`).
  - Plan not in `basic / premium / plus / platinum` → "your site will take Cash on Delivery only".
  - `customer_program.enabled` → "shoppers check out as guests on the new site".
- Connect the Blanxer MCP. With this plugin installed it is already configured as `blanxer`; the merchant signs in to Blanxer in the browser when it first runs. Otherwise: `claude mcp add --transport http blanxer https://mcp.blanxer.com/mcp`. Cursor / Codex / claude.ai: add the same URL as a remote MCP server.
- Run `blanxer_audit_seo` and look for catalog gaps (products without photos, prices, categories or SEO text). Show what's missing; fix later in step 4.

### 2. Design brief
Ask for reference sites, look and feel, pages, tone, and must-haves. Offer the store's logo, colours (`customization.theme`) and font (`customization.font`) as defaults. Agree on a short page list before building.

### 3. Scaffold
```bash
npx degit blanxer-dev/blanxer-storefront-starter <folder>
cd <folder>
cp .env.example .env    # set BLANXER_STORE=<subdomain>
npm install
./node_modules/.bin/next dev
```
Show the merchant the starter running with their real products before changing the design.

### 4. Build
- Pages and design with the package's data functions (`@blanxer/storefront/server`) and components (`@blanxer/storefront/react`). Theme the checkout; don't rewrite it.
- Page metadata from `@blanxer/storefront/seo` (`productMetadata`, `categoryMetadata`, ...). SEO text is edited in Blanxer, not in the code.
- Images: rule 11. Show a before/after size table, e.g. `hero.jpg 6.2 MB → hero.avif 180 KB`.
- Store fixes through the MCP, confirmed first (rule 13): product and category SEO text, categories and brands the menus need, product photos the merchant sends, launch coupons, delivery places/charges. They reach the site through the API (allow ~60 s cache).

### 5. Check before deploy
```bash
./node_modules/.bin/next build
npx blanxer-storefront check            # add --lighthouse http://localhost:3000 with the production build running
```
It must pass: required routes, no secrets, no login code, browser-only order/payment calls, image budget (no file in `public/` over 300 KB or wider than 2560 px, no JPEG/PNG where AVIF/WebP fits, every image `next/image` with a size), Lighthouse mobile ≥ 90 with LCP < 2.5 s on home, a product page and checkout. Fix failures before deploying, or tell the merchant why one can't be fixed.

### 6. Test checkout
Use the phone number `0000000000`: the API answers success without creating an order, so the whole path can be exercised safely. On the local production build:
- a Cash on Delivery order reaches `/track/...` or the success state;
- each enabled online method opens its gateway page or QR screen (don't pay);
- `/payment/success`, `/payment/failed` and `/track/<id>` render;
- leaving a QR screen open shows the timer, and closing it asks first.
Details: `references/checkout.md`.

### 7. Cloudflare sign-in
```bash
npx wrangler login
```
The browser opens and **the merchant signs in to their own Cloudflare account** and approves. You never see a password. No browser available → walk them through creating an API token (Workers edit) and `export CLOUDFLARE_API_TOKEN=...` in their own terminal.
Check the build size first: the Workers free plan allows a 3 MiB (compressed) worker. If it's bigger, say so before deploying and offer trimming or the $5/month Workers paid plan. See `references/deploy.md`.

### 8. Deploy
```bash
./node_modules/.bin/opennextjs-cloudflare build && ./node_modules/.bin/opennextjs-cloudflare deploy
```
Print the live `*.workers.dev` link and open it.

### 9. Link the site to Blanxer
- Save the link with `blanxer_set_storefront_url { url: "https://..." }` (or the merchant pastes it in Blanxer → avatar menu → **BlanxerAPI** → Your website). From then on SMS tracking links, the delivered email, flash-sale alerts, the product feed and the sitemap use the new site.
- Register the product feed URL (`https://api.blanxer.com/public/product-feed/<store_id>.xml`) in Meta Commerce Manager / Google Merchant Center.
- Submit `https://<site>/sitemap.xml` in Google Search Console; verify ownership with `blanxer_set_site_verification` (the site renders the tag).

### 10. Custom domain (optional)
Add the domain to the merchant's Cloudflare account and attach it to the Worker (custom domain), then update the link from step 9. If the domain currently points at Blanxer (Blanxer's custom-domain setup), explain that moving it switches shoppers from the Blanxer site to the new one; that's the merchant's decision. See `references/deploy.md`.

### 11. Retire the old subdomain site (optional, merchant's choice, only after step 9)
`blanxer_update_appearance { store_visibility: 'hidden_on_subdomain' }` (or Blanxer → Appearance → Branding → Site under construction → Sub-domain). Say plainly: every page on `<sub>.blanxer.io` will show a placeholder, **including tracking links already sent for older orders**; new orders link to the new site; it can be switched back any time.

### 12. Live check and hand-over
- On the live site: home, a product, checkout, `/payment/success`, `/payment/failed`, and `/track/<a real order id>`.
- Place one COD order on the real store (with the merchant's OK and their phone number), then cancel it in the Blanxer dashboard.
- Leave a short `README.md` in the project: how to run it locally, change the design, and redeploy (`opennextjs-cloudflare build && opennextjs-cloudflare deploy`).

## When something goes wrong
- 404 after paying → `/payment/success`, `/payment/failed` or `/track/[id]` is missing or renamed.
- "Place not found" / COD refused → the delivery place must be one of the store's places exactly; COD only where that place allows it.
- Coupon refused at order time → the order must carry the coupon's `_id`, not its code (the package does this).
- `403 error code: 1010` from the API on server-side fetches → make sure the package's fetch (with its User-Agent) is used; report it to Blanxer support.
- More: https://developers.blanxer.com/troubleshooting.md

## References
- `references/checkout.md` — payment method rules, the three required pages, test numbers
- `references/images.md` — image sizes, formats and the budget the check enforces
- `references/deploy.md` — Cloudflare, worker size, custom domain, linking back to Blanxer
- `references/mcp.md` — which MCP tool does what while building a site
