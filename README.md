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

## Prérequis

- **Python 3.12+**
- **PostgreSQL 14+** (port 5432, user: `odoo`, password: `odoopwd`)
- **Odoo 19** (Community ou Enterprise)
- **Node.js / npm** (optionnel, pour outils JS)
- **Modules Odoo** : `website`, `website_sale`, `website_sale_wishlist`, `sale_loyalty`, `website_sale_loyalty`, `stock`, `delivery`

## Installation locale (step by step)

### 1. Installer Odoo 19

Télécharger Odoo 19 depuis [nightly.odoo.com](https://nightly.odoo.com/19.0/nightly/) ou installer via l'installateur Windows.

### 2. Installer PostgreSQL

```bash
# Vérifier que PostgreSQL tourne sur le port 5432
psql -U odoo -h localhost -p 5432
```

Créer l'utilisateur Odoo si nécessaire :

```sql
CREATE USER odoo WITH PASSWORD 'odoopwd' SUPERUSER;
```

### 3. Cloner le dépôt

```bash
git clone https://github.com/reblox01/sneakers-presentation.git
cd sneakers-presentation
```

### 4. Configurer Odoo

Éditer le fichier `odoo.conf` d'Odoo (généralement dans `C:\Program Files\Odoo 19.x\server\odoo.conf`) :

```ini
[options]
addons_path = C:\Program Files\Odoo 19.x\server\odoo\addons,C:\path\to\sneakers-presentation
admin_passwd = admin
db_host = localhost
db_port = 5432
db_user = odoo
db_password = odoopwd
db_name = sneakers
http_port = 8069
dev_mode = xml
```

> Remplacer `C:\path\to\sneakers-presentation` par le chemin réel du dossier cloné.

### 5. Lancer Odoo

```bash
# Windows
"C:\Program Files\Odoo 19.x\python\python.exe" "C:\Program Files\Odoo 19.x\server\odoo-bin" -c "C:\Program Files\Odoo 19.x\server\odoo.conf" -d sneakers -i sneakers --stop-after-init

# Puis relancer le serveur pour garder Odoo ouvert
"C:\Program Files\Odoo 19.x\python\python.exe" "C:\Program Files\Odoo 19.x\server\odoo-bin" -c "C:\Program Files\Odoo 19.x\server\odoo.conf"
```

```bash
# Linux / macOS
./odoo-bin -c odoo.conf -d sneakers -i sneakers --stop-after-init
./odoo-bin -c odoo.conf
```

> L'option `-i sneakers` installe le module pour la première fois.
> Sans `-i`, Odoo charge simplement le module s'il est déjà installé.

### 6. Accéder au site

- **Frontend** : http://localhost:8069/shop
- **Backend** : http://localhost:8069/web
- **Admin** : email `admin` / mot de passe `admin`

### 7. Mettre à jour le module (après modification de code)

```bash
# Option A : via l'URL du navigateur
# Aller dans Apps >邋Rechercher "Sneakers" >邋Mettre à jour

# Option B : via la ligne de commande (nécessite redémarrage)
"C:\Program Files\Odoo 19.x\python\python.exe" "C:\Program Files\Odoo 19.x\server\odoo-bin" -c "C:\Program Files\Odoo 19.x\server\odoo.conf" -d sneakers -u sneakers --stop-after-init
```

> **Note** : Odoo 19 n'a pas de `--dev=reload`. Tout changement de contrôleur Python nécessite un redémarrage manuel.

### 8. Redémarrer le serveur

```bash
# Windows — arrêter puis relancer
taskkill /F /IM python.exe
"C:\Program Files\Odoo 19.x\python\python.exe" "C:\Program Files\Odoo 19.x\server\odoo-bin" -c "C:\Program Files\Odoo 19.x\server\odoo.conf"
```

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
