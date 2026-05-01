# Implementation Plan — Theme Sections

## Overview

Two custom Shopify sections to be built for the Horizon theme.

---

## Section 1: `gift-guide.liquid`

### What it is
A full-width hero/banner section called **"The Gift Guide"**. It contains:
1. An internal **top navigation bar** (part of the section, not a separate header)
2. A **full-bleed background image** (line-art illustration) with **text overlay on the bottom-left**
3. A **bottom ticker/tagline** strip

### Layout Breakdown

```
┌─────────────────────────────────────────────────────────────┐
│ TISSO VISON   │  Find the ideal gift for your loved ones.  │ [CHOOSE GIFT →] │  ← Top Bar
├─────────────────────────────────────────────────────────────┤
│                                                             │
│   [Full-bleed background image: line-art illustrations]     │
│                                                             │
│  The Gift Guide          (bold heading, bottom-left)        │
│  Discover Joy: ...       (subheading paragraph)             │
│  [SHOP NOW →]            (black CTA button)                 │
│                                                             │
├─────────────────────────────────────────────────────────────┤
│     SUSTAINABLE, ETHICALLY MADE CLOTHES IN SIZES XXS TO 6XL │  ← Bottom bar
└─────────────────────────────────────────────────────────────┘
```

### Theme Editor Settings (schema)

**Top Bar:**
| Setting            | Type       | Description                                  |
|--------------------|------------|----------------------------------------------|
| brand_name         | text       | Left text (default: "TISSO VISON")           |
| tagline            | text       | Center text (default: "Find the ideal gift…") |
| nav_button_label   | text       | Button text (default: "CHOOSE GIFT")         |
| nav_button_url     | url        | Button link                                  |

**Hero:**
| Setting         | Type         | Description                                    |
|-----------------|--------------|------------------------------------------------|
| background_image| image_picker | Full-bleed line-art illustration               |
| heading         | text         | Main heading (default: "The Gift Guide")       |
| subheading      | richtext     | Paragraph text                                 |
| button_label    | text         | CTA text (default: "Shop Now")                 |
| button_url      | url          | CTA link                                       |

**Bottom Bar:**
| Setting       | Type | Description                                           |
|---------------|------|-------------------------------------------------------|
| bottom_text   | text | Tagline (default: "SUSTAINABLE, ETHICALLY MADE CLOTHES IN SIZES XXS TO 6XL") |

### Implementation Notes
- Top bar: 3-column flex row (brand | tagline | button), white/light bg
- Hero: `position: relative`, image fills full width via `object-fit: cover`
- Text overlay: `position: absolute; bottom: 0; left: 0` inside the hero
- Heading: large bold serif/sans, black text
- CTA button: black background, white text, uppercase, `→` arrow
- Bottom bar: centered text, small uppercase, thin strip
- Fully responsive: top bar collapses on mobile, text overlay stacks

---

## Section 2: `shoppable-gallery.liquid`

### What it is
A shoppable lifestyle image grid titled **"Tisso Vison in the Wild"**. A 3×2 grid of fashion photos. Each photo has a `+` hotspot button whose position is configurable via the theme editor. Clicking `+` opens an inline product quick-view popup.

### Layout

```
Tisso vison in the wild     ← section heading

[ Photo 1 [+] ] [ Photo 2 [+] ] [ Photo 3 [+] ]
[ Photo 4 [+] ] [ Photo 5 [+] ] [ Photo 6 [+] ]
```

### Popup Card (on + click)
```
┌──────────────────────────────────────┐
│                                   [×]│
│ [Thumbnail] │ Product Title          │
│             │ Price                  │
│             │ Description            │
│ Color: [White] [Black]               │
│ Size:  [Choose your size ▼]          │
│ [ADD TO CART →]                      │
└──────────────────────────────────────┘
```

### Theme Editor — Section Settings
| Setting | Type | Description                               |
|---------|------|-------------------------------------------|
| heading | text | Section title (default: "In the Wild")    |

### Theme Editor — Blocks (repeatable, max 6)
Each block = one grid cell:

| Setting   | Type         | Description                                      |
|-----------|--------------|--------------------------------------------------|
| image     | image_picker | Lifestyle photo                                  |
| product   | product      | Linked Shopify product for popup                 |
| hotspot_x | range 0–100  | Horizontal % position of `+` button on the image |
| hotspot_y | range 0–100  | Vertical % position of `+` button on the image   |

### Implementation Notes

**HTML Structure:**
- CSS grid: `grid-template-columns: repeat(3, 1fr)` with small gap
- Each cell: `position: relative`
- `+` button: `position: absolute`, placed using `left: hotspot_x%` and `top: hotspot_y%`
- Popup: `position: absolute`, z-index high, hidden by default (`display: none`)

**JavaScript Behavior:**
- Click `+` → show popup (add `is-open` class)
- Click `×` → hide popup
- Click outside any popup → close all open popups
- Only one popup open at a time

**Product Popup - Variant Logic:**
- Product variant structure: **Size** (XS, S, M, L) + **Color** (Black, White) — option1=Size, option2=Color
- Render size options as clickable buttons (from `product.options_by_name['Size'].values`)
- Render color options as clickable buttons (from `product.options_by_name['Color'].values`)
- On size/color button click → find matching variant → update price display + check availability
- "ADD TO CART" → POST to `/cart/add.js` with `variant_id` and `quantity: 1`
- On success → show "Added!" confirmation, update cart bubble count

**Responsive:**
- 2 columns on tablet (≤768px)
- 1 column on mobile (≤480px)
- Popup becomes full-width bottom sheet on mobile

---

## File Structure

```
sections/
  gift-guide.liquid             ← Section 1
  shoppable-gallery.liquid      ← Section 2
assets/
  gift-guide.css                ← Styles for Section 1
  shoppable-gallery.css         ← Styles for Section 2
  shoppable-gallery.js          ← JS: popup toggle + AJAX cart
```

---

## Build Order

1. **`gift-guide.liquid`** — Build first (simpler, no JS). Top bar + hero image overlay + bottom bar.
2. **`shoppable-gallery.liquid`** — Build second. Block structure + CSS grid + JS popup + AJAX cart.

---

## Key Constraints
- No `prestige-` prefix on section names
- Must work within the existing Horizon theme (no external dependencies)
- Vanilla CSS and JS only — no build tools
- AJAX cart via Shopify's native `/cart/add.js`
- `+` button position fully configurable per block via `hotspot_x` / `hotspot_y` range sliders
- **PIXEL PERFECT** — output must match the shared design screenshots exactly: spacing, font weight, button styles, top bar layout, hero overlay position, grid gaps, popup card layout
- Product variant structure: Size (XS/S/M/L) as option1, Color (Black/White) as option2
