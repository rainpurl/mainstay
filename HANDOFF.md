# MainStay Suites Houston: HANDOFF

**Version:** v4
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

Both accent colors were sampled directly out of the official MainStay Suites logo PNG, so they are the real brand values, not approximations.

| Token | Hex | Use |
|---|---|---|
| `--navy` | `#003046` | Brand navy. Dark sections, headings, body text on light. |
| `--seafoam` | `#A1D7BE` | Brand mint. Buttons, figures, the logo mark. |
| `--seafoam-deep` | `#2F7A5E` | Darkened seafoam, used only for focus rings. |
| `--paper` | `#EDF1F0` | Page background, a cool neutral derived from the seafoam. |
| `--paper-hi` | `#F7F9F8` | Alternating section background. |
| `--ink` | `#08202B` | Body text and labels. |
| `--slate` | `#4A6270` | Secondary text. |

**Type: Poppins only.** Weights 300, 400, 500, 600. One family, one request, no display face, no mono.

Choice Hotels does not publish a brand type spec, so the face was identified from the logo artwork itself. The wordmark has a splayed `M` whose vertex reaches the baseline, a single-storey circular `a`, a round dot on the `i`, a flat-topped `t` and a straight diagonal `y`. Poppins matches all of those. Jost and Questrial were tested and rejected: Jost is too narrow and its `M` vertex stops short of the baseline. Corroborating evidence: the old mainstayhouston.com already loaded Poppins from Google Fonts, so the previous agency reached the same conclusion.

