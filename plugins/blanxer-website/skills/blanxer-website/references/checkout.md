# Checkout: what the package does and what you must keep

Use `Checkout` from `@blanxer-dev/storefront/react` on `/checkout`. It reproduces Blanxer's own checkout. Under the place-order button it shows **"Checkout Powered by Blanxer"** (linking to blanxer.com). Restyle it with the `.bx-powered-by` class; never remove or hide it. Full spec: https://developers.blanxer.com/api-docs/checkout.md

## Payment methods the site shows

Cash on Delivery: always offered, but only for delivery places where the store allows COD (place flag `true` or `'Y'`).

Online methods: only when the store's `plan` is `basic`, `premium`, `plus` or `platinum`, and only those the store has turned on:

| Method (`paymentMethod`) | Shown when |
|---|---|
| `QR_Dynamic` (Fonepay QR) | `addons.fonepay_dynamic` and `setting.fonepay_dynamic_key` |
| `FonepayCheckout` (Checkout by Fonepay) | `addons.fonepay_checkout` and `setting.fonepay_checkout_key` |
| `QR` (Blanxer QR) | `addons.qr_request_enabled` |
| `Esewa` | `addons.esewa` and `setting.esewa_secret` |
| `FonePay` | `addons.fonepay` |
| `Khalti` | `addons.khalti` |
| `NPX` | `addons.npx` |
| `nbank_card` (Nabil card) | `addons.nbank_card` |

Never add, hide or rename these by hand (rule 9). A method the merchant turns on later appears by itself.

## The three pages that must exist

| Route | Why |
|---|---|
| `/payment/success` | eSewa, Khalti, Fonepay, NPX and Nabil return here after a successful payment (through api.blanxer.com) |
| `/payment/failed` | the same gateways return here after a failed or cancelled payment |
| `/track/[id]` | COD and paid QR orders land here; order emails and the dashboard's tracking link point here |

The starter has all three, built from `PaymentSuccess`, `PaymentFailed` and `OrderTracking`. Restyle them; don't move or delete them.

## Delivery

- Places and charges come from `GET /public/delivery_charge/<store_id>`.
- The charge depends on the cart's total weight. The brackets are ≤1, ≤2, ≤3, ≤5, ≤10 and >10 kg. A place value of `-1` means the store default for that bracket.
- The server recalculates everything, so a wrong number on the page can't be charged. It can still confuse the shopper, so keep the package's logic.

## Offers (stores on the Offers plugin)

Full spec: https://developers.blanxer.com/api-docs/checkout.md#offers

- `GET /public/promotions/<store_id>` → `{ enabled, data, settings, server_time }`. `enabled: false` = not on Offers: coupons below. `useOffers()` in the package.
- The cart is priced by the server: `POST /public/promotions/<store_id>/quote` with the shopper's lines (`product`, `variant`, `quantity`), `code`, `place`, `payment`, `phone`, `removed_offers`, `gift_choices`. It returns `lines` (per-line `allocations`, `net`, `free_qty`, `added_by` on gift lines), `applications`, `code` (`applied` / `rejected` + `message`), `delivery`, `totals` and `messages`. `useCartQuote()` calls it, debounced. Rate limit 300 per 10 minutes per IP. It holds nothing.
- **Show the quote; never compute a discount** (rule 13). Customization charges aren't in the quote's totals; add them.
- Lines with `added_by` are gifts or free items the server added. Show them ("Free"), never send them back in a quote or an order.
- One code per order. The order sends `code`, `removed_offers`, `gift_choices` and `coupon: ''`. A rejected code fails the order with HTTP 401 `{fields: {code: <message>}}`. The order response has `offers { applications, code, totals }`; the tracking page shows the order's `promotions`.
- Stock isn't the site's concern: the server adds a gift only when it is in stock (simple or advanced inventory alike) and takes it from stock like a paid unit.

## Coupons (stores not on Offers) and stock holds

- Coupons are applied by code (any case: the server upper-cases it). The order must carry the coupon's `_id`. Coupon rules (minimum order, COD-only, online-only, places) are checked on the page and again on the server.
- Applying a coupon holds nothing (40 checks per 10 minutes per IP). A use is claimed when the order is created and given back if it is cancelled, its payment fails or its checkout expires.
- Online orders reserve stock for a few minutes (`hold_seconds` on the order response). QR screens count this down and check automatically. Closing the screen asks first, then releases the hold. The cart is cleared only once the payment is confirmed.

## Safe testing

- **Phone `0000000000`:** the API answers success but creates no order, and the checkout shows "Test order accepted". Use it to test the form, delivery charge, offers or coupons in the cart, and the order request. No gateway or QR screen opens on this path, because there is no order to pay for, and the code isn't checked on it: read a code's result in the cart.
- **Payment screens:** check them once, live, with the merchant's OK:
  1. Place a real order on one online method.
  2. Open the gateway or QR screen and stop without paying. Closing the QR screen releases the stock hold.
  3. Cancel the order in the dashboard.
- **Live COD check:** one real COD order with the merchant's own phone number, cancelled afterwards.
