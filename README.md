# PromoBridge

Creator-first collaboration portal for approved creators and businesses.

## Live pages

- Homepage: https://igv0161-source.github.io/PromoBridge/
- Creator / business portal: https://igv0161-source.github.io/PromoBridge/portal.html

The GitHub Pages URLs are the intended deployment addresses. Confirm deployment status in **Repository → Settings → Pages** if a page does not load.

## Current experience

### Public homepage
- Creator-first product introduction and business/creator entry points.
- Responsive layout and mobile navigation.
- Interactive Three.js collaboration scene with drag rotation, selectable nodes, keyboard controls and a graceful non-WebGL fallback.
- Scroll progress, reveal transitions, and reduced-motion support.
- Enquiry email opens the user's email application.

The 3D scene loads Three.js from jsDelivr and therefore requires an internet connection and browser WebGL support. If the module or WebGL is unavailable, the page falls back to a static/animated scene treatment and the Creator / Brand / Collaboration controls remain available.

### Authenticated portal
- Email/password signup and sign-in through Supabase Auth.
- Profile editing for creator and business accounts.
- Approved creator directory for businesses.
- Campaign requests created by businesses for approved creators.
- Creators can accept or decline requests.
- Accepted creators can submit a JPG, PNG or WebP proof image (maximum 8 MB) with the live post URL.
- Admin-only proof review and restricted contact-review panel.
- Private business/creator chat with realtime message refresh.
- Detected phone numbers are redacted from participant-visible messages; detected originals are restricted to admin review.

## Backend and security

Supabase project ref: `zjwfavkhymypkvhlpuva`.

- Row Level Security is enabled on all seven public tables.
- Anonymous table privileges have been revoked.
- Signed-in users can discover only approved profiles plus their own profile; administrators can review all profiles.
- Only approved business accounts can create campaign requests for approved creators.
- Only the assigned approved creator can accept/decline their request.
- Proof submission is limited to the assigned approved creator after campaign acceptance.
- Proof storage bucket `campaign-proof` is private, limited to 8 MB, and allows JPEG, PNG and WebP.
- Chat conversation/message reads are participant-scoped. Contact reports are admin-only.
- Edge Function `promobridge-chat` is deployed with JWT verification enabled.
- Authorization decisions use database profile roles and approval state, not user-editable metadata.

Phone-number detection is best-effort pattern matching, not a guarantee against every possible obfuscation or written-out number. Do not send passwords, payment details or other sensitive information in chat.

## Not enabled yet

- Payments, escrow or automatic payouts.
- Social account OAuth or automatic publishing.
- Guaranteed creator reach, sales or campaign outcomes.
- Automatic AI campaign matching.

The portal must not be treated as a payment processor. A campaign request, chat or proof approval does not mean money has been transferred.

## Development notes

This is a static HTML/CSS/JavaScript project. The public homepage and portal are separate files:

- `index.html` — marketing homepage and interactive 3D scene.
- `portal.html` — Supabase Auth, profile, campaign, proof, admin and chat interface.

The portal uses the browser-safe Supabase publishable key. Never put a Supabase service-role key or other server secret in either HTML file. The `promobridge-chat` Edge Function uses server-side Supabase secrets.

For local testing, serve the repository through an HTTP server rather than opening the HTML files as `file://` URLs. Check signup, login, profile updates, creator discovery, campaign acceptance/decline, proof upload, admin review, chat, and mobile touch behavior with separate approved business, creator and admin test accounts.

## Verification status

Inline JavaScript syntax and DOM ID references have been statically checked. Supabase RLS, grants, core policies, edge-function status and advisor output were inspected after the latest database updates.

A full browser/device end-to-end test has not yet been completed. Real signup, realtime chat, file upload, WebGL rendering and mobile touch behavior still need a live test using the deployed Pages site and test accounts.

## Security follow-up

Supabase currently reports **Leaked Password Protection Disabled**. Enable leaked-password protection in Supabase Auth password security settings before inviting public users. The performance advisor also reports 11 indexes not yet used; they were retained because the current database has very little activity and removing them would be premature.
