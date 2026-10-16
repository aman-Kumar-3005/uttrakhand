# Hotel inventory and booking CRM

## Overview

A server-rendered CRM for hotel listings, agent enquiries, room availability, quotations, and bookings. Admin and employee screens share PHP services and MySQL tables. This is plain PHP; the repository does not contain a Laravel application or Composer manifest.

## System architecture and modules

| Path | Responsibility |
| --- | --- |
| `hotel-manager.php` | Hotel and room inventory interface. |
| `rates.php` | Rate-management screen. |
| `bookingquery.php` | Agent enquiry and quotation workflow. |
| `booking-details.php` | Booking records and operational updates. |
| `agents-details.php / employees-detail.php` | Agent and employee management. |
| `ajax/` | JSON handlers for hotels, rooms, rates, availability, and bookings. |
| `includes/booking_service.php` | Reservation validation, date-based pricing, inventory locking, and cancellation updates. |
| `includes/crm_booking.php` | CRM booking/payment schema and service functions. |
| `includes/config.php / includes/db_connect.php` | Configuration loader and PDO connection/schema helpers. |
| `database/schema.sql` | Base schema without inserted records. |
| `scripts/install.php / scripts/migrate_bookings.php` | CLI installation and booking migration. |
| `tests/` | Isolated MySQL fixtures and HTTP/concurrency regression scripts. |

## Tech stack

PHP 8.1+ (the code uses `never` return types), PDO MySQL, MySQL/MariaDB, HTML, CSS, JavaScript. Apache rewrite rules are provided in `.htaccess`. PHP `mbstring` is used by booking validation; mail settings are separate from database setup.

## Local setup

Start with a new, disposable local database. Do not import the historical `database/all.sql` dump into an existing system.

```bash
git clone https://github.com/shreyasingh3005/uttrakhand.git
cd uttrakhand
cp .env.example.php .env.php
mysql -u root -p -e 'CREATE DATABASE crm_dev CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;'
```

Edit `.env.php`: set `APP_URL` to `http://127.0.0.1:8000`, `DB_HOST` to `127.0.0.1`, `DB_NAME` to `crm_dev`, and `DB_USER` / `DB_PASS` to a local database account. Configure mail separately. The loader also supports `CRM_DB_HOST`, `CRM_DB_NAME`, `CRM_DB_USER`, `CRM_DB_PASS`, and `CRM_APP_URL` overrides.

The installer creates the schema and an initial administrator. In Bash:

```bash
export CRM_ADMIN_USERNAME=localadmin
export CRM_ADMIN_EMAIL=admin@example.test
read -r -s -p 'Local admin password (12+ characters): ' CRM_ADMIN_PASSWORD
export CRM_ADMIN_PASSWORD
php scripts/install.php
unset CRM_ADMIN_PASSWORD CRM_ADMIN_USERNAME CRM_ADMIN_EMAIL
php -S 127.0.0.1:8000
```

Open `http://127.0.0.1:8000/index.php`. PHP's built-in server does not apply `.htaccess`; the extensionless agent lookup route needs Apache with `mod_rewrite` or an equivalent router. Keep this preview on loopback.

For regression testing, follow [tests/README.md](tests/README.md). Those scripts target a separate disposable database; they are not a production health check.

## Implementation notes and limits

- Data model: `users`, `agents_details`, `employees_details`, `agent_query_locks`, `activity_logs`, and `bookings_details` support CRM work. `hotels`, `hotel_room_categories`, `meal_plans`, `room_prices`, `hotel_bookings`, `booking_rooms`, and `room_availability` support hotel inventory.
- `booking_create()` locks the room category, checks each night, reserves inventory, and writes the booking in a transaction. Cancellation restores date-specific availability.
- The repository contains two related booking models. Schema helpers and migration code must be reviewed together; `schema.sql` is the starting point, not a complete description of all runtime schema changes.
- The tracked configuration and historical dumps require a credential/data review. Replace local settings before running anything. Any credentials previously used outside a disposable environment must be rotated; a later commit cannot erase old Git history.
- The existing workflow is named `laravel.yml`, but the application has no `artisan` or `composer.json`; it is not evidence of a Laravel build. Local setup and tests above have not been executed as part of this documentation pass.
