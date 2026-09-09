# Rapport Technique — Module Sneakers (Odoo 19 E-Commerce)

## Table des matières
1. [Introduction](#1-introduction)
2. [Architecture Globale du Module](#2-architecture-globale-du-module)
3. [Flux de Fonctionnement](#3-flux-de-fonctionnement)
4. [Modélisation UML](#4-modélisation-uml)
5. [Architecture Technique](#5-architecture-technique)
6. [Analyse du Code Frontend](#6-analyse-du-code-frontend)
7. [Vérification et Validation](#7-vérification-et-validation)
8. [Conclusion et Perspectives](#8-conclusion-et-perspectives)

---

## 1. Introduction

### 1.1 Contexte du projet

Le projet consiste en la création d'un site e-commerce dédié à la vente de sneakers, développé sous la version 19 d'Odoo. Le module `sneakers` est construit sur les principes fondamentaux du framework Odoo tout en intégrant un frontend personnalisé.

### 1.2 Objectifs

- Créer une expérience e-commerce complète avec catalogue, panier, wishlist et paiement
- Intégrer des fonctionnalités avancées : code promo, newsletter, réseaux sociaux, livraison SendCloud
- Maintenir une architecture propre et maintenable en suivant les conventions Odoo
- Offrir un design responsive (mobile + desktop)

### 1.3 Technologies utilisées

| Technologie | Version/Type | Rôle |
|---|---|---|
| Odoo | 19.0.20260721 | Framework backend |
| Python | 3.10+ | Controllers, models |
| JavaScript (vanilla) | ES6+ | Interactivité frontend |
| CSS | Plain CSS | Styles (sans SCSS) |
| QWeb | Templates XML | Rendu côté serveur |
| PostgreSQL | 14+ | Base de données |
| GitHub | — | Version control |

---

## 2. Architecture Globale du Module

### 2.1 Structure des fichiers

```
sneakers/
├── __manifest__.py              # Métadonnées du module (v19.0.1.0.4)
├── __init__.py                  # Imports controllers + models
├── controllers/
│   ├── __init__.py
│   └── main.py                  # Deux classes : SneakersWebsiteSale, SneakersController
├── models/
│   ├── __init__.py
│   ├── product_template.py      # Héritage : brand_id, stock, country_of_origin
│   ├── product_brand.py         # NOUVEAU : marque produit
│   ├── delivery_carrier.py      # Héritage : intégration SendCloud
│   ├── newsletter.py            # NOUVEAU : subscriber + campaign
│   ├── social_post.py           # NOUVEAU : posts réseaux sociaux
│   └── sendcloud_shipping_method.py  # NOUVEAU : méthodes SendCloud
├── views/
│   ├── templates/
│   │   ├── layout.xml           # Header + footer globaux
│   │   ├── header.xml           # Barre supérieure + menu principal
│   │   ├── footer.xml           # Liens, newsletter, réseaux sociaux
│   │   └── product_card.xml     # Composant réutilisable
│   ├── pages/
│   │   ├── home.xml             # Page d'accueil
│   │   ├── shop.xml             # Catalogue avec filtres
│   │   ├── product.xml          # Fiche produit détaillée
│   │   ├── wishlist.xml         # Liste de souhaits
│   │   ├── cart.xml             # Panier
│   │   ├── checkout.xml         # Processus de commande
│   │   ├── payment.xml          # Paiement
│   │   ├── confirmation.xml     # Confirmation
│   │   └── ... (contact, terms, login, register, account, orders, static_pages)
│   ├── delivery_carrier_views.xml
│   └── sendcloud_shipping_method_views.xml
├── static/src/
│   ├── js/
│   │   ├── wishlist.js          # Gestion wishlist
│   │   ├── product.js           # Galerie, variantes, panier
│   │   ├── checkout.js          # Wizard 4 étapes
│   │   ├── cart.js              # Synchronisation DOM panier
│   │   ├── header.js            # Sticky header + hamburger
│   │   ├── search.js
│   │   ├── slider.js
│   │   ├── home.js
│   │   ├── faq.js
│   │   ├── confirmation.js
│   │   └── shop.js
│   └── css/
│       ├── variables.css        # Variables CSS (--primary-color, fonts, spacing)
│       ├── responsive.css       # Responsive (1556 lignes)
│       ├── js-ui.css            # Styles JS (sticky header, dropdowns)
│       ├── main.css             # Styles globaux
│       └── ... (14 autres fichiers)
├── data/
│   ├── product_brand_data.xml
│   ├── product_category_data.xml
│   ├── product_demo.xml
│   └── rating_data.xml
└── security/
    └── ir.model.access.csv      # 6 règles ACL
```

### 2.2 Dépendances du module

Le module `sneakers` dépend de sept modules Odoo standards :

| Module | Rôle |
|---|---|
| `website` | Infrastructure website Odoo |
| `website_sale` | Core e-commerce (produits, panier, checkout) |
| `website_sale_wishlist` | Wishlist (modèle `product.wishlist`) |
| `sale_loyalty` | Système de coupons/promotions |
| `website_sale_loyalty` | Interface web pour loyalty/coupons |
| `stock` | Gestion des stocks |
| `delivery` | Gestion des transporteurs |

### 2.3 Approche architecturale

Le module suit une **approche vanilla** — aucun framework frontend (OWL, React, etc.) n'est utilisé. Le code JavaScript est entièrement écrit en vanilla JS avec des IIFEs (Immediately Invoked Function Expressions), le CSS est du CSS plain (sans préprocesseur), et les templates utilisent le moteur QWeb d'Odoo.

Cette approche a été choisie pour :
- Minimiser les dépendances externes
- Faciliter la maintenance et la compréhension
- Assurer la compatibilité avec le système d'assets d'Odoo 19

---

## 3. Flux de Fonctionnement

### 3.1 Flux de checkout

Le checkout utilise un **wizard en 4 étapes** implémenté dans `checkout.js` :

```
Étape 1 : Facturation (Billing)
    ↓ Vérification champs obligatoires
Étape 2 : Expédition (Shipping)
    ↓ Sélection méthode de livraison
Étape 3 : Paiement (Payment)
    ↓ Sélection moyen de paiement
Étape 4 : Détails (Details)
    ↓ Récapitulatif
Confirmation → /confirmation
```

**Note** : La fonction `submitOrder()` dans `checkout.js` est un **stub** — elle redirige avec une référence aléatoire (`tx_ref`) sans validation côté serveur. Le traitement réel est délégué à Odoo.

### 3.2 Flux de paiement

Le paiement suit le flux standard Odoo avec une légère personnalisation :

```
Client → POST /payment
  → Redirect vers /shop/payment
    → Odoo gère le traitement
      → Retour vers /confirmation
        → Lecture transaction_ids + vérification wire transfer
```

Le contrôleur `payment()` dans `main.py` lit `transaction_ids` et vérifie si le paiement est un virement bancaire (`transfer`).

### 3.3 Flux de code promo (coupons)

Le système de coupons est géré via `sale_loyalty` + `website_sale_loyalty` :

```python
# controllers/main.py — _clear_order_coupons()
def _clear_order_coupons(self, order):
    # Supprime les lignes de récompense
    order.order_line.filtered('is_reward_line').unlink()
    # Supprime les coupons appliqués
    order.applied_coupon_ids = False
    # Réinitialise les points de coupon
    order.coupon_point_count = 0
    # Applique la politique de coupon unique
    # (un seul coupon actif à la fois)
```

Cette fonction est appelée avant chaque application de nouveau coupon pour éviter les conflits.

### 3.4 Flux de wishlist

La wishlist utilise le modèle intégré `product.wishlist` d'Odoo avec une contrainte importante : **le champ `website_id` est NOT NULL** — il doit être fourni lors de la création.

```
Client → Toggle wishlist (cœur)
  → JSON-RPC /shop/wishlist/toggle
    → Création/suppression dans product.wishlist
      → Mise à jour DOM (cœur rempli/vide)
        → Sync localStorage (déduplication via window.snWishlistLoaded)
```

**Points clés** :
- Déduplication via `window.snWishlistLoaded` pour éviter les double-chargements
- Sync localStorage pour persistance côté client
- Le cœur utilise des SVG custom (filled : viewBox 0 0 90 90, broken : viewBox 0 0 96 96)

### 3.5 Flux de newsletter

```python
# models/newsletter.py
class NewsletterSubscriber(models.Model):
    _name = "newsletter.subscriber"
    email = fields.Char(required=True, index=True, unique=True)
    state = fields.Selection([
        ('subscribed', 'Subscribed'),
        ('unsubscribed', 'Unsubscribed')
    ], default='subscribed')
```

```python
class NewsletterCampaign(models.Model):
    _name = "newsletter.campaign"
    name = fields.Char(required=True)
    subject = fields.Char(required=True)
    body_html = fields.Html(string="Body")
    state = fields.Selection([
        ('draft', 'Draft'),
        ('sent', 'Sent'),
        ('failed', 'Failed')
    ], default='draft')
```

---

## 4. Modélisation UML

### 4.1 Diagramme de classes

```
┌─────────────────────────────┐
│        product.template      │ (hérité)
│─────────────────────────────│
│ + brand_id : Many2one        │──→ product.brand
│ + website_availability       │
│ + stock_threshold : Integer  │
│ + country_of_origin          │──→ res.country
└─────────────────────────────┘

┌─────────────────────────────┐
│        product.brand         │ (NOUVEAU)
│─────────────────────────────│
│ + name : Char (required)     │
│ + logo : Image               │
└─────────────────────────────┘

┌─────────────────────────────┐
│     delivery.carrier         │ (hérité)
│─────────────────────────────│
│ + delivery_type (+=sendcloud)│
│ + sendcloud_public_key       │
│ + sendcloud_secret_key       │
│ + sendcloud_shipping_method  │──→ sendcloud.shipping.method
└─────────────────────────────┘

┌─────────────────────────────┐
│ sendcloud.shipping.method    │ (NOUVEAU)
│─────────────────────────────│
│ + name : Char                │
│ + sendcloud_id : Integer     │
│ + carrier : Many2one         │──→ delivery.carrier
│ + min_weight : Float         │
│ + max_weight : Float         │
│ + price : Float              │
│ + country_prices : Text      │
└─────────────────────────────┘

┌─────────────────────────────┐
│    newsletter.subscriber     │ (NOUVEAU)
│─────────────────────────────│
│ + email : Char (unique)      │
│ + state : Selection          │
└─────────────────────────────┘

┌─────────────────────────────┐
│    newsletter.campaign       │ (NOUVEAU)
│─────────────────────────────│
│ + name : Char                │
│ + subject : Char             │
│ + body_html : Html           │
│ + state : Selection          │
└─────────────────────────────┘

┌─────────────────────────────┐
│        social.post           │ (NOUVEAU)
│─────────────────────────────│
│ + platform : Selection       │
│ + state : Selection          │
└─────────────────────────────┘
```

### 4.2 Diagramme de séquence — Ajout au panier

```
Utilisateur          product.js         Controller         Odoo ORM          Database
    │                    │                  │                  │                  │
    │─ Clic "Ajouter" ─→│                  │                  │                  │
    │                    │─ POST /shop/cart ├→                 │                  │
    │                    │  /add (JSON-RPC) │                  │                  │
    │                    │                  │─ sale.order.line →│                  │
    │                    │                  │  .create()       │─ INSERT ────────→│
    │                    │                  │←─── OK ──────────│                  │
    │                    │←── redirect ─────│                  │                  │
    │←── Redirection ───│                  │                  │                  │
```

---

## 5. Architecture Technique

### 5.1 Backend (Python)

#### 5.1.1 Controllers

Le fichier `controllers/main.py` contient **deux classes** :

**`SneakersWebsiteSale`** (ligne 33) — hérite de `WebsiteSale` :
- Gère les routes e-commerce (panier, checkout, paiement)
- `_clear_order_coupons()` : nettoyage des coupons avant application

**`SneakersController`** (ligne 416) — hérite de `CustomerPortal` :
- Routes personnalisées (wishlist, newsletter, etc.)
- Endpoints JSON pour le frontend

**Note** : Pas de `--dev=reload` — nécessite un redémarrage manuel après modification des contrôleurs.

#### 5.1.2 Models

| Fichier | Lignes | Type | Description |
|---|---|---|---|
| `product_template.py` | 50 | Héritage | Ajoute brand_id, stock, country_of_origin |
| `product_brand.py` | 10 | Nouveau | Modèle marque (name + logo) |
| `delivery_carrier.py` | ~120 | Héritage | Intégration SendCloud (auth, test) |
| `sendcloud_shipping_method.py` | 30 | Nouveau | Méthodes de livraison SendCloud |
| `newsletter.py` | 49 | Nouveau | Subscriber + Campaign |
| `social_post.py` | 34 | Nouveau | Posts réseaux sociaux |

#### 5.1.3 Sécurité

Fichier `security/ir.model.access.csv` — 6 règles ACL :

| Modèle | Groupe | Lecture | Écriture | Création | Suppression |
|---|---|---|---|---|---|
| newsletter.subscriber | Utilisateur | ✓ | ✓ | ✓ | ✓ |
| newsletter.campaign | Utilisateur | ✓ | ✓ | ✓ | ✓ |
| social.post | Utilisateur | ✓ | ✓ | ✓ | ✓ |
| product.brand | Public | ✓ | — | — | — |
| product.brand | Utilisateur | ✓ | ✓ | ✓ | ✓ |
| sendcloud.shipping.method | Utilisateur | ✓ | ✓ | ✓ | ✓ |

### 5.2 Frontend

#### 5.2.1 JavaScript

**11 fichiers** — tous en vanilla JS avec le pattern IIFE :

```javascript
// Pattern utilisé dans chaque fichier
(function() {
    'use strict';
    // Code ici
})();
```

Aucun `odoo.define()`, aucun OWL, aucun framework.

**Fichiers principaux** :

| Fichier | Lignes | Rôle |
|---|---|---|
| `product.js` | ~300 | Galerie, variantes, panier, wishlist |
| `wishlist.js` | ~120 | Toggle/remove, déduplication localStorage |
| `checkout.js` | ~80 | Wizard 4 étapes, barre de progression |
| `cart.js` | ~150 | Parse/format prix, sync DOM |
| `header.js` | ~60 | Sticky header (seuil 80px), hamburger mobile |
| `search.js` | — | Barre de recherche |
| `slider.js` | — | Carrousel homepage |
| `home.js` | — | Page d'accueil |
| `faq.js` | — | FAQ |
| `confirmation.js` | — | Page de confirmation |
| `shop.js` | — | Page boutique |

#### 5.2.2 CSS

**18 fichiers CSS** — aucun SCSS.

| Fichier | Lignes | Rôle |
|---|---|---|
| `responsive.css` | 1 556 | Media queries, mobile breakpoints |
| `variables.css` | — | `--primary-color: #6E0F1A`, fonts, spacing |
| `js-ui.css` | — | Styles sticky header, dropdowns |
| `main.css` | — | Styles globaux |
| + 14 autres | — | Composants spécifiques |

**Couleur thème** : `#6E0F1A` (bordeaux) — pas `#FF5A00` (orange, ancienne valeur corrigée).

**Fonts** : Poppins (titres), Inter (body).

#### 5.2.3 Templates QWeb

| Template | Lignes | Rôle |
|---|---|---|
| `layout.xml` | — | Hérite `website.layout`, remplace header/footer |
| `header.xml` | — | Barre supérieure + menu + icônes |
| `footer.xml` | — | Liens, newsletter, réseaux sociaux |
| `product_card.xml` | 195 | Composant réutilisable (dual mode) |
| `home.xml` | 427 | Hero slider, catégories, produits, features |
| `shop.xml` | 702 | Filtres sidebar, grille produits, pagination |
| `product.xml` | 468 | Galerie, infos, sélecteurs, onglets |
| `wishlist.xml` | — | Grille wishlist + newsletter |
| + 11 autres | — | cart, checkout, payment, confirmation, etc. |

**Pattern `oe_structure oe_empty`** utilisé 49 fois — permet l'édition inline via Website Builder.

### 5.3 Modèle de données

#### Données statiques

| Fichier | Contenu |
|---|---|
| `product_brand_data.xml` | Marques (Nike, Adidas, etc.) |
| `product_category_data.xml` | Catégories (Homme, Femme, Enfant) |
| `product_demo.xml` | Produits de démonstration |
| `rating_data.xml` | Avis/notes |

#### Données de configuration

| Fichier | Contenu |
|---|---|
| `delivery_carrier_views.xml` | Vues admin transporteurs |
| `sendcloud_shipping_method_views.xml` | Vues admin SendCloud |

---

## 6. Analyse du Code Frontend

### 6.1 Pattern IIFE

Chaque fichier JavaScript utilise le pattern IIFE pour encapsuler le scope :

```javascript
// product.js
(function() {
    'use strict';

    // Toute la logique est encapsulée dans l'IIFE
    // Pas de pollution du scope global
    // Pas de dépendances externes
    // Pas de DOMContentLoaded — exécution immédiate au chargement du script

    var gallery = document.querySelector('.sn-product-gallery');
    if (gallery) {
        // Logique galerie ici
    }
})();
```

**Avantages** :
- Pas de conflit de noms
- Pas de pollution du scope global
- Code autonome et maintenable

### 6.2 Communication avec le backend

La communication se fait via JSON-RPC d'Odoo :

```javascript
// Exemple : toggle wishlist
fetch('/shop/wishlist/toggle', {
    method: 'POST',
    headers: {
        'Content-Type': 'application/json',
        'X-CSRF-Token': csrfToken
    },
    body: JSON.stringify({ product_id: productId })
})
.then(response => response.json())
.then(data => {
    // Mise à jour DOM
});
```

### 6.3 Sticky Header

```javascript
// header.js
(function() {
    'use strict';
    var threshold = 80;

    window.addEventListener('scroll', function() {
        var header = document.querySelector('.sn-header--sticky');
        if (window.scrollY > threshold) {
            header.classList.add('sn-header--scrolled');
        } else {
            header.classList.remove('sn-header--scrolled');
        }
    });
})();
```

### 6.4 Galerie produit

La galerie dans `product.js` gère :
- Image principale avec zoom
- Thumbnails cliquables
- Navigation par flèches
- Responsive (grid sur mobile)

### 6.5 Gestion des prix

`cart.js` contient les utilitaires de formatage :

```javascript
function parsePrice(priceStr) {
    // Parse le prix depuis le DOM
    // Gère les formats français (1 234,56 €)
}

function formatPrice(price) {
    // Formate le prix pour l'affichage
    // Retourne: "1 234,56 €"
}
```

---

## 7. Vérification et Validation

### 7.1 Check-list de vérification (20 points)

| # | Critère | Statut | Référence |
|---|---|---|---|
| 1 | Module manifest present et complet | ✓ | `__manifest__.py` |
| 2 | Models herites correctement | ✓ | `product_template.py`, `delivery_carrier.py` |
| 3 | Nouveaux models definis | ✓ | 5 nouveaux (brand, newsletter, social, sendcloud) |
| 4 | Controllers heritent de modules standards | ✓ | `WebsiteSale`, `CustomerPortal` |
| 5 | Templates heritent de `website.layout` | ✓ | `layout.xml` |
| 6 | CSS sans SCSS | ✓ | 18 fichiers CSS, 0 SCSS |
| 7 | JS vanilla sans OWL | ✓ | 11 fichiers IIFE |
| 8 | ACL definis | ✓ | 6 regles dans `ir.model.access.csv` |
| 9 | Donnees de demo | ✓ | 4 fichiers dans `data/` |
| 10 | Assets declares dans manifest | ✓ | 18 CSS + 11 JS |
| 11 | Wishlist inclut website_id | ✓ | Corrige dans PR #135 |
| 12 | Coupon system fonctionnel | ✓ | `_clear_order_coupons()` |
| 13 | Payment flow standard Odoo | ✓ | `/payment` → `/shop/payment` |
| 14 | Checkout wizard 4 etapes | ✓ | `checkout.js` |
| 15 | Header sticky fonctionnel | ✓ | `header.js` + `js-ui.css` |
| 16 | Mobile responsive | ✓ | `responsive.css` (1556 lignes) |
| 17 | Product card reutilisable | ✓ | `product_card.xml` (dual mode) |
| 18 | Newsletter subscription (modèle subscriber) | ✓ | `newsletter.subscriber` fonctionnel |
| 19 | Newsletter campaign (envoi emails) | ✗ | `newsletter.campaign` = modèle seul, pas d'envoi SMTP |
| 20 | Social post data model | ✓ | `social.post` avec sélection plateforme |
| 21 | Social media API publishing | ✗ | `action_mark_published` = stub, pas d'API externe |
| 22 | SendCloud auth + shipping methods | ✓ | `_sendcloud_auth()` Basic auth |
| 23 | SendCloud label generation | ✗ | Stub retourne tracking vide |
| 24 | Payment (standard Odoo) | ✓ | Flux `/payment` → `/shop/payment`, pas de provider custom |
| 25 | AI / n8n integration | ✗ | Non implémenté |

### 7.2 Points non vérifiés

| # | Élément | Raison |
|---|---|---|
| 1 | search.js (complet) | Non lu en profondeur |
| 2 | slider.js | Non lu en profondeur |
| 3 | home.js | Non lu en profondeur |
| 4 | faq.js | Non lu en profondeur |
| 5 | confirmation.js | Non lu en profondeur |
| 6 | shop.js | Non lu en profondeur |
| 7 | Templates: cart, checkout, payment | Non lus en détail |
| 8 | Templates: confirmation, static_pages | Non lus en détail |
| 9 | Vues admin SendCloud | Non inspectées |
| 10 | Fichiers demo data | Non inspectés en détail |

### 7.3 Bugs connus

1. **Odoo SH deploy error** : `ValueError: 'utm_campaign_id'` — module tiers cassé, pas du code sneakers
2. **Checkout stub** : `submitOrder()` ne valide pas les champs côté serveur
3. **Pas de `--dev=reload`** : nécessite redémarrage manuel du serveur

---

## 8. Conclusion et Perspectives

### 8.1 Réalisation

Le module `sneakers` est un **site e-commerce fonctionnel** qui :
- Intègre toutes les fonctionnalités essentielles (catalogue, panier, checkout, wishlist)
- Suit les conventions Odoo 19 (models, views, controllers)
- Utilise un frontend vanilla propre et maintenable
- Est responsive (mobile + desktop)
- Intègre des fonctionnalités avancées (coupons, newsletter data, SendCloud auth)

### 8.2 Points forts

- **Architecture propre** : pas de dépendances externes, code vanilla
- **Maintenabilité** : fichiers bien organisés, patterns cohérents
- **Responsive** : media queries complètes (1556 lignes)
- **Évolutivité** : models extensibles, CSS modulaire

### 8.3 Axes d'amélioration

- **Sécurité** : Ajouter la validation serveur pour `submitOrder()`
- **Performance** : Optimiser les images (WebP, lazy loading)
- **Tests** : Ajouter des tests unitaires Python et JS
- **CI/CD** : Pipeline de déploiement automatisé
- **Accessibilité** : Conformité WCAG 2.1

### 8.4 Déploiement

Le module est prêt pour le déploiement sur Odoo SH une fois l'erreur `utm_campaign_id` résolue par l'administrateur (Eric).

---

*Rapport généré le : 30 août 2026*
*Module sneakers v19.0.1.0.4 — Odoo 19*