If you get hold of the real MainStay brand kit and the licensed face is different (Century Gothic and Avant Garde are the other plausible candidates, and the old site's component CSS did declare Century Gothic), swapping is one line: the Google Fonts `<link>` in the head, plus the `font-family` on `body`.

**The wordmark is always the logo file, never type.** `img/logo-navy.png` in the header, `img/logo-reverse.png` on the splash and in the footer. The reverse lockup was generated from the official artwork by recoloring the navy type to white while leaving the seafoam mark untouched. Nowhere does the page set "MainStay Suites" in a typeface.

## Structure

The page opens on a full-viewport splash: a crossfading slideshow of five property photos behind a navy scrim, the logo, one line of description, and a single Book now button. Nothing else competes with it.

Slide order is king suite, two-queen suite, suite kitchenette, breakfast area, lobby. The lobby is deliberately not first because its wall signage sits right next to the overlaid logo. Slides change every 5.2 seconds, pause when the browser tab is hidden, pause when the splash scrolls out of view, and do not run at all under `prefers-reduced-motion`. The first slide is preloaded and marked `fetchpriority="high"`; the other four are lazy.

To change the slideshow, edit the five `<picture class="slide">` blocks in the splash and the matching preload `<link>` in the head. Any image in `img/` with 900, 1400 and 2000 wide variants can drop in.

The header is fixed but starts translated off-screen. An IntersectionObserver on the splash slides it in once the splash scrolls out of view, and slides it back out when you return to the top. The mobile booking dock follows the same trigger. If IntersectionObserver is missing the header just stays visible, so there is no state where the page loses its navigation.

Below the splash: Suites, The property, Getting around, Details, Book direct, footer. No section navigation, no menu, no internal links other than the scroll cue. Every call to action goes to the same place.

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

The link uses the `en-us` path rather than the `en-ca` one you sent. Choice geo-redirects anyway, and the guest base here is US medical-center traffic.

The JSON-LD `sameAs` and `ReserveAction` still reference the canonical property page without `/rates`, which is correct for structured data.

---

## Photography

Every photo is a real photograph of this property. No stock, no staged interiors from other hotels.

**All imagery was re-exported in v4 from true originals.** The earlier versions pulled some files from resized derivatives on the old server, and one of those, the pool shot used on the splash, was a 365 KB re-encode of a 2000x1333 frame. That is where the grain came from. The unsuffixed originals turned out to be 2400x1599 at 1.1 to 2.0 MB each, so everything was rebuilt from those at quality 86 with lighter sharpening.

Source paths for the originals, if you ever need them again:

```
https://www.mainstayhouston.com/site/assets/files/17994/<name>.jpg
https://www.mainstayhouston.com/site/assets/files/17245/<name>.jpg
https://www.mainstayhouston.com/site/assets/files/17453/<name>.jpg
https://www.mainstayhouston.com/site/assets/files/14896/tx933pool2_1.jpg
```

Note there is no size suffix. Adding one (`.2000x0.jpg`, `.600x450.jpg`) gets you the degraded copy.

**One caveat that has not changed.** These images are dated. They read as early-2010s interiors, and several Google reviews mention rooms feeling outdated. New photography would do more for conversion than anything else on this page. The layout takes drop-in replacements at the same filenames and aspect ratios.

## Copy

The page carries about 380 words of visible text, down from roughly 900. It was written against the Wikipedia:Signs of AI writing field guide. Specifically removed:

- Rule-of-three constructions. The old headline was "A kitchen, a couch, and a short ride to the Medical Center."
- Negative parallelism. "Book on Choice Hotels, not the resellers" became "Book direct".
- The bold inline-header vertical list with 01/02/03 markers in the booking section, which is one of the more recognisable tells. It is now two plain sentences.
- Present-participle analysis tacked onto sentence ends ("so you can pack lighter than you think").
- Invented claims presented as fact: "The free shuttle is the reason most people book here", "The one families book when several people are taking turns at the hospital". Neither was sourced.
- Punched-up rhetorical framing: "Three weeks of hospital cafeteria food is a long three weeks."
- All uppercase letter-spaced eyebrow labels. There are now zero `text-transform:uppercase` rules in the stylesheet.
- Em dashes. There are none in the file.

Headings are plain nouns in sentence case: Suites, The property, Getting around, Details, Book direct. The copy uses `is` and `has` rather than `serves as` and `features`, which the field guide notes as a human signal.

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

## Search ranking and the price shown on Google

This section exists because it was asked for directly, and because most of it cannot be done in code. Worth reading before anyone spends money on it.

### Nothing on this page can make it rank first

There is no markup, meta tag, or script that produces a top ranking. Anyone who tells you otherwise is selling something. What the page can do is not get in its own way, and it now does that: one H1, accurate title and description targeting the real queries, valid `Hotel` structured data, fast images, mobile layout, clean semantics, 301s from the old URLs.

The harder problem is that the queries you named mostly do not return a list of websites at all. "Hotels near Texas Medical Center", "extended stay Houston medical center" and similar searches return Google's hotel unit: a map, a set of properties, and prices. That unit is not built from website HTML. It is built from Google Business Profile and Hotel Center. **Optimizing this page cannot put you in it.**

So the ranking work that actually matters is not on this site:

1. **Google Business Profile.** Claim it, verify it, set the category to Extended Stay Hotel, load the new photography when it exists, set the website link to this domain, and keep the hours and phone identical to what this page says. For a single property this outranks the website for most searches that matter.
2. **Reviews.** The property sits at 3.8 from about 900 Google reviews. Rating and review volume are ranking inputs in the local pack. A steady flow of recent reviews will move the needle more than any change to this file.
3. **NAP consistency.** Name, address and phone must match exactly across Google, Choice's listing, Yelp, Apple Maps and this site. Right now this page and the Choice listing agree, which is the important pair.

### Why the resellers show a lower price, and what can be done

Two separate things are going on, and neither is fixable from your website.

**Google displays base rates before tax to US users.** This is Google's own documented behavior: for travelers in the United States and Canada it highlights the base rate without taxes and fees, while everywhere else it highlights the total. So every listing in that row, including the direct one, is showing a pre-tax number. Houston hotel tax is 17 percent. A $79 room is about $92 once tax is on it, whoever sold it.

**Your rate does not reach Google from this website.** Google takes hotel prices through Hotel Center, fed by a connectivity partner. For a Choice franchise that is Choice's central distribution, not your marketing site. The "Official site" row in that price comparison is Choice's feed. You cannot add, undercut, or edit it from here. If the direct rate is showing higher than an OTA, that is a rate and distribution question for Choice, not a web development question.

Real levers, roughly in order of effect:

1. **Set the rates.** The property controls its own rates in Choice's system. Member rates are usually exempt from OTA rate parity clauses, which is why the Choice Privileges rate can legitimately sit below the public OTA rate. This is the actual price lever.
2. **Choice's lowest price guarantee.** Choice publishes one covering choicehotels.com, its call center and hotel-direct bookings against third-party sites. It is now cited on the page. It is the honest answer to "they showed me cheaper".
3. **Google's Price Accuracy Policy.** Free booking link ranking depends partly on historical price accuracy. If an OTA advertises a rate that is not what the guest ends up paying, that is reportable and it affects their placement.
4. **The FTC Junk Fees Rule.** Effective 12 May 2025, 16 CFR Part 464 makes it an unfair and deceptive practice to advertise short-term lodging without clearly disclosing the total price including all mandatory fees. Civil penalties run to about $51,744 per violation. Note the limit: the rule covers **mandatory fees**, not taxes. An OTA showing a pre-tax base rate is complying. An OTA hiding a mandatory fee until checkout is not. Texas has already settled with Booking Holdings over deceptive pricing, so the state has form here.

### What the page does instead

Since the price display cannot be won, the page explains it. The "Why the resellers look cheaper" section states plainly that US price displays are pre-tax, gives the 17 percent figure, and includes a small estimator so a guest can see the real total before they leave. It also surfaces the strongest genuine advantage this property has: **Texas exempts stays of 30 or more consecutive days from hotel occupancy tax.** On a month-long medical stay that is worth more than any rate difference an OTA can advertise, and no reseller comparison row will ever show it.

The tax rate is a single constant at the top of the script:

```js
var TAX = 0.17;   // 6% state + 7% Houston + 2% Harris County + 2% Sports Authority
```

Confirm it against the Texas Comptroller before launch and edit there if it has moved. The 30-night exemption should also be confirmed with your accountant for how it is applied at this property, since the refund-versus-waiver mechanics vary.

## Accessibility

Skip link, visible focus rings, labelled slider with live `aria-valuetext`, `role="status"` on the copy that changes, real alt text on every image describing what is actually in the frame, `prefers-reduced-motion` respected throughout.

The scroll reveal is a progressive enhancement. Content is only hidden after JavaScript confirms `IntersectionObserver` exists, anything scrolled past gets revealed, and a 2.5 second failsafe reveals everything regardless. There is no path where the page renders blank.

---

## If you pick this up in a new chat

Start from `index.html`. It is self-contained: styles in one `<style>` block, behaviour in one `<script>` block, no imports beyond Google Fonts. The booking config block is the first thing in the script and is commented. Design tokens are the first thing in the stylesheet.

Most likely next tasks, in the order they would pay off:

1. Verify the Choice deep link parameters actually prefill dates. Thirty seconds: open the page, set check-in and check-out, click See rates, see whether Choice shows your dates.
2. Confirm the shuttle hours and breakfast, fix the copy.
3. New photography.
4. Google Business Profile. See the ranking section above; this is the highest-leverage item on the whole list.
5. Decide whether Meetings and Events deserves more than the one line it has in Details.
