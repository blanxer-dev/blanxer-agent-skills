# Images and speed

Goal: Lighthouse mobile performance ≥ 90 and LCP < 2.5 s on home, a product page and checkout.

## Images the merchant gives you (logo, banners, hero, lifestyle, icons, backgrounds)

Prepare every one before it goes into `public/`:

```bash
npx blanxer-storefront images ./incoming ./public/images --max-width 2560
```

| Slot | Width to export (×2 the largest display size) |
|---|---|
| Hero / full-width banner | 2560 |
| Half-width banner | 1600 |
| Card, tile, category image | 800 |
| Logo | SVG if the merchant has one, otherwise PNG/WebP at 2× its display height |
| Icon | SVG |

- Output: AVIF + WebP at quality ~75, with EXIF and location data stripped.
- Originals never go into the repo.
- Show the merchant a before/after size table.

## Loading

- Use `next/image` everywhere, with `width` and `height` (or `fill` with a sized parent) and correct `sizes`.
- The main image of each page (the largest one in the first screen, the LCP image) gets `preload`; Next.js 16 replaced `priority` with it (`loading="eager"` + `fetchPriority="high"` also work). Everything else loads lazily (the default).
- No CSS background images for main content. No autoplay video backgrounds or GIF banners unless the merchant insists after hearing the speed cost.

## Product photos

- **They live in Blanxer, not in the site.** If the merchant sends product photos, upload them to the product through the MCP (`blanxer_upload_product_image`, or `blanxer_request_image_upload` for many). Blanxer trims, crops and converts them to WebP, and they then appear in the catalog, the product feed and the POS.
- On the site, show product photos with `next/image` and the package's `blanxerImageLoader`, which requests resized copies.

## Fonts

`next/font` (self-hosted), at most 2 families, only the weights you use.

## What `blanxer-storefront check` enforces

It fails on:
- any file in `public/` over 300 KB or wider than 2560 px;
- JPEG/PNG where AVIF/WebP should be used;
- a raw `<img>` instead of `next/image`;
- an `<Image>` without `width`/`height` or `fill`.

It warns when no image is marked as the page's main image.

`--lighthouse <url>` adds the speed check against a running production build.
