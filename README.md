# 3D-Agencies-new-site
an upgraded site for 3D
# Three-D Agencies — B2B Industrial Portal & Technical Sales Agent

A self-contained, single-file B2B industrial website for **Three-D Agencies**, a premier South African distributor of cable accessories, tools, and electrical equipment (est. 1977). Built with Tailwind CSS (CDN) and vanilla JavaScript — no build step, no backend, no paid APIs.

---

## Quick Start

1. Save the HTML file as `index.html`.
2. Open it directly in any modern browser (Chrome, Edge, Firefox, Safari).
3. That's it — everything runs client-side.

> **Note:** An internet connection is required on first load to fetch the Tailwind CDN and the Inter font from Google Fonts. For fully offline use, see [Offline Deployment](#offline-deployment).

---

## Features

### 1. Visual System & Layout
- **Sticky dark-slate header** with logo, account access, RFQ cart badge, and a prominent predictive search bar.
- **Predictive search** indexes SKUs (`HP240`), dimensions (`120mm²`), brands (`UNI-T`, `Makita`, `SapiSelco`), tags, and aliases. Clicking a result adds the item to the cart.
- **Multi-column mega-menu** on hover covering: Lugs & Terminals, Cable Management, Cable Protection, Hand/Hydraulic Tools, Test Instruments, and Power Tools.
- **Split hero** with SADC export coverage (Lesotho, Botswana, Mozambique, Namibia, Zambia) and two CTAs: *Request Trade Account* and *Quick BOM / SKU Order*.
- **Product catalog** as a responsive card grid with:
  - Stock status badges ("In Stock — JHB Warehouse" vs "Export Order")
  - Technical tags (Polyolefin, Dual-Wall Waterproof, 1000V Insulated, etc.)
  - Brand labels and prices
  - Direct **Add to Quote** buttons
  - Category filter pills

### 2. Embedded Technical Agent (Slide-in Drawer)

Opens via the **Tech Agent** button in the header, or the **Quick BOM / SKU Order** hero button.

#### A. Bulk SKU / BOM Importer
- Paste raw contractor lists, e.g.: `HP240 x 3, UT33+ x 5, 120mm lug x 50`
- **Parser handles:**
  - Delimiters: commas, newlines, semicolons
  - Quantity formats: `item x 3`, `3 x item`, `item * 3`
  - Matching hierarchy: exact SKU → alias → dimension (`120mm` + `lug`) → fuzzy name/brand
- **Output:**
  - Matched lines with resolved SKU, quantity, and line price
  - Unmatched lines flagged in red
  - Estimated total
  - One-click **Add all matched to RFQ Cart**

#### B. Technical Assistant & Product Spec Matcher
A rule-based Q&A engine (no external API) covering:

| Topic | Example Questions |
|---|---|
| Crimper-to-lug matching | "What crimping tool for 120mm² lugs?" |
| Heatshrink temperatures | "Polyolefin vs PTFE shrink temp" |
| Insulation types | "XLPE vs PVC difference" |
| Multimeter selection | "UT33+ vs UT593" |
| UNI-T warranty | "UNI-T warranty rules" |
| Repairs | "Tool repair turnaround" |
| Returns | "Return policy for unused tools" |
| Order routing | "Where do orders despatch from?" |
| Lug/ferrule sizing | "120mm lug size" |

Includes **quick-ask chips** for one-tap common questions and a **fallback to product spec lookup** when a SKU is mentioned.

### 3. RFQ Cart
- Slide-in drawer with line items, quantities, per-line and total pricing (ZAR).
- Remove individual items.
- Submit button generates a demo RFQ reference.
- Badge in header shows total item count.

### 4. Support & Trade Sections
- SADC export coverage panel with country routes.
- Trade account request form (demo — shows an alert on submit).
- Support cards for tool repairs, UNI-T warranty, and returns.

---

## Project Structure

```
index.html          ← the entire application (single file)
├── <head>
│   ├── Tailwind CDN + config
│   ├── Google Fonts (Inter)
│   └── Custom CSS (drawer transitions, badges, chips)
├── <body>
│   ├── Header (logo, search, cart, agent toggle, mega-menu)
│   ├── Hero
│   ├── Trust bar
│   ├── Product catalog grid + filters
│   ├── SADC export section
│   ├── Trade account form
│   ├── Support cards
│   ├── Footer
│   ├── RFQ cart drawer
│   ├── Agent drawer (BOM + Assistant tabs)
│   └── Overlay
└── <script>
    ├── PRODUCTS dataset
    ├── Cart state & rendering
    ├── Drawer/tab controls
    ├── Predictive search
    ├── Mega-menu hover logic
    ├── BOM parser & matcher
    └── Rule-based assistant (KB + scorer)
```

---

## Data Model

Products live in the `PRODUCTS` array near the top of the `<script>` block:

```js
{
  sku: 'HP240',
  name: 'Hydraulic Crimping Tool HP240',
  brand: 'Three-D',
  cat: 'Tools',
  price: 4850,
  stock: true,
  tags: ['Hydraulic', '6–240mm²', '13 dies'],
  aliases: ['hp240', 'hp 240', 'hydraulic crimper'],
  specs: { range: '6–240mm²', force: '10 ton', dies: '13 pairs' }
}
```

### Adding a Product
1. Append a new object to `PRODUCTS`.
2. Use a unique `sku`.
3. Add sensible `aliases` (lowercase) so the BOM parser and search pick it up.
4. Use one of the existing `cat` values so the category filter works: `Tools`, `Test Instruments`, `Lugs & Terminals`, `Cable Protection`, `Cable Management`.

The catalog grid, search, and BOM matcher all read from this single array — no other changes needed.

### Extending the Assistant
Add entries to the `KB` array:

```js
{
  keys: ['keyword1', 'keyword2'],
  answer: () => `Your response text here.`
}
```

The scorer picks the entry with the most keyword-character hits. Keys are matched as lowercase substrings.

---

## How the BOM Parser Works

For each line:

1. **Split** the raw input on `[\n,;]+`.
2. **Extract quantity** using regex:
   - `^(.*?)\s*[xX*]\s*(\d+)\s*$` → `item x qty`
   - `^(\d+)\s*[xX*]\s*(.*)$` → `qty x item`
3. **Normalize** by lowercasing and stripping whitespace.
4. **Match in order:**
   - Exact SKU match
   - Alias match
   - Dimension regex (`(\d+)\s*mm`) combined with intent keywords (`lug`, `terminal`, `heat`, `shrink`, `ferrule`)
   - Fuzzy name/brand substring
5. **Render** matched and unmatched groups, compute totals, and offer bulk add.

---

## Customization

### Brand Colors
Edit the `tailwind.config` block in `<head>`:

```js
colors: {
  brand: { 500: '#f97316', 600: '#ea580c', /* ... */ },
  slate: { 850: '#1b2431' }
}
```

`brand-500` is the primary CTA color; `brand-600` is the hover state.

### Currency
Prices use `toLocaleString('en-ZA')`. To change currency formatting, search for `toLocaleString` and replace `'en-ZA'` with your locale (e.g., `'en-US'` for USD) and update the `R` prefix.

### Country List
The trade form and SADC section are static HTML — edit the `<option>` list and the country cards directly.

### Contact Details
Update the footer block (email, phone, hours) and the trade form endpoint (currently a demo `alert`).

---

## Offline Deployment

To run without internet access:

1. Download the Tailwind Play CDN script and host it locally, replacing the `<script src="https://cdn.tailwindcss.com">` tag.
2. Download the Inter font files and add a local `@font-face` rule, removing the Google Fonts `<link>`.
3. Optionally, compile Tailwind to a static CSS file for production (recommended — the Play CDN is not intended for production).

---

## Browser Support

- Chrome / Edge (latest)
- Firefox (latest)
- Safari (latest, desktop and iOS)
- Requires JavaScript enabled.

---

## Known Limitations (Demo Build)

- **No backend** — trade account form, RFQ submit, and warranty claims show alerts only.
- **No authentication** — the "Account" link is a placeholder.
- **Static pricing** — prices are indicative; trade pricing is not applied.
- **No persistence** — cart and chat reset on page reload (no `localStorage`).
- **Demo product images** — the catalog uses gradient placeholders instead of real background-removed product photos.
- **Tailwind Play CDN** — fine for prototyping, not recommended for production traffic.

---

## Production Roadmap (Suggested)

1. **Replace the Play CDN** with a compiled Tailwind build.
2. **Add real product imagery** (background-removed PNGs/WebPs) and populate an `image` field per product.
3. **Wire the RFQ form** to an email service or ERP endpoint; add a server-side reference generator.
4. **Add `localStorage`** persistence for the cart.
5. **Back the assistant with a JSON file** loaded via `fetch` so non-developers can edit answers.
6. **Add authentication** for the account portal and trade pricing tiers.
7. **Integrate live stock** from the warehouse system.
8. **Add analytics** for search terms and BOM match failures (to improve aliases).

---

## License

Demo build prepared for Three-D Agencies. No license specified — treat as proprietary until confirmed.

---

## Credits

- **Design & build:** Custom single-file implementation
- **Typography:** [Inter](https://fonts.google.com/specimen/Inter) via Google Fonts
- **CSS framework:** [Tailwind CSS](https://tailwindcss.com) (Play CDN)
- **Icons:** Inline SVG (Feather-style stroke icons)

---

*Established 1977 — Southern Africa's trusted cable accessories & hardware partner.*
