# Deploy to the merchant's Cloudflare account

The site is Next.js 16 (App Router) running on Cloudflare Workers through `@opennextjs/cloudflare`. The starter already has `wrangler.jsonc` and `open-next.config.ts`.

## Sign in

```bash
npx wrangler login
```

- **Normal case:** a browser opens. The merchant signs in to **their own** Cloudflare account (creating one is free) and approves access. You never see a password.
- **No browser** (remote machine, CI): the merchant creates an API token in Cloudflare → My Profile → API Tokens, using the "Edit Cloudflare Workers" template, and sets it in their own terminal with `export CLOUDFLARE_API_TOKEN=...`. Don't ask them to paste it into the chat.

## Size limit

```bash
./node_modules/.bin/opennextjs-cloudflare build
```

- The compressed worker must be under **3 MiB** on the Workers free plan, or 10 MiB on the paid plan ($5/month). The build prints the size.
- Static assets (images, CSS, JS chunks) don't count toward it.
- If the worker is over the limit, say so before deploying and offer the options:
  - remove heavy server-side dependencies;
  - move work to client components;
  - move to the paid plan.

## Deploy

Before the first deploy:
- in `wrangler.jsonc`, rename `"name": "blanxer-storefront"` to the site's name and the `WORKER_SELF_REFERENCE` service to match;
- make sure `.env` has `BLANXER_STORE` (OpenNext copies `.env` into the Worker).

The starter keeps no Cloudflare cache storage, so a first deploy needs no extra setup. Pages render per request, and the package caches API reads for 60 s per Worker. An R2 incremental cache is commented in `open-next.config.ts` and `wrangler.jsonc` if a busy store needs it.

```bash
./node_modules/.bin/opennextjs-cloudflare build && ./node_modules/.bin/opennextjs-cloudflare deploy
```

The output ends with the live `https://<name>.<account>.workers.dev` link. Open it and check a product page and `/checkout`.

## Link the site back to Blanxer

1. Save the website link:
   - `blanxer_set_storefront_url { url: "https://<live link or custom domain>" }`, or
   - the merchant opens Blanxer → avatar menu → BlanxerAPI → Your website.

   Blanxer then uses this link in SMS tracking links, the delivered email, flash-sale alerts, the product feed and the sitemap.
2. Register the product feed (`https://api.blanxer.com/public/product-feed/<store_id>.xml`) in Meta Commerce Manager and/or Google Merchant Center.
3. Submit `https://<site>/sitemap.xml` in Google Search Console, and verify ownership with `blanxer_set_site_verification`.

## Custom domain

1. The domain must be on the merchant's Cloudflare account. Add it as a site there, and change the nameservers at the registrar if Cloudflare asks.
2. Workers & Pages → the worker → Settings → Domains & Routes → Add → Custom domain.
3. Update the Blanxer link with step 1 above, using the custom domain.

**If the domain currently points at Blanxer** (Blanxer's own custom-domain setup), moving it to the Worker switches all shoppers on that domain from the Blanxer-hosted site to the new site. Explain that, and let the merchant decide.

## Retiring the old subdomain site

Optional, and only after the link is saved:
- `blanxer_update_appearance { store_visibility: 'hidden_on_subdomain' }`, or
- Blanxer → Appearance → Branding → Site under construction → Sub-domain.

Every page on `<sub>.blanxer.io`, including tracking links already sent for older orders, then shows a placeholder. It can be switched back at any time.

## Redeploying later

Edit, run `next build` and `npx blanxer-storefront check`, then run the deploy command again.

Store data changes (products, prices, stock, SEO text) need no redeploy. They appear within about a minute.
