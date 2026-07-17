# Bhagavad Gita — Illustrated Edition Landing Page

A single, production-ready landing page for the Bhagavad Gita illustrated
edition (combining both source PDFs into one product), built as a static
site in the style of krishnavani.org's landing pages — with Razorpay
payment placeholders ready to activate.

## Files

```
ebook/
├── index.html              Main landing page (all content + SEO + schema)
├── styles.css               Design system (saffron/maroon/cream, responsive)
├── script.js                 Form validation, Razorpay checkout logic, UI behavior
├── success.html              Post-payment success page
├── failure.html               Post-payment failure/retry page
├── robots.txt                 Search engine crawl rules
├── sitemap.xml                 XML sitemap (single URL)
├── google-ads-assets.md         Full Google Ads copy: headlines, sitelinks, keywords
├── .env.example                 Environment variable template (backend only)
├── server/
│   ├── server-stub.js            Optional Node/Express backend for Razorpay
│   └── package.json               Backend dependencies
└── README.md                      This file
```

## 1. Quick start (view the site locally)

No build step needed — it's plain HTML/CSS/JS.

- **Easiest:** double-click `index.html` to open it in a browser.
- **Recommended (avoids relative-path quirks):** serve it locally:
  ```bash
  # from the ebook/ folder
  npx serve .
  # or
  python -m http.server 5500
  ```
  Then visit `http://localhost:5500`.

## 2. Deploying

Any static host works — no server required for the base site:

- **Netlify / Vercel:** drag-and-drop the `ebook/` folder (excluding `server/`), or connect a Git repo.
- **GitHub Pages:** push this folder to a repo and enable Pages.
- **Shared hosting / cPanel:** upload all files except `server/` to `public_html`.

Before going live, update:
- `<link rel="canonical">`, Open Graph `og:url`/`og:image`, and all JSON-LD `@id`/`url` fields in `index.html` — replace `https://www.yourdomain.com/` with your real domain.
- `robots.txt` and `sitemap.xml` — same domain replacement.
- Footer email address and WhatsApp number (`wa.me/910000000000` — replace with your real WhatsApp Business number, including country code, no `+` or spaces).

## 3. Connecting Razorpay (you said you'll send the link later)

The site is wired for **two levels** of Razorpay integration. Pick whichever
matches what you're given.

### Option A — Razorpay Payment Link (simplest, no backend needed)

1. In the Razorpay Dashboard (account: `synapsedigitalsolutions.dm@gmail.com`) → **Payment Links** → create a link for ₹299 (or your final price).
2. Open `script.js`, find:
   ```js
   RAZORPAY_PAYMENT_LINK: 'https://rzp.io/l/PLACEHOLDER-PAYMENT-LINK',
   ```
3. Replace the placeholder with your real link. Save and redeploy.
4. That's it — the "Pay ₹299 & Get Instant Access" button will redirect there once the form is validated.

Until you do this, the button shows a friendly "payment being finalized" message instead of erroring out, so the page is safe to publish today.

### Option B — Inline Checkout (advanced, needs a backend)

Use this if you want the Razorpay popup modal to open **on this page**
instead of redirecting to a hosted link.

1. Get your **Key ID** and **Key Secret** from Razorpay Dashboard → Settings → API Keys.
2. Deploy `/server/server-stub.js` (Node/Express) to any Node host (Render, Railway, Fly.io, a VPS, etc.):
   ```bash
   cd server
   npm install
   cp ../.env.example ../.env   # fill in real RAZORPAY_KEY_ID / SECRET / WEBHOOK_SECRET
   npm start
   ```
3. In `script.js`, set:
   ```js
   USE_INLINE_CHECKOUT: true,
   RAZORPAY_KEY_ID: 'rzp_live_xxxxxxxxxxxx',   // public key — safe to expose in frontend
   CREATE_ORDER_ENDPOINT: 'https://your-backend.example.com/api/create-order',
   ```
   **Never** put `RAZORPAY_KEY_SECRET` in `script.js` or any frontend file — it stays in `.env` on the server only.
4. In Razorpay Dashboard → Settings → Webhooks, add an endpoint pointing to `https://your-backend.example.com/api/razorpay-webhook`, subscribe to `payment.captured` and `order.paid`, and copy the generated **Webhook Secret** into `.env`.
5. Test with Razorpay's [test mode cards](https://razorpay.com/docs/payments/payments/test-card-upi-details/) before switching keys to live mode.

### Test → Production checklist

- [ ] Start with `rzp_test_...` keys, confirm a full test payment end-to-end.
- [ ] Swap to `rzp_live_...` keys only after testing.
- [ ] Confirm webhook signature verification works (check server logs).
- [ ] Confirm `success.html` / `failure.html` messaging matches your actual fulfillment process (e.g. how the ebook file is delivered).
- [ ] Set up an email/WhatsApp automation (Zapier, Make, or your ESP) triggered by the webhook to actually deliver the digital file.

## 4. Forms

The enquiry/checkout form (`#checkout-form` in `index.html`) validates:
Name, Email, Mobile (10-digit Indian format), City, State, optional Message —
client-side, with inline error messages and a success state.

It also fires a non-blocking `POST` to `CONFIG.ENQUIRY_ENDPOINT` (`/api/enquiry`)
so you can capture leads even before Razorpay is fully wired up — implement
that route in `server-stub.js` (already stubbed) to save leads to a database,
Google Sheet, or CRM of your choice.

## 5. SEO

- Title, meta description, Open Graph, Twitter Card tags: in `<head>` of `index.html`.
- Structured data included: `Product`, `FAQPage`, `BreadcrumbList` (JSON-LD).
- Update `priceValidUntil`, `aggregateRating`, and `sku` in the Product schema to real, accurate values before launch — schema must reflect real data to stay compliant with Google's structured data policies.
- **Replace the sample testimonials** (clearly marked with an HTML comment in `index.html`) with real, verifiable customer reviews before publishing — placeholder reviews should never go live as-is.

## 6. Google Ads

All ready-to-import copy — headlines, descriptions, sitelinks, callouts,
structured snippets, keyword lists (with match types), and negative keywords —
is in [`google-ads-assets.md`](./google-ads-assets.md).

## 7. Performance & accessibility notes

- No external images are used (the hero "book cover" is pure CSS) — keeps the page extremely fast and avoids broken-image risk until you add real photography/illustrations.
- To add real images later: use modern formats (WebP/AVIF), set explicit `width`/`height`, and add `loading="lazy"` to any image below the fold.
- Fonts are loaded via `<link>` with `preconnect` for faster first paint.
- All interactive elements (`<details>` FAQ, form fields, buttons) are keyboard-accessible by default.

## 8. Customizing for a different price or product

Change these in one pass:
- `index.html`: all `₹299` / `₹999` occurrences, hero copy, chapter list, schema `price`.
- `script.js`: `CONFIG.AMOUNT_INR` and `CONFIG.PRODUCT_NAME`.
- `google-ads-assets.md`: price-mentioning headlines/descriptions.
