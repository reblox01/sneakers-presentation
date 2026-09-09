# Sneakers E-Commerce — Odoo 19 Module

Module Odoo 19 pour un site e-commerce de sneakers (Exocoms).

## Structure

```
sneakers/
├── __manifest__.py              # v19.0.1.0.4
├── controllers/main.py          # Routes: shop, product, cart, wishlist, etc.
├── models/                      # 5 nouveaux + 2 hérités
│   ├── product_template.py      # brand_id, stock, country_of_origin
│   ├── product_brand.py         # Marques
│   ├── delivery_carrier.py      # SendCloud
│   ├── newsletter.py            # Abonnés + campagnes
│   ├── social_post.py           # Posts réseaux sociaux
│   └── sendcloud_shipping_method.py
├── static/src/js/               # 11 fichiers vanilla JS (IIFEs, pas de OWL)
├── static/src/css/              # 18 fichiers CSS (pas de SCSS)
├── views/pages/                 # Templates QWeb: home, shop, product, cart, etc.
├── views/templates/             # layout, header, footer, product_card
├── data/                        # Marques, catégories, produits démo, notes
├── security/                    # ACL rules
└── i18n/                        # fr.po, ar.po
```

## Installation

1. Copier le dossier `sneakers/` dans le répertoire addons d'Odoo
2. Ajouter le chemin dans `odoo.conf` :
   ```
   addons = C:\path\to\addons;C:\path\to\sneakers-presentation
   ```
3. Redémarrer Odoo
4. Aller dans **Apps > Mettre à jour le module** et chercher "Sneakers"

## Prérequis

- Odoo 19
- Python 3.10+
- Modules : `website`, `website_sale`, `website_sale_wishlist`, `sale_loyalty`, `website_sale_loyalty`, `stock`, `delivery`

## Screenshots

Voir le dossier `screenshots/` pour les captures d'écran du site.

## Rapports

- `rapport_technique.md` — Rapport technique complet
- `rapport_preuves_techniques.md` — Preuves techniques (code, commits, architecture)

## Diagrammes

Voir `diagrams/` pour les diagrammes PlantUML (use case, classes, composants).

## Auteurs

- Salma — UI/UX, CSS
- Sohail Koutari — Backend, Controllers, JS
