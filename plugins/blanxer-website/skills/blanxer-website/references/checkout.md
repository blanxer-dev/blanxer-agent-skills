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

## Store pickup

Some stores (Platinum with advanced inventory; one outlet is enough) let shoppers collect the order at a store. `check_domain` then carries `store_pickup.outlets`. `Checkout` from `@blanxer-dev/storefront` **0.4.0 or later** handles it all:

- the Home delivery / Pick up at store choice and the outlet list;
- per-outlet stock, with another outlet recommended when the picked one is short;
- no delivery charge;
- "Pay at store" in place of Cash on Delivery, or online only / a part paid online first, as the store set it.

Keep it; restyle it through the `bx-pickup*` classes. Never build a pickup choice yourself, and never offer it when `store_pickup` is missing or empty. Older package versions simply show delivery only. Test it with phone `0000000000` like any order.

## Selling from several stores (pooled stock)

A few stores sell online from several of their shops. Blanxer support switches it on per store (Platinum, advanced inventory, at least two shops that can ship). `check_domain` then has `online_fulfilment { pooled, display_stock, show_stores, checkout_plan }`. **When it is missing or `pooled` is false, nothing below applies: don't change a site whose store isn't pooled.** Full spec: https://developers.blanxer.com/api-docs/pooled-stock.md

What changes on a pooled store, and what `Checkout` from `@blanxer-dev/storefront` **0.5.0 or later** does by itself:

- **Stock:** the same `quantity` fields, but the number can be the shops' total. Show it the way the site already does. With `show_stores`, the product page can say "Available at Pokhara, Butwal" (`GET /order/public/store-availability/<store_id>/<product_id>`, bands `in_stock` / `few`, never counts).
- **Delivery comes from the server's checkout plan.** With `checkout_plan: true`, each shop can have its own delivery charge, so **never compute delivery from the delivery table**. `POST /order/public/checkout-plan` returns the parcels (the shop each ships from, its lines, its delivery, its share of the discount and advance), `totals`, `delivery_label` ("Delivery: Pokhara Rs 150 · Butwal free"), the checkout's `advance` and `payable.total`. It holds nothing; 120 per 10 minutes per IP, browser only. Stores that aren't pooled keep the delivery-table calculation.
- **The order** carries the plan's `plan_id` as `checkout_plan_id`. The quoted delivery is final. A stale plan (30 minutes, or the cart changed) is HTTP 409 `CHECKOUT_PLAN_STALE` with a fresh `plan`: show the new total and let the shopper place again. Never place it again for them at a new price.
- **One checkout can become several orders** (one per shop). The response's top-level fields are the whole checkout's (`_id` / `order_number` = parcel 1), plus `checkout_group`, `plan_changed` and `orders[]`. One online payment covers them all (payment start with any of their ids). Unpaid online orders of the checkout all stay `Inactive` until it is paid.
- **Tracking:** the order read adds `checkout_group` and `linked_orders`; the success page lists every order number. Cancel takes `scope: 'order' | 'group'`.

What you must keep:

- **The payment grid stays as it is.** Parcels, "Arrives in 2 parcels" and the delivery line go in the order summary, a payment tile's subtitle or the place-order button, never in a second grid of choices.
- Update the package before Blanxer support switches the store to pooled: an older checkout shows the delivery table's charge while the server charges per shop.
- Restyle through the `bx-plan-*`, `bx-checkout-parcels`, `bx-store-availability` and `bx-cancel-scope` classes; "Available at" is `AddToCart`'s `showStores`. Test with phone `0000000000` like any order; the parcels and delivery show in the checkout before you place it.

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
