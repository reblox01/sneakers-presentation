# E-Commerce Project — Agent Instructions

## Project Overview
Odoo 19 e-commerce site (`sneakers` module) for Exocoms. Bootstrap-style frontend with vanilla JS, plain CSS, QWeb templates. **No SCSS, no OWL, no Odoo WebBuilder components.**

**GitHub:** `https://github.com/exocoms-teams/e-commerce`
**Dev SH:** `https://exocoms-e-commerce-sneekers-37146684.dev.odoo.com`
**Local:** `http://localhost:8069` (database: `sneakers`)

## Architecture

```
sneakers/
├── __manifest__.py          # v19.0.1.0.4
├── __init__.py              # imports controllers/ + models/
├── controllers/main.py      # Two classes: SneakersWebsiteSale (line 33), SneakersController (line 416)
├── models/                  # 5 new + 2 inherited
│   ├── product_template.py  # Inherited: brand_id, stock, country_of_origin
│   ├── product_brand.py     # NEW
│   ├── delivery_carrier.py  # Inherited: SendCloud
│   ├── newsletter.py        # subscriber + campaign (49 lines)
│   ├── social_post.py       # posts with platform selection (34 lines)
│   └── sendcloud_shipping_method.py
├── static/src/js/           # 11 files — ALL vanilla JS IIFEs, NO odoo.define, NO OWL
├── static/src/css/          # 18 files — plain CSS, no SCSS
├── views/templates/         # layout.xml, header.xml, footer.xml, product_card.xml
├── views/pages/             # home, shop, product, cart, checkout, payment, confirmation, wishlist, etc.
├── data/                    # brands, categories, demo products, ratings
├── security/                # ACL rules
└── i18n/                    # fr.po, ar.po
```

## Key Patterns

### Sticky Header
- `.sn-header--sticky` → `position: fixed; z-index: 999`
- Header base z-index: 1000 (layout.css)
- **120px tall on mobile** — watch for overlaps
- Mobile breakpoint: `@media (max-width: 767px)` in responsive.css

### Theme Colors
- **Primary: `#6E0F1A` (burgundy)** — NOT `#FF5A00` (that's old/wrong)
- Variables in `variables.css`: `--primary-color: #6E0F1A`
- Fonts: Poppins (headings), Inter (body)

### JS Convention
- ALL files use IIFE pattern: `(function(){ ... })()`
- NO `odoo.define()` — plain vanilla
- Key files: checkout.js (4-step wizard), wishlist.js (localStorage + Odoo toggle), product.js (image gallery)

### Templates
- QWeb XML with `t-call`, `t-if`, `t-foreach`
- `oe_structure oe_empty` used 49 times — enables Website Builder inline editing
- Inherits `website.layout` via `layout.xml`

### Wishlist
- Uses Odoo's built-in `product.wishlist` model
- **Must include `website_id` on create** (NOT NULL constraint)
- `/shop/wishlist/toggle` JSON route
- `window.snWishlistLoaded` dedup guard in wishlist.js

### Checkout
- Custom 4-step wizard: Billing → Shipping → Payment → Details
- Backend `submitOrder()` is a **stub** — redirects with random reference, no name attrs on inputs

### Payment
- `/payment` POST redirects to `/shop/payment`
- Actual processing delegated to Odoo
- `/confirmation` reads `transaction_ids` + wire transfer check

## Module Dependencies
`website`, `website_sale`, `website_sale_wishlist`, `sale_loyalty`, `website_sale_loyalty`, `stock`, `delivery`

## Other Modules in Repo
6 Odoo addons under `e-commerce/`: `sneakers`, `travel_agency`, `exocoms_signup_verify`, `produits_tendance`, `exocoms_debranding`, `luxury_services`

## Local Dev

### Server
- Odoo 19 at `C:\Program Files\Odoo 19.0.20260721`
- Config: `server\odoo.conf` (includes `e-commerce` in addons)
- **No `--dev=reload`** — restart required after controller changes
- Restart: `taskkill /F /IM python.exe` → relaunch

### Common Commands
```powershell
# Restart server
taskkill /F /IM python.exe
Start-Process -FilePath "C:\Program Files\Odoo 19.0.20260721\python\python.exe" -ArgumentList "C:\Program Files\Odoo 19.0.20260721\server\odoo-bin -c C:\Program Files\Odoo 19.0.20260721\server\odoo.conf"

# Update module (after code changes)
# From browser: /web#action=base.module_action&model=ir.module.module&view_type=list
# Or via CLI:
& "C:\Program Files\Odoo 19.0.20260721\python\python.exe" "C:\Program Files\Odoo 19.0.20260721\server\odoo-bin" -c "C:\Program Files\Odoo 19.0.20260721\server\odoo.conf" -d sneakers -u sneakers
```

## Deployment
- Odoo SH requires admin (Eric) to fix broken module (`utm_campaign_id` error)
- Eric may have assigned deploy to Ahmed
- PR #135 merged to main
- **No custom domain** set for SNEAKERS

## Rules
- **Theme color is burgundy `#6E0F1A`**, never orange
- **No SCSS** — plain CSS only
- **No OWL components** — vanilla JS IIFEs only
- **Include `website_id`** when creating wishlist items
- **No `--dev=reload`** — must restart server manually
- **PowerShell on Windows** — no bash syntax in commands
