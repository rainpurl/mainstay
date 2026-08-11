# MainStay Suites Houston: HANDOFF

**Version:** v8
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
index.html      the entire site, single file, no build step
vercel.json     caching, security headers, 301s from the old URL structure
robots.txt      explicit allow list for search, AI retrieval and training
llms.txt        plain-text summary for language models
sitemap.xml     one URL
img/            property photos as WebP with JPEG fallbacks, logo assets,
                the reel poster, and og.jpg
vid/            the three splash reel encodes
```

Deploy by dragging the folder into Vercel, or push to a repo and import it. No framework, no dependencies, no build command. Output directory is the folder itself.

**Note on Root Directory.** If you unzip and commit the resulting folder, your repo will have a wrapper directory and Vercel will 404 on `/`. Either commit the folder's *contents* at the repo root, or set Settings, Build and Deployment, Root Directory to the wrapper folder name. Framework Preset should be Other with Build Command and Output Directory left empty.

**Bandwidth.** The reel is served from Vercel's CDN and counts against your bandwidth allowance. At 335 KB for the AV1 path, the Hobby tier's 100 GB would take roughly 300,000 splash views to exhaust, so this is not a practical concern at current settings. It would be if anyone swaps in a multi-megabyte clip.

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

The page opens on a full-viewport splash: a silent 8 second video reel behind a navy scrim, the logo, one line of description, and a single Book now button. Nothing else competes with it.

### The reel

Source was a 22.9 MB, 10 second, 1920x1080 H.264 file at 18 Mbps with an AAC track. Three problems were fixed on the way in:

1. **A 222 px black bar was baked into the right edge of every frame.** Real content was 1698x1080 at x=0, an export fault. Cropped off. Full-bleed it would have shown.
2. **One scene showed pre-renovation decor.** The bed at 2 to 4 seconds had green patterned carpet and a floral quilt, while the same room type in the Suites section below has wood floors and gold bedding. That scene is cut.
3. **Scene order started on the lobby**, whose wall signage sits right beside the overlaid logo. Order is rotated to lounge, pool, suite, lobby.

Result: 8 seconds, 240 frames, 1280x814, no audio track. Transitions are hard cuts in the original, so looping back to frame 0 reads as one more cut and needs no crossfade.

Shipped encodes:

| File | Codec | Size |
|---|---|---|
| `vid/reel.vp9.webm` | VP9 | 496 KB |
| `vid/reel.h264.mp4` | H.264 | 650 KB |
| `img/reel-poster-1400.jpg` | poster | 102 KB |

**AV1 was encoded and rejected.** At matched quality it came out at 507 KB, slightly *larger* than VP9's 496 KB. The content is static photographs with a slow zoom, so there is almost no interframe motion for AV1's tooling to exploit, and its Safari support is hardware-gated anyway. Two formats instead of three, for no size penalty. If a future reel has real camera motion, AV1 is worth retesting.

Shipped quality is PSNR 39.5 dB against a near-lossless master, mean per-pixel error under 2 of 255. Invisible under a scrim running 80 to 99 percent opacity.

### How the reel loads

The `<source>` elements are injected by JavaScript, not written into the markup. That means the download never starts for anyone who asked not to have it: `prefers-reduced-motion`, `Save-Data`, or a 2G-class `effectiveType`. Those visitors keep the poster frame, verified at zero video bytes requested. Without JavaScript the poster also stands in.

Playback pauses when the tab is hidden and when the splash scrolls out of view, so it is not decoding while someone reads the Details section.

To swap the clip: replace the two files in `vid/`, regenerate the poster from the first frame into `img/reel-poster-1400.jpg`, and nothing else needs touching. `vercel.json` sets immutable year-long caching plus `Accept-Ranges` on `/vid/`, so filenames should change when content does.

Below the splash: Suites, Near the Texas Medical Center, The property, Details, Book direct, footer. Each Details entry is two lines, a label and a value, with any secondary detail folded into the value line. No section navigation, no menu, no internal links other than the scroll cue. Every call to action goes to the same place.

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

## Splash reel

The splash plays an 8 second silent loop of the property behind the navy scrim, with a poster still underneath.

### Files

```
vid/reel.av1.mp4    335 KB   AV1, Main profile, level 3.1, 8-bit
vid/reel.vp9.webm   446 KB   VP9
vid/reel.h264.mp4   653 KB   H.264 High
img/reel-poster-1280.jpg     poster, frame 0 of the loop
img/reel-poster-900.jpg      poster for narrow viewports
```

All three are 1280x816, 30 fps, no audio track. The browser takes the first `<source>` it can decode, so most visitors get the 335 KB AV1 and only that file is requested. Verified: desktop and tablet download `reel.av1.mp4` and nothing else.

### How they were made

Source is `Video_Project_2.mp4`, 1920x1080, 10 seconds, 30 fps, with an AAC track and a black pillarbox down the right edge.

The 8 second loop is not a straight trim. It starts at t=4.0s, runs to the end, and wraps back through t=0 to t=2.0. This was recovered by matching frames rather than guessed: six spot-checks across the loop align with the source at MAE around 2, which is just compression noise.

```bash
FC="[0:v]trim=start=4:end=10,setpts=PTS-STARTPTS[a];\
[0:v]trim=start=0:end=2,setpts=PTS-STARTPTS[b];\
[a][b]concat=n=2:v=1[c];\
[c]crop=1694:1080:2:0,scale=1280:816:flags=lanczos,fps=30[v]"

