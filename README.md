<div align="center">

# 🛒 PHP E-Commerce Store

**A fully-featured single-vendor PHP e-commerce platform — products, cart, Stripe checkout, order tracking, and a powerful admin panel. All from scratch.**

[![PHP](https://img.shields.io/badge/PHP-8.x-777bb4?style=flat-square&logo=php&logoColor=white)](https://php.net)
[![MySQL](https://img.shields.io/badge/MySQL-8.x-4479A1?style=flat-square&logo=mysql&logoColor=white)](https://mysql.com)
[![Stripe](https://img.shields.io/badge/Stripe-Payments-635BFF?style=flat-square&logo=stripe&logoColor=white)](https://stripe.com)
[![License](https://img.shields.io/badge/license-MIT-a855f7?style=flat-square)](LICENSE)

[Features](#-features) · [Quick Start](#-quick-start) · [Architecture](#-architecture) · [Admin Panel](#-admin-panel) · [Configuration](#-configuration)

</div>

---

## The Problem

Building an e-commerce store usually means choosing between a bloated CMS (Magento, WooCommerce) that brings a mountain of complexity, or stitching together SaaS platforms that take a 2–3% cut of every transaction. This project is a clean, hand-rolled PHP + MySQL storefront — no framework baggage, no transaction fees beyond Stripe's own rate, and full control over every line of code.

---

## ✨ Features

**Storefront**
- 🗂️ **Product catalogue** with categories, subcategories, and tag-based filters
- 🔍 **Search** with relevance sorting (title, description, SKU)
- 🖼️ **Product pages** with image galleries, size/colour variant selectors, and stock indicators
- 🛒 **Session-based cart** — persists across browser tabs, no login required
- ❤️ **Wishlist** — save products for later, move to cart in one click
- 🔐 **User accounts** — register/login, address book, order history
- 💳 **Stripe Checkout** — hosted payment page, supports cards + Apple/Google Pay
- 📦 **Order tracking** — real-time status: Placed → Confirmed → Shipped → Delivered
- 📧 **Email receipts** — PHPMailer-powered HTML emails on order placed and shipped

**Admin Panel**
- 📋 **Inventory management** — add/edit/archive products, bulk stock update
- 🏷️ **Category management** — hierarchical categories, drag-and-drop ordering
- 🧾 **Order management** — view all orders, update status, add tracking number
- 👥 **Customer management** — view accounts, order history per customer
- 📊 **Dashboard** — revenue today / this week / this month, recent orders, low-stock alerts
- 🎟️ **Coupon codes** — percentage or fixed discount, usage limits, expiry dates

---

## ⚡ Architecture

```
Browser
  │
  ├─── GET /products, /cart, /checkout ──► PHP Router (index.php + .htaccess)
  │                                              │
  │                                    ┌─────────▼──────────┐
  │                                    │   Controller Layer   │
  │                                    │  ProductController   │
  │                                    │  CartController      │
  │                                    │  OrderController     │
  │                                    └─────────┬───────────┘
  │                                              │
  │                                    ┌─────────▼───────────┐
  │                                    │     Model Layer       │
  │                                    │  MySQL (PDO)          │
  │                                    │  products, orders,    │
  │                                    │  users, cart_items    │
  │                                    └─────────┬───────────┘
  │                                              │
  ├─── POST /checkout/pay ───────────────────────►│
  │         └── Stripe PHP SDK → Stripe API       │
  │                  └── webhook → order update ──┘
  │
  └─── /admin/* ──► Admin middleware (role check) ──► Admin Controllers
```

**Key Design Decisions**
- Pure PHP MVC — no framework dependency
- PDO with prepared statements throughout — no raw SQL interpolation
- Stripe webhook for reliable payment confirmation (not redirect-based)
- Session cart tied to `session_id`, merged to user cart on login

---

## 🚀 Quick Start

### Prerequisites

- PHP 8.x + Composer
- MySQL 8.x
- A Stripe account (test mode works)
- Apache with `mod_rewrite` or Nginx

### 1 — Clone and install

```bash
git clone https://github.com/harryatwork/php-ecommerce-store
cd php-ecommerce-store
composer install
```

### 2 — Database setup

```bash
mysql -u root -p
CREATE DATABASE ecommerce_store;
USE ecommerce_store;
SOURCE database/schema.sql;
SOURCE database/seed.sql;  # optional sample data
```

### 3 — Configure

```bash
cp config/config.example.php config/config.php
# Edit config.php — set DB credentials, Stripe keys, SMTP settings
```

### 4 — Configure Stripe webhook

In the Stripe dashboard → Webhooks → Add endpoint:
- URL: `https://yourdomain.com/webhooks/stripe`
- Events: `checkout.session.completed`, `payment_intent.payment_failed`

Copy the signing secret into `config.php` as `STRIPE_WEBHOOK_SECRET`.

### 5 — Point your web server

```apache
# Apache .htaccess (already included)
RewriteEngine On
RewriteCond %{REQUEST_FILENAME} !-f
RewriteRule ^ index.php [QSA,L]
```

Open `http://localhost/` — store homepage. Admin at `/admin` (default: `admin@store.com` / `admin123` — change immediately).

---

## 🛠️ Admin Panel

| Section | Path | What you do here |
|---|---|---|
| Dashboard | `/admin` | Revenue stats, recent orders, low-stock alerts |
| Products | `/admin/products` | Add, edit, archive, bulk stock update |
| Categories | `/admin/categories` | Create hierarchy, reorder by drag-and-drop |
| Orders | `/admin/orders` | View all orders, update status, add tracking |
| Customers | `/admin/customers` | Browse accounts, see order history per customer |
| Coupons | `/admin/coupons` | Create discount codes, set limits and expiry |
| Settings | `/admin/settings` | Store name, currency, SMTP, Stripe keys |

---

## ⚙️ Configuration

Edit `config/config.php`:

| Key | Default | Description |
|---|---|---|
| `DB_HOST` | `localhost` | MySQL host |
| `DB_NAME` | `ecommerce_store` | Database name |
| `DB_USER` | `root` | MySQL username |
| `DB_PASS` | `""` | MySQL password |
| `STRIPE_PUBLIC_KEY` | — | Stripe publishable key |
| `STRIPE_SECRET_KEY` | — | Stripe secret key |
| `STRIPE_WEBHOOK_SECRET` | — | Webhook signing secret |
| `SMTP_HOST` | — | Mail server host |
| `SMTP_USER` | — | SMTP username |
| `SMTP_PASS` | — | SMTP password |
| `SMTP_FROM` | — | Sender email address |
| `CURRENCY` | `GBP` | ISO 4217 currency code |
| `SITE_NAME` | `My Store` | Store display name |

---

## 📁 Project Structure

```
php-ecommerce-store/
├── config/
│   ├── config.php              # DB, Stripe, SMTP credentials
│   └── config.example.php      # Template — copy to config.php
├── controllers/
│   ├── ProductController.php
│   ├── CartController.php
│   ├── CheckoutController.php
│   ├── OrderController.php
│   └── UserController.php
├── models/
│   ├── Product.php
│   ├── Cart.php
│   ├── Order.php
│   └── User.php
├── views/
│   ├── layout/
│   │   ├── header.php
│   │   └── footer.php
│   ├── products/
│   ├── cart/
│   ├── checkout/
│   └── account/
├── admin/
│   ├── controllers/
│   ├── views/
│   └── middleware/
├── public/
│   ├── css/
│   ├── js/
│   └── images/
├── database/
│   ├── schema.sql
│   └── seed.sql
├── webhooks/
│   └── stripe.php              # Stripe webhook handler
├── .htaccess
└── index.php                   # Front controller / router
```

---

<details>
<summary><strong>Common issues and fixes</strong></summary>

| Issue | Fix |
|---|---|
| Blank page / 500 error | Enable PHP error display: `ini_set('display_errors', 1)` in `index.php` temporarily |
| Stripe webhook 400 | Wrong `STRIPE_WEBHOOK_SECRET` — copy it from Stripe Dashboard → Webhooks → your endpoint |
| Emails not sending | Check SMTP credentials; run `php -r "mail('test@example.com','test','test');"` to verify mail() works |
| Cart not persisting | Check `session_start()` is called before any output; verify PHP session dir is writable |
| Images not loading | Check `public/images/` is writable and `MAX_FILE_SIZE` in `config.php` |
| 404 on all routes | `mod_rewrite` not enabled — run `a2enmod rewrite` and restart Apache |
| Payment stuck in "pending" | Webhook not configured or not reaching your server — use Stripe CLI to test locally: `stripe listen --forward-to localhost/webhooks/stripe` |

</details>

---

## 📋 Requirements

```
PHP >= 8.0
MySQL >= 8.0
Composer
ext-pdo
ext-pdo_mysql
ext-curl
ext-json
stripe/stripe-php ^10
phpmailer/phpmailer ^6
```

---

<div align="center">

Built by [Harish K](https://github.com/harryatwork) · Full-stack PHP engineer

</div>