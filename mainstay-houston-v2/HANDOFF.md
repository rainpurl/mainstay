# MainStay Suites Houston: HANDOFF

**Version:** v2
**Built:** August 2026
**Deploy target:** Vercel (domain pending, eventually `mainstayhouston.com`)
**Property:** MainStay Suites Texas Medical Center / Reliant Park, 3134 Old Spanish Trail, Houston, TX 77054
**Choice property code:** TX933

---

## What this is

A single-page replacement for the old multi-page ProcessWire site. The old site had 8+ pages, a mailing list form, a search page, a blog, an accessibility widget, and a packages carousel. All of it is gone. This page has one job: get someone to the hotel's own booking page on choicehotels.com with their dates already filled in.

There is deliberately **no navigation menu**. Every path through the page ends at the same button.

---

## Files

```
index.html the entire site, single file, no build step
vercel.json caching, security headers, 301s from the old URL structure
robots.txt points at the sitemap
sitemap.xml one URL
img/ 33 processed photos, WebP with JPEG fallbacks, plus logo assets and og.jpg
```

Deploy by dragging the folder into Vercel, or push to a repo and import it. No framework, no dependencies, no build command. Output directory is the folder itself.

---

## Design system

Both accent colours were sampled directly out of the official MainStay Suites logo PNG, so they are the real brand values, not approximations.

| Token | Hex | Use |
|---|---|---|
| `--navy` | `#003046` | Brand navy. Dark sections, headings, body text on light. |
| `--seafoam` | `#A1D7BE` | Brand mint. Every call to action, the shuttle band, accent text. |
| `--seafoam-deep` | `#2F7A5E` | Darkened seafoam for small text on light backgrounds (contrast). |
| `--paper` | `#EDF1F0` | Page background, a cool neutral pulled from desaturating the seafoam. |
| `--paper-hi` | `#F7F9F8` | Alternating section background. |
| `--ink` | `#08202B` | Body text. |
| `--slate` | `#4A6270` | Secondary text. |

**The old site's gold `#d8a641` is gone.** It was never a MainStay brand colour, it was a local addition. Everything is now navy and seafoam only.

**Type:** Fraunces (display, variable, WONK axis on), Archivo (body), IBM Plex Mono (figures, labels, fine print). All from Google Fonts, one request.

---

## The signature element

The hero is a **stay-length planner**, not a headline and a photo. A slider from 1 to 30 nights that:

1. sets the number of nights and computes the departure date,
2. swaps in a different line of copy depending on the length of stay (short stay talks about the shuttle, seven nights mentions weekly rates, thirty nights tells you to call for a monthly rate),
3. relabels the button to "Check rates for N nights",
4. writes the dates into **every** booking link on the page at once.

This is the one place the design spends its boldness. The reasoning: this property's guests are mostly at the Texas Medical Center for treatment or caregiving, and the variable that actually defines their booking is how long they will be in Houston, not which room they want.

---

## Booking deep link, please verify this

All booking buttons point at:

```
https://www.choicehotels.com/texas/houston/mainstay-hotels/tx933/rates?checkInDate=YYYY-MM-DD&checkOutDate=YYYY-MM-DD&adults=1
```

Two things to know about that URL.

**The `/rates` path.** Your old site's own Book Now links used `.../tx933/rates` rather than the plain property page. That path lands on the availability screen instead of the marketing page, which saves the guest a click. Reusing it here.

**The date parameters are unverified.** Choice does not publish its URL parameter spec and its site blocks automated requests, so the names could not be confirmed. Open the site, move the slider, click through, and check whether the dates carry over to Choice's booking screen.

If they do not, everything you need is in one block at the top of the `<script>` in `index.html`:

```js
var HOTEL = "https://www.choicehotels.com/texas/houston/mainstay-hotels/tx933";
var PATH  = "/rates";     // "/rates" -> availability screen | "" -> property page
var P_IN  = "checkInDate";
var P_OUT = "checkOutDate";
var FMT   = "iso";        // "iso" -> 2026-08-10   |   "us" -> 08/10/2026
var EXTRA = { adults: "1" };
```

Change the parameter names, or flip `FMT` to `"us"`, and every link on the page updates at once. Unknown query parameters get ignored by web apps, so a wrong guess costs nothing: the guest still lands on the correct availability screen. The static `href` values in the markup point at `/rates` too, so this works with JavaScript disabled.

The link uses the `en-us` path rather than the `en-ca` one you sent. Choice geo-redirects anyway, and the guest base here is US medical-centre traffic.

The JSON-LD `sameAs` and `ReserveAction` still reference the canonical property page without `/rates`, which is correct for structured data.

---

## Photography

Every photo is a real photograph of this property. No stock, no staged interiors from other hotels.