# AV1
ffmpeg -i src.mp4 -filter_complex "$FC" -map "[v]" -an -sn \
  -c:v libsvtav1 -crf 46 -preset 6 -g 240 -pix_fmt yuv420p \
  -movflags +faststart vid/reel.av1.mp4

# VP9
ffmpeg -i src.mp4 -filter_complex "$FC" -map "[v]" -an -sn \
  -c:v libvpx-vp9 -crf 40 -b:v 0 -row-mt 1 -deadline good -cpu-used 2 \
  -g 240 -pix_fmt yuv420p vid/reel.vp9.webm

# H.264
ffmpeg -i src.mp4 -filter_complex "$FC" -map "[v]" -an -sn \
  -c:v libx264 -crf 28 -preset slow -profile:v high -pix_fmt yuv420p \
  -g 240 -movflags +faststart vid/reel.h264.mp4
```

The crop is `1694:1080:2:0` rather than the `1698:1080:0:0` that cropdetect reports. That trims 4 more pixels of width so both output dimensions land on a multiple of 8, which encoders prefer, and it makes the scale to 1280x816 geometrically exact instead of stretching by 0.2 percent.

Two things worth knowing if you re-encode:

- **Encode from the 1080p original, not from an existing MP4.** Going from the H.264 produced an AV1 file *larger* than the VP9, because AV1 was spending bits reproducing H.264's artifacts. Generation-1 from the source at CRF 46 came out at 335 KB instead.
- **CRF 46 is aggressive on purpose.** The splash scrim sits at 80 to 99 percent opacity navy, so fine detail is destroyed before anyone sees it. Compared side by side at 100 percent against a CRF 24 reference, the fence railings, the hardest detail in the frame, are indistinguishable.

### Who gets the video

Sources are injected by script, never written into the markup, so visitors who should not download 335 KB of video make zero requests for it. The poster still is used instead when any of these is true:

- `prefers-reduced-motion: reduce`
- `navigator.connection.saveData`
- effective connection type 2g or slow-2g
- viewport narrower than `REEL_MIN_WIDTH`, currently 640px

The poster is set by script rather than in the markup, and the preload links are `media` gated, so a phone fetches only the 900px poster (66 KB) and a desktop only the 1280px one. On a phone the poster is the permanent hero, which is why it gets its own size rather than a downscaled 1280.

That last one is a framing decision more than a bandwidth one. The reel is landscape at 1.57:1. A phone in portrait is about 0.46:1, so `object-fit: cover` shows only **29 percent of the frame width**. The scene becomes an unreadable vertical strip. Desktop at 1440x900 shows 100 percent of it. If you ever shoot a vertical cut, add it as a second source set and drop `REEL_MIN_WIDTH` to 0.

The video also pauses when the tab is hidden and when the splash scrolls out of view, so it is not decoding while someone reads the rest of the page.

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

## Discoverability: search and assistants

### What is on the page

- One H1, sentence-case H2 tree, title and description both naming the Texas Medical Center and MD Anderson
- `Hotel` JSON-LD: address, geo coordinates, phone, check-in and out, amenities, `sameAs` to the Choice listing
- `FAQPage` JSON-LD generated from the Questions section, so both stay in sync
- Canonical, Open Graph, Twitter card, sitemap
- 301s from the old ProcessWire URLs
- `llms.txt` at the root, linked from the head

### The Questions section

Ten plain question-and-answer pairs in `<details>` elements. Collapsed by default, so the page stays visually calm, but every answer is in the HTML on load: 349 characters of visible text against 2,485 characters of markup. Google indexes accordion content normally, and retrieval crawlers read the DOM, so nothing is hidden from them.

This section exists because of a real trade-off. Cutting the page to 384 words made it clean but gave retrieval systems very little to match. Someone asking an assistant for an extended stay near MD Anderson with a kitchen and a shuttle needs those facts stated plainly somewhere. The page is now 676 words, and almost all of the addition is inside collapsed answers.

**The FAQ answers and the JSON-LD are generated from the same source.** If you edit a question or answer in the markup, regenerate the `FAQPage` block or they will drift apart. Note also that `FAQPage` no longer produces rich results in Google for most sites, since Google restricted them to government and health authorities in 2023. It is here for machine parsing, not for a snippet.

### robots.txt

34 explicit user-agent groups, no `Disallow` anywhere. Grouped by what the crawler actually does, because the categories matter differently:

- **Search index crawlers** (`OAI-SearchBot`, `Claude-SearchBot`, `PerplexityBot`, `Amazonbot`) build the indexes assistant answers are assembled from. Blocking these makes the hotel invisible to chatbots. These are the ones that matter most.
- **User-triggered fetchers** (`ChatGPT-User`, `Claude-User`, `Perplexity-User`) fire when a live person asks something. Several providers say these may not strictly honor robots.txt because a human initiated the request.
- **Training crawlers** (`GPTBot`, `ClaudeBot`, `CCBot`, `Meta-ExternalAgent`) shape what future models know. No immediate citations.
- **Data-use control tokens** (`Google-Extended`, `Applebot-Extended`) are not crawlers at all. They control whether Google and Apple may use content their ordinary crawlers already fetched.

Two things to keep in mind:

1. **The wildcard group already allowed all of these.** The named groups are documentation, not new access. Their value is that a future edit cannot silently cut off AI retrieval without someone noticing.
2. **robots.txt is most-specific-group-wins.** If anyone adds a `Disallow` to the `User-agent: *` group later, the 33 named groups will not inherit it. They have to be updated too. There is a comment at the top of the file saying so.

One trap worth knowing: you cannot block AI Overviews without blocking Google Search, because both run through `Googlebot`. `Google-Extended` only opts out of Gemini and Vertex training.

**Check the Vercel firewall.** CDN and WAF bot management can block AI crawlers regardless of what robots.txt says. If Vercel's bot protection is on, verify these agents are not being challenged, or the file is decorative. Test with `curl -A "OAI-SearchBot" -I https://yourdomain/` and expect a 200 with no `X-Robots-Tag: noindex`.

