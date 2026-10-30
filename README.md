# My Shop: Django E-commerce Store

An online store built with Django: product catalog, session cart, discount coupons, orders, Stripe Checkout payments (test mode), PDF invoices, background e-mails with Celery, Redis-based product recommendations and an English/Spanish interface.

![Python](https://img.shields.io/badge/Python-3.10%2B-3776AB?logo=python&logoColor=white)
![Django](https://img.shields.io/badge/Django-5.0-092E20?logo=django&logoColor=white)
![Celery](https://img.shields.io/badge/Celery-5.2-37814A?logo=celery&logoColor=white)
![Stripe](https://img.shields.io/badge/Stripe-Checkout%20(test%20mode)-635BFF?logo=stripe&logoColor=white)
![Redis](https://img.shields.io/badge/Redis-recommendations-DC382D?logo=redis&logoColor=white)
![SQLite](https://img.shields.io/badge/SQLite-3-003B57?logo=sqlite&logoColor=white)

## Features

**Catalog and cart**
- Product catalog with categories, product images and availability.
- Session-based shopping cart (add, update quantity, remove), available in every template through a context processor.
- "Customers also bought" recommendations on the product and cart pages, computed from Redis sorted sets (`shop/recommender.py`).

**Coupons**
- Percentage discount coupons with a validity period and an active flag, applied to the cart and saved on the order.

**Orders and payments**
- Checkout form that creates the order and its items from the cart.
- Payment through a **Stripe Checkout** session built from the order items (designed for Stripe test mode).
- Stripe **webhook** endpoint that verifies the signature and marks the order as paid on `checkout.session.completed`.
- Asynchronous **Celery** tasks: an order confirmation e-mail, and an e-mail with a **PDF invoice** (rendered with WeasyPrint) after successful payment.

**Admin**
- Order list with links to a custom order detail page and to a PDF invoice.
- "Export to CSV" admin action for orders.

**Internationalization**
- English and Spanish translations (`gettext` with `.po`/`.mo` files) and `LocaleMiddleware`.

## Tech stack

| Area | Technology |
|------|------------|
| Backend | Python, Django 5.0 |
| Payments | Stripe Python SDK (Checkout Sessions, webhooks) |
| Background jobs | Celery (default broker: RabbitMQ on localhost) |
| Recommendations | Redis (`redis-py`) |
| PDF generation | WeasyPrint |
| Database | SQLite |
| i18n | Django translation framework (English, Spanish) |

## Project structure

```
Myshop/
├── myshop/        # Settings, URLs, Celery app
├── shop/          # Categories, products, catalog views, Redis recommender
├── cart/          # Session cart, cart views, context processor
├── coupons/       # Coupon model and "apply coupon" view
├── orders/        # Orders, checkout, Celery e-mail task, admin PDF/CSV
├── payment/       # Stripe Checkout, webhook, invoice e-mail task
├── locale/        # English and Spanish translations
├── static/        # Collected static files (CSS used for PDF invoices)
├── manage.py
└── requirements.txt
```

## Getting started

Requirements: Python 3.10+, a Redis server (`localhost:6379`), RabbitMQ for Celery (`localhost:5672`) and the system libraries WeasyPrint needs (Pango; see the [WeasyPrint installation guide](https://doc.courtbouillon.org/weasyprint/stable/first_steps.html)). Redis and RabbitMQ can be started with Docker:

```bash
docker run -d -p 6379:6379 redis
docker run -d -p 5672:5672 rabbitmq
```

Install and run the project:

```bash
git clone https://github.com/umar-rustamov/Myshop.git
cd Myshop

python -m venv .venv
# Windows: .venv\Scripts\activate
source .venv/bin/activate

pip install -r requirements.txt

# Environment variables (see .env.example); use Stripe *test* keys
export DJANGO_SECRET_KEY="<your-secret-key>"
export STRIPE_PUBLISHABLE_KEY="pk_test_..."
export STRIPE_SECRET_KEY="sk_test_..."
export STRIPE_WEBHOOK_SECRET="whsec_..."

python manage.py migrate
python manage.py createsuperuser
python manage.py runserver
```

In separate terminals, start the Celery worker and forward Stripe webhooks to the local server with the [Stripe CLI](https://stripe.com/docs/stripe-cli):

```bash
celery -A myshop worker -l info
stripe listen --forward-to localhost:8000/payment/webhook/
```

Open http://127.0.0.1:8000/, add categories and products in the admin (`/admin/`), and pay with a Stripe test card such as `4242 4242 4242 4242`. E-mails are printed to the console (`EMAIL_BACKEND` is the console backend).