They were pulled from the live mainstayhouston.com media library at original resolution (2000×1333 and 2400×1599), then resized to responsive widths, given a light exposure and saturation correction, sharpened after downscale, and exported as WebP with JPEG fallbacks. The old site was serving several of these at 600×450, which is where the low-res look came from. The source files were always bigger.

Photos used: pool and courtyard (hero), lobby, breakfast lounge, fitness room, guest laundry, suite kitchenette, king suite, two-queen suite, queen suite, accessible bathroom.

**One caveat.** These images are dated. They read as early-2010s interiors, and several Google reviews mention rooms feeling outdated. New photography would do more for conversion than anything else on this page. The layout is built to take better images the moment you have them, same filenames, same aspect ratios.

---

## Facts to verify before you point a domain at this

Everything below came from the property's own Choice Hotels listing or its existing site. It is worth a five-minute call to the front desk to confirm, because publishing a wrong detail is the exact problem this site exists to solve.

- **Shuttle hours.** Listed here as Monday to Friday, 7:00 am to 7:00 pm, three-mile radius. Choice's page says 7 to 7. HotelPlanner's listing says 7:00 am to 9:00 pm weekdays plus weekend service 9:00 am to 5:00 pm. These disagree. **Confirm which is current.**
- **Breakfast.** Described here as "grab and go". The property's own marketing copy elsewhere says "free breakfast", one recent Google review praises a grab-and-go setup, another review complains there was no breakfast at all. Pin down what is actually served and adjust the caption in the amenity mosaic.
- **Pool.** Labelled "seasonal". Some listings call it a heated outdoor pool. Confirm.
- **Distances.** The mileage figures are the driving distances Choice supplies to booking partners. Straight-line distance to the Texas Medical Center is about 1.1 miles, so "under 2 miles" is safe, but check the rest.
- **Room count.** Structured data says 18 room types, taken from Choice's live inventory. That is room *types*, not rooms. Fine as written but do not let it become "18 rooms".

---

## Deliberate omissions

- **No nightly price anywhere on the page.** Your stated problem is that other sites show wrong prices. Hardcoding "from $69" here would make this site the next one to go stale. Instead the copy says rates are live on Choice. If you want a price for conversion reasons, add it with a visible "as of" date and put a reminder in a calendar to update it.
- **No aggregate rating in the structured data.** Google requires review markup to come from reviews collected by the site itself. Publishing a scraped rating risks a manual action, which would work directly against the ranking goal.
- **No mailing list form.** It needed a backend and it was not serving the page's job.
- **Meetings and events** is reduced to one line in the practical details. If it turns out to be a real revenue line, that is the first thing to promote to its own section.

---

## SEO notes

Present: `Hotel` JSON-LD with address, geo coordinates (29.699891, -95.378284), telephone, check-in and check-out times, amenity list, `ReserveAction`, and `sameAs` pointing at the Choice listing. Canonical, Open Graph, Twitter card, sitemap, robots.

Still to do once the domain is live:

1. Update the four hardcoded `https://www.mainstayhouston.com/` URLs in `index.html` (canonical, three OG tags) and the one in `sitemap.xml` if the domain differs.
2. Claim and fill the Google Business Profile. For a single-property hotel, that profile outranks the website for most branded searches. It matters more than anything on this page.
3. The `vercel.json` redirects map the old ProcessWire URLs to `/` so existing inbound links and any accumulated authority survive the cutover. Check your analytics for other old paths worth adding.
4. This page will not outrank Booking.com or Expedia on generic queries like "hotels near Texas Medical Center". It can and should own branded queries: "mainstay suites houston", "mainstay old spanish trail", "mainstay medical center houston". The title tag and H1 are written for that.

---

## Accessibility

Skip link, visible focus rings, labelled slider with live `aria-valuetext`, `role="status"` on the copy that changes, real alt text on every image describing what is actually in the frame, `prefers-reduced-motion` respected throughout.

The scroll reveal is a progressive enhancement. Content is only hidden after JavaScript confirms `IntersectionObserver` exists, anything scrolled past gets revealed, and a 2.5 second failsafe reveals everything regardless. There is no path where the page renders blank.

---

## If you pick this up in a new chat

Start from `index.html`. It is self-contained: styles in one `<style>` block, behaviour in one `<script>` block, no imports beyond Google Fonts. The booking config block is the first thing in the script and is commented. Design tokens are the first thing in the stylesheet.

Most likely next tasks, in the order they would pay off:

1. Verify the Choice deep link parameters actually prefill dates. Thirty seconds: open the page, drag the slider to 12 nights, click the button, see whether Choice shows your dates.
2. Confirm the shuttle hours and breakfast, fix the copy.
3. New photography.
4. Google Business Profile.
5. Decide whether Meetings and Events deserves a section.