### llms.txt

A plain-text summary at `/llms.txt` following the llmstxt.org convention: address, distances, shuttle hours, kitchen contents, rates, the booking URL, the front desk number.

Be clear-eyed about this one. It is a community proposal, not a standard, and no major provider has committed to reading it. It costs about 1 KB and cannot hurt. Do not count it as a measure that does anything yet.

### What none of this achieves

There is still no mechanism that guarantees a top position, and the two biggest levers remain off the site entirely:

1. **Google Business Profile.** Queries like "hotels near Texas Medical Center" return Google's hotel unit, which is built from Business Profile and Hotel Center, not from HTML. No amount of work on this file enters that unit.
2. **Get listed where this audience already looks.** MD Anderson publishes a lodging PDF and directs patients to **joeshouse.org**, which has a dedicated MD Anderson Houston page. Those pages are exactly what an assistant retrieves when asked this question, and they reach every patient planning a trip. There is also MD Anderson's iDeal program for staff and vendor discounts. Getting on those lists is worth more than everything in this zip.

### The medical rate gap

MD Anderson tells patients to call hotels near the Texas Medical Center and ask for a medical or patient rate. Competing hotels nearby publish theirs; one at 1.5 miles lists patient rates from $99 including parking. This property's shuttle runs 7:00 am to 7:00 pm against that competitor's 8:00 to 5:00, and this property has full kitchens.

The Questions section now tells guests to call and ask about a medical rate, which is honest without asserting one exists. **Confirm with the front desk whether there is one and what it is.** If there is, state the figure. It is the phrase this audience literally searches for.

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

### What the page does not do

An earlier version of this build carried a "Why the resellers look cheaper" section explaining the pre-tax display, with a total-cost estimator and a note about the 30-night tax exemption. It was removed at your request in v5, along with its stylesheet block and the `TAX` constant.

If you ever want it back, the two facts it rested on are worth keeping somewhere, because they are the strongest honest arguments this property has:

- Houston hotel occupancy tax is 17 percent (6 state, 7 city, 2 Harris County, 2 Harris County / Houston Sports Authority). A $79 room is about $92 all in, on any site.
- Texas exempts stays of 30 or more consecutive days from hotel occupancy tax. On a month-long medical stay that is worth more than any rate difference a reseller can advertise, and no comparison row will ever show it.

The booking block still points guests at the front desk for stays over 30 nights, which is where that conversation should happen anyway.

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
