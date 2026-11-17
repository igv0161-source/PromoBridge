# PromoBridge payment setup

## Current implementation status

The payments foundation is deployed to Supabase:

- `payment_transactions` records provider-confirmed campaign payment states.
- `promobridge-payments` creates Cashfree hosted checkout orders on the server and verifies signed Cashfree webhooks.
- Payment session tokens are stored in a restricted table and are not exposed through the browser database API.
- Only approved businesses can start checkout, and only after the creator accepts the campaign.
- The campaign amount is read from the database; the browser cannot choose or change it.
- A browser redirect is never treated as proof of payment.
- The portal displays provider-confirmed payment and payout status.
- Proof submissions receive a 24-hour business review window. Final administrator approval creates a payout record with the configured campaign commission split.
- Payout records are tracking records only. No money is automatically transferred to creators.

**Live payments are not enabled until merchant onboarding, provider approval and secure secrets are completed.** Do not set production credentials before the legal and provider checks below.

## 1. Confirm the business model with a qualified adviser and payment provider

PromoBridge connects businesses with independent creators and deducts a platform commission. Before accepting customer money, obtain advice on the Indian entity, tax, invoicing, GST, FEMA/cross-border treatment, consumer refunds/disputes and payment-aggregator/marketplace rules.

Ask the payment provider to confirm in writing that it supports this exact creator marketplace flow, including:
- Collecting campaign fees from a business.
- Identifying the creator as the service provider/beneficiary.
- Platform commission calculation and settlement.
- Refunds, chargebacks, failed campaigns and disputes.
- Cross-border business payments and creator payouts, where eligible.
- Whether a regulated marketplace/split-settlement partner or other legal structure is required.

Do not pool or hold creator funds in a PromoBridge bank account or describe this as escrow. RBI's current Payment Aggregator Directions include restrictions on PA marketplace business; have the structure reviewed before live activation.

## 2. Cashfree merchant onboarding

Cashfree is the first checkout adapter implemented because it offers an India-focused gateway and international payment products for eligible merchants. Availability, methods, currencies, settlement and marketplace/split-payment eligibility depend on Cashfree's approval and contract.

1. Create/onboard the merchant account through the official Cashfree merchant portal.
2. Complete business KYC and provide the PromoBridge website, policies, business model and campaign flow.
3. Ask Cashfree support/sales for explicit approval for creator/marketplace campaign collection and creator disbursements. Do not assume ordinary payment-gateway approval includes marketplace payouts.
4. Start in **Sandbox/Test** mode.
5. Generate API credentials only after the account is approved for the required product.

Official links:
- Cashfree merchant onboarding: https://merchant.cashfree.com/merchants/signup
- Cashfree developer documentation: https://www.cashfree.com/docs
- Cashfree support: https://www.cashfree.com/support
- RBI Payment Aggregator Directions: https://www.rbi.org.in/Scripts/NotificationUser.aspx?Id=12896&Mode=0

## 3. Add secrets in Supabase (never in GitHub or portal.html)

Open Supabase Dashboard → Project Settings → Edge Functions → Secrets.

Add these names with the values issued by Cashfree:

- `CASHFREE_CLIENT_ID`
- `CASHFREE_CLIENT_SECRET`
- `CASHFREE_WEBHOOK_SECRET` — set this to the Cashfree Client Secret used by its PG webhook signature verifier (the same secret as `CASHFREE_CLIENT_SECRET` unless Cashfree gives your account a separate webhook secret).
- `CASHFREE_ENV` = `sandbox`

Do not paste credentials into ChatGPT, GitHub, README files, browser code or public issues. Never expose the Supabase service-role key.

The payment function returns a clear unavailable response if the Cashfree API credentials are absent. It does not simulate a payment.

## 4. Configure Cashfree webhooks

In the Cashfree dashboard, configure the payment notification URL:

`https://zjwfavkhymypkvhlpuva.supabase.co/functions/v1/promobridge-payments`

Enable the relevant payment success/failure notifications and configure the webhook signing secret used for `CASHFREE_WEBHOOK_SECRET`. Cashfree must send the signature and timestamp headers. The function verifies the signature against the raw request body before updating a transaction.

## 5. Sandbox test checklist

Use separate approved business, creator and admin test accounts.

1. Business creates a campaign request.
2. Creator accepts the request.
3. Business opens Bookings, enters a phone number and starts Cashfree sandbox checkout.
4. Complete a sandbox payment.
5. Confirm the portal shows Paid only after a valid provider webhook updates the database.
6. Repeat a webhook delivery and verify it does not create duplicate payment records.
7. Test failed/cancelled checkout; it must not show Paid.
8. Creator submits a real proof URL and screenshot.
9. Business reviews within 24 hours.
10. Admin gives final approval; a payout record is created with the commission and creator amount.
11. Confirm no payout is labelled Paid until a payout provider confirms the actual transfer.
12. Test refunds, disputes and payment reconciliation with the provider before launch.

Do not use live customer money for initial testing.

## 6. Creator payouts

The portal tracks creator payout eligibility and balances after the 24-hour business review and final admin approval. Actual bank/UPI/PayPal payouts are not connected yet.

Before enabling transfers, choose a provider product approved for creator/marketplace disbursements, complete beneficiary KYC, obtain payout API credentials, configure signed payout webhooks, and test failed/returned transfers. Do not collect or store raw bank account numbers, card numbers, passwords, seed phrases or private keys in PromoBridge.

## 7. Other payment methods

- PayPal: not connected. Add only after PayPal approves the business model and a server-side order/capture/refund/webhook adapter is implemented.
- Stripe: not connected. Account availability and marketplace payout features must be confirmed for the Indian entity.
- Crypto: deliberately disabled. Virtual digital asset providers in India have AML/CFT registration and due-diligence obligations; tax and legal treatment must be assessed. Do not accept wallet transfers or store private keys.

## Important

A `paid` payment status means the payment provider confirmed the campaign payment. It does not mean the creator has been paid. A `not_configured` payout status means the creator transfer provider is not connected. No fake payment or payout states are generated.
