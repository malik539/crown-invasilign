# Crown Dental Care — Invisalign PPC Landing Page (Rockville)

Dedicated Google Ads landing page for the **$1,500 OFF Invisalign + Free Consultation ($200 value)** campaign.

| File | Purpose |
| --- | --- |
| `index.html` | The landing page (single file: HTML + critical CSS + ~120 lines of JS, no frameworks) |
| `thank-you.html` | Post-submission page; fires the `invisalign_lead_thank_you` dataLayer event for Google Ads conversion tracking |
| `assets/img/` | Optimized, responsive WebP image sets generated from the approved assets |
| `assets/src/` | Original approved assets (not referenced by the page) |

## Page structure

1. Promo bar + compact header (logo, `tel:3014506730`, Book Free Consultation)
2. Hero + the **single** lead form + trust strip
3. What to Expect During Your Initial Invisalign Consultation (30 minutes, 5 approved items, compact comfort amenities)
4. Why Invisalign at Crown Dental Care (aligner image split layout, approved copy)
5. Meet Dr. Thy Nguyen ("Dr. T") — approved bio and credentials
6. Reviews (Google 5.0 asset from the source, Winnie T. and Alexandria T., team photo)
7. Final offer summary + location card + responsive Google Business Profile map
8. Footer and mobile sticky CTA (Call Now / Claim $1,500 Off)

Every later CTA smooth-scrolls to `#offer-form` and focuses the first field. There is no second form.

## Before go-live: wire the integrations

The source page (`smilehub.dentistinrockville.com`) could not be fetched from the build environment, so the existing form embed and tag IDs could not be copied. All integration points are isolated in **one config block** at the top of `index.html`:

```js
window.CDC_CONFIG = {
  gtmId: "GTM-XXXXXXX",          // 1. Google Tag Manager container ID
  formEndpoint: "",              // 2. LeadConnector / GHL inbound webhook (or any CRM endpoint)
  formEncoding: "json",          // "json" or "form" (x-www-form-urlencoded)
  thankYouUrl: "thank-you.html", // 3. redirect after a successful submission
  campaign: "Invisalign PPC - Rockville"
};
```

1. **GTM** — paste the real container ID (also in `thank-you.html` and both `<noscript>` iframes). GTM is only injected once a real ID is set, so a placeholder never breaks the page. Google Ads conversion, GA4, Meta Pixel and call tracking should all run from GTM.
2. **Form endpoint** — paste the LeadConnector/GHL inbound-webhook URL (Automations → Workflows → Inbound Webhook) or the existing form's POST endpoint. Field names posted:
   `first_name, last_name, phone, email, treatment_timing, main_concern, dental_insurance, consent, full_name, source, campaign, offer, page_url, utm_source, utm_medium, utm_campaign, utm_term, utm_content, gclid, submitted_at`.
   If `formEndpoint` is left empty the page still validates, pushes the `generate_lead` event (with the non-PII answers) to `dataLayer`, and redirects to the thank-you page, so a GTM-based capture also works. A console warning is logged.
3. **Prefer to keep the existing GHL form widget?** Replace the `<form id="invisalign-form">…</form>` block inside `.card` with the existing `<iframe src="https://api.leadconnectorhq.com/widget/form/…">` embed and its `form_embed.js` script. Styling of the card, headings and the rest of the page is unaffected. Keep the submit button label as **Claim My Invisalign Offer** in the GHL form builder.

### dataLayer events available to GTM

| Event | When |
| --- | --- |
| `cta_click` (`cta_location`, `cta_text`) | Any "Claim/Book" button |
| `phone_click` (`cta_location`) | Any `tel:3014506730` link (header, hero, form, final, location, footer, sticky) |
| `generate_lead` | Successful form submission (before redirect) |
| `invisalign_lead_thank_you` | Thank-you page view — recommended trigger for the Google Ads conversion |

### Select options to confirm

The approved content lists the three question fields but not their dropdown options. The options in `index.html` are sensible defaults; align them with the live GHL form's options if they differ:

- When are you looking to start treatment? — As soon as possible / Within 1–3 months / Within 3–6 months / Just exploring my options
- What's your main concern? — Crowded or crooked teeth / Gaps between teeth / Bite alignment / Teeth have shifted since braces / Other, not sure
- Do you have dental insurance? — Yes / No / Not sure

## Locked campaign facts (do not change)

- Offer: **Start Invisalign and Receive $1,500 OFF**; **Free Consultation — $200 Value**; **$1,700 Total Savings**
- Financing: **0% Interest Financing Available. Third-Party Financing Offered to Qualified New Patients.**
- Phone: **301-450-6730** everywhere, links `tel:3014506730` (the emergency campaign number is not used)
- Address: **1900 Chapman Ave Ste 100, Rockville, MD 20852**
- Consent text and privacy-policy link (`https://dentistinrockville.com/privacy-policy/`) preserved verbatim from the approved content

## Notes

- `index.html` carries `<meta name="robots" content="noindex, follow">` so the PPC page does not compete with the main site. Remove it if the page should be indexed.
- Hero image is the practice's own reception photo (preloaded, `fetchpriority="high"`). All below-the-fold images are lazy-loaded WebP with `srcset`, width/height set to prevent layout shift.
- Fonts: Fraunces (headings) + Manrope (body) from Google Fonts with `display=swap`; the only other external request is the Google Maps embed, which is lazy-loaded.
- Animations are limited to a soft reveal and hover states and are disabled under `prefers-reduced-motion`.
