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

Some stores (Platinum, several outlets) let shoppers collect the order at a store. `check_domain` then carries `store_pickup.outlets`. `Checkout` from `@blanxer-dev/storefront` **0.4.0 or later** handles it all:

- the Home delivery / Pick up at store choice and the outlet list;
- per-outlet stock, with another outlet recommended when the picked one is short;
- no delivery charge;
- "Pay at store" in place of Cash on Delivery, or online only / a part paid online first, as the store set it.

Keep it; restyle it through the `bx-pickup*` classes. Never build a pickup choice yourself, and never offer it when `store_pickup` is missing or empty. Older package versions simply show delivery only. Test it with phone `0000000000` like any order.

## Coupons and stock holds

- Coupons are applied by code. The order must carry the coupon's `_id`. Coupon rules (minimum order, COD-only, online-only, places) are checked on the page and again on the server.
- Online orders reserve stock for a few minutes (`hold_seconds` on the order response). QR screens count this down and check automatically. Closing the screen asks first, then releases the hold. The cart is cleared only once the payment is confirmed.

## Safe testing

- **Phone `0000000000`:** the API answers success but creates no order, and the checkout shows "Test order accepted". Use it to test the form, delivery charge, coupons and the order request. No gateway or QR screen opens on this path, because there is no order to pay for.
- **Payment screens:** check them once, live, with the merchant's OK:
  1. Place a real order on one online method.
  2. Open the gateway or QR screen and stop without paying. Closing the QR screen releases the stock hold.
  3. Cancel the order in the dashboard.
- **Live COD check:** one real COD order with the merchant's own phone number, cancelled afterwards.
