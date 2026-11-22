[Uploading README.md…]()
# Property Matchmaker

A simple, mobile-first real estate matchmaker. Plain HTML, CSS and JavaScript, no login, data stays on the device.

## How it works

**Find** asks the client questions: name (optional), phone/WhatsApp (optional), to rent or to buy, property type, preferred area(s), state/region, country, budget, currency, and additional requirements. It saves the request and ranks the saved properties.

**Import** reads pasted messages, TXT, CSV, PDF, Word (.docx) and screenshots (JPG, PNG, WEBP, BMP, GIF, via OCR). Modes: mixed messages (auto-sorted), requests only, listings only. For every message it works out:

- whether it is a listing, a client request or just chat (chat, greetings and "media omitted" lines are skipped)
- category (buy or rent) from explicit wording only; a message that says both is left for you to choose
- property type, area(s), state, country, price or budget (₦, N, #, $, £, €, "45m", "1.5 million", ranges, per year/month/week/day), currency, bedrooms and extra requirements
- contact name and phone, from the message or the WhatsApp sender
- the **date last available**: the WhatsApp message date, Facebook post time ("3d", "yesterday"), a date written in the text, or the screenshot's file date, falling back to the import date

It splits WhatsApp exports (Android and iPhone formats), Facebook pastes, numbered lists and multi-ad messages. Everything is shown for review, with missing fields flagged, before saving. Listings older than 30 days are marked as possibly outdated, and "Available today" on a saved property refreshes its date.

**Saved** lists clients and properties, shows the clients matching a property, and exports/restores a JSON backup.

## Matching rules

A property matches a client only when **every** requirement passes. There are no partial scores.

1. **Category** (strict gate). BUY = to buy / for sale. RENT = to rent / for rent / to let / for let. A buy request never meets a rental listing, and the reverse. A message that mentions both is not assigned until you choose one.
2. **Property type** must be the same type (House is not a type; use Duplex, Bungalow, Flat, etc.). If the listing does not say its type, it does not match.
3. **Location**: the country must be the same, one of the wanted areas must match (spelling variants such as Ungwan/Unguwan are accepted), and the state must match. Known areas fill in a missing state (for example Barnawa is in Kaduna). If the state cannot be confirmed, the listing does not match.
4. **Budget**: the price must be at or under the budget, in the same currency. No currency conversion is done. Rent is compared per year, and monthly prices are multiplied by 12.
5. **Additional requirements**: every requirement the app can check must be found in the listing: bedrooms, bathrooms, toilets, parking, furnishing, land size, and features such as borehole, gated estate, security, C of O. Other free words are shown as "confirm with the agent" and do not block a match.

Properties that fail are listed under "did not match" with the exact reason. Use Edit on a saved property to fix missing details such as state or price.

## Run / deploy

Serve this folder from any static HTTPS host (GitHub Pages works). Keep index.html, manifest.webmanifest, sw.js and the two icons together. Data from the earlier Property Desk app in the same browser is carried over once on first load.

`worker.js`, `wrangler.toml`, `schema.sql` and `CLOUD-SETUP.md` belong to the old cloud-sync version and are not used by this app.

## Import notes

- **Dates.** Every imported item records when it was last available: the WhatsApp or Facebook post date first, then a date written in the text or a CSV date column, then the file date, then the import date. Dates are shown as "Last available as at ..." and can be edited before saving.
- **WhatsApp exports** (Android and iPhone), **Facebook pastes** (name, time, post), **numbered multi-listing messages**, **labelled messages** ("Price: 45m") and **CSV files** (columns read by heading) are each split into separate listings and requests. Chat notices, media and greetings are skipped.
- House is not a property type. A message that only says "house" gets no type and must be completed before it can match.

## Block of flats / apartments

A group of flats is its own property type. "Block of 6 flats", "8 units of 2 bedroom flats", "10 apartments" and "apartment complex" are read as a block, never as a single flat. In a block request, the additional requirements are checked per block: number of flats ("at least 6 flats"), bedrooms ("2 bedrooms each" means every flat; without "each" the largest flat counts), plus features such as separate/prepaid meters, fully tenanted / rental income, borehole and C of O. Existing saved listings that describe a block are moved to this type automatically.

## Supabase (saved data, no login yet)

Everything you import or enter is saved to Supabase (project "match Maker", tables `pm_requests`, `pm_listings`, `pm_matches`) and cached on the device, so the app still works offline and sends waiting changes when the connection returns.

- **No login.** Each browser gets a private workspace code (Settings). The tables are closed to direct access; the app reads and writes only through the functions `pm_load`, `pm_save`, `pm_delete` and `pm_wipe`, which need that code. Anyone who has the code can see and change the data, so keep it private. To use the same data on another device, enter the code under Settings.
- **Sections.** Find (enter a request), Requests (view, edit, rematch, delete), Listings (view, edit, confirm still available, delete, remove old ones), Matches (saved results with status, notes, WhatsApp, rematch, dismiss) and Import.
- **Rematch.** Matches are saved. They are rechecked automatically when you save a request, import properties or edit a property, and on demand with Rematch. A match that no longer fits is removed, or marked "No longer matches" if you had already contacted the client.
- **Adding a login later.** Add a user column and row-level policies in place of the workspace code; the app's data layer is the only part that talks to Supabase.

## Close alternatives (second layer)

After the exact matches, each request also shows up to 10 close alternatives. These are not exact matches. They stay in the same category (a buy request never sees a rental) and the same country, and must be in the same area or the same state. They are ranked out of 100 in this order of priority:

| Priority | Points |
|---|---|
| Location: area 30, state 10, country 5 | 45 |
| Property type 14, bedrooms and other specs (bedrooms, flats in a block, bathrooms, size) 10, features 11 | 35 |
| Budget (within budget 20, then 14, 8 and 3 as it goes over) | 20 |

Each one is written as a sentence, for example "2-bedroom flat available in Barnawa, Kaduna, for ₦2,300,000 a year (your budget: ₦2,000,000 a year), with 2+ bedrooms, borehole." Only the additional requirements it actually meets are listed. "Where it falls short" lists what differs, and "How it ranked" shows the points. Alternatives are worked out live from the current listings and are not stored.

### Saved close alternatives and reach

When a request is saved, a property is added or edited, or you press Rematch, each client's top close alternatives (5 by default) are saved in Matches under "Close alternatives", with a status (New, Contacted, Closed), a note and a Dismiss button, like exact matches. A listing moves between "match" and "close alternative" automatically as it changes, and keeps its status and note. Under Settings you can choose the reach (same area or state, or anywhere in the same country), the lowest closeness to show (30 to 60%) and how many to save per client (3, 5 or 10). Buy and rent are never mixed at any setting. Saved alternatives are stored in `pm_matches` (`is_alt`, `score`, `sentence`, `diffs`).

## Photos and videos

Any request or listing can have pictures and short videos ("Photos & videos" on its card). Pictures are resized before upload. Videos can be MP4, WebM or MOV, up to 60 seconds and 25 MB. Files are stored in the Supabase bucket `pm-media` under `<workspace code>/<id>.<ext>`, and the `pm-media` edge function signs each upload and deletes files. The bucket cannot be listed and is not linked to any login, so a file's address is known only to someone with the workspace code. Deleting a picture, request or listing deletes its files.

## Search

The magnifier in the header searches everything saved (text, category, type, state or area, price or budget range, last available date, has photos or videos). Below the results, "Search the internet" opens Google, Facebook posts and groups, Instagram, X, TikTok, Telegram, PropertyPro, Nigeria Property Centre, Jiji and PrivateProperty for any text or any client's request. The sites' results open in a new tab; the app cannot read them back.

## Sources, posting and replying

The Sources tab keeps Facebook groups, pages and profiles, WhatsApp numbers and group links, Instagram, Telegram, X, TikTok, YouTube, LinkedIn and real estate websites. Paste a link or number and the platform is detected. Launch opens it. Post and Reply write the message (or use your own), then open the platform: WhatsApp numbers, Telegram and X open with the text filled in; WhatsApp groups, Facebook, Instagram and websites open with the text copied for you to paste. Share sends the text and the pictures and videos through your phone's share sheet. Platforms do not allow an app to post for you on Facebook, Instagram or into WhatsApp groups, so the last tap is always yours.
