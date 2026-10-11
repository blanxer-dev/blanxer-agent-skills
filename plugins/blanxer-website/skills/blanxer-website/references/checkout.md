# Checkout: what the package does and what you must keep

Use `Checkout` from `@blanxer/storefront/react` on `/checkout`. It reproduces Blanxer's own checkout. Full spec: https://developers.blanxer.com/checkout.md

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
| `/track/[id]` | COD and paid QR orders land here; the order email and SMS link here |

The starter has all three, built from `PaymentSuccess`, `PaymentFailed` and `OrderTracking`. Restyle them; don't move or delete them.

## Delivery

- Places and charges come from `GET /public/delivery_charge/<store_id>`.
- The charge depends on the cart's total weight. The brackets are ≤1, ≤2, ≤3, ≤5, ≤10 and >10 kg. A place value of `-1` means the store default for that bracket.
- The server recalculates everything, so a wrong number on the page can't be charged. It can still confuse the shopper, so keep the package's logic.

## Coupons and stock holds

- Coupons are applied by code. The order must carry the coupon's `_id`. Coupon rules (minimum order, COD-only, online-only, places) are checked on the page and again on the server.
- Online orders reserve stock for a few minutes (`hold_seconds` on the order response). QR screens count this down and check automatically. Closing the screen asks first, then releases the hold. The cart is cleared only once the payment is confirmed.

## Safe testing

- **Phone `0000000000`:** the API answers success but creates no order. Use it for every test before going live.
- **Online methods:** open the gateway page or QR screen to check the flow, then leave without paying.
- **Live check:** one real COD order with the merchant's OK, cancelled afterwards in the dashboard.
