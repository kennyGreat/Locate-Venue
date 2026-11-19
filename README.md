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
