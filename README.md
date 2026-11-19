# Property Matchmaker

A simple, mobile-first real estate matchmaker. Plain HTML, CSS and JavaScript, no login, data stays on the device.

## How it works

**Find** asks the client questions: name (optional), phone/WhatsApp (optional), to rent or to buy, property type, preferred area(s), state/region, country, budget, currency, and additional requirements. It saves the request and ranks the saved properties.

**Import** adds listings and requests from pasted messages, TXT, CSV, PDF, Word (.docx) and screenshots (JPG, PNG, WEBP, BMP, GIF, read with OCR). Modes: mixed messages (auto-sorted), requests only, listings only.

**Saved** lists clients and properties, shows the clients matching a property, and exports/restores a JSON backup.

## Matching rules

Transaction category is a strict gate, not a score. Two categories only:

- BUY: To buy / For sale
- RENT: To rent / For rent / To let / For let

A BUY request never matches a RENT listing and vice versa. A message that mentions both is not assigned to either; you choose one before saving.

Eligible properties score out of 100, 25 points each:

| Criterion | Points |
|---|---|
| Property type | 25 (same type 25, similar type such as duplex/house 12.5) |
| Preferred location | 25 (area 17.5, state 7.5; different country 0) |
| Budget | 25 (within budget 25, up to 5% over 20, 10% over 15, 20% over 7.5) |
| Additional requirements | 25 (share of detected requirements the listing meets) |

Rent budgets and prices are compared per year (monthly prices are multiplied by 12). Different currencies are not compared. Matching is rule-based text matching, not AI; check imported details before contacting anyone.

## Run / deploy

Serve this folder from any static HTTPS host (GitHub Pages works). Keep index.html, manifest.webmanifest, sw.js and the two icons together. Data from the earlier Property Desk app in the same browser is carried over once on first load.

`worker.js`, `wrangler.toml`, `schema.sql` and `CLOUD-SETUP.md` belong to the old cloud-sync version and are not used by this app.
