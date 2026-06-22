# JustMy.Health

JustMy.Health is a digital health & wellbeing platform that connects everyday users with therapists, healthcare providers, and wellness professionals through a guided, structured experience — **Connect → Engage → Educate → Empower**.

The platform currently focuses on **Online Counselling**, with Physical Training and Dietitian services in active development, alongside Business (B2B) and System Administration modules.

> ⚠️ This is an active, in-development product. Some modules are marked "Coming Soon" in the UI and APIs/config may change without notice.

---

## ✨ Features

- **Multi user-type platform** — Patients, Therapists, Business accounts (Local/Regional/National/Global), and tiered System Admins, each with role-based dashboards and a dynamically rendered sidebar menu.
- **Online counselling**
  - Patient onboarding questionnaire (40 Q&A) used to match patients with suitable therapists.
  - Therapist profile builder — BIO, photos, qualifications, salutation/languages, therapy types.
  - Therapist onboarding **verification → approval** workflow for admins.
  - Weekly availability calendar for therapists; patients book directly against open slots.
  - Live video/audio sessions via **ZegoCloud UIKit Prebuilt**, with a waiting room, post-session notes, and collateral document sharing.
  - In-app real-time messaging (ZegoCloud **ZIM**) backed by persistent DB chat history.
  - Session history, recordings (where available), and downloadable session resources.
- **Payments (Stripe)**
  - Therapist/business registration fees.
  - Patient session-credit packs (4/8/12 sessions), processed via Stripe Checkout + webhooks.
  - Session credit balance ledger (purchases vs. remaining balance).
- **Admin & reporting**
  - Revenue, payments, and platform operating cost dashboards (ApexCharts).
  - User growth/analytics by type, device, OS, and browser.
  - Configurable menu system, report access permissions, and auto-email templates.
- **Identity & access**
  - Username/password auth (Laravel Breeze-based) with email verification.
  - Social login (Google, Facebook, X/Twitter).
  - Keycloak SSO integration for cross-platform session sharing (e.g. social/community module).
  - Per-user device/session logging and single-active-session enforcement.

---

## 🛠 Tech Stack

| Layer            | Technology |
|-------------------|------------|
| Backend           | Laravel (PHP) |
| Frontend          | Blade templates + Alpine.js + Tailwind CSS |
| Build tooling     | Vite |
| Database          | MySQL |
| Realtime video    | ZegoCloud (UIKit Prebuilt) |
| Realtime chat     | ZegoCloud ZIM SDK |
| Payments          | Stripe Checkout + Webhooks |
| SSO               | Keycloak |
| Charts            | ApexCharts |
| Device detection  | Jenssegers/Agent |

---

## 🏗 Architecture & Module Structure

Controllers, views, and routes are organised by **module number**, matching the platform's internal numbering scheme:

```
app/Http/Controllers/Modules/
  Mod00UserAccess/              # mod-00 — Profile, auth-adjacent account management
  Mod01SystemAdministration/    # mod-01 — Table management, therapist mgmt, menu config
  Mod02SystemReporting/         # mod-02 — Device/user reports, finance reports
  Mod03SocialMedia/             # mod-03 — Social/community redirects (Keycloak SSO)
  Mod04MedicalData/             # mod-04 — Health news feed (in development)
  Mod05BusinessDirectory/       # mod-05 — Business finder (in development)
  Mod10ProfessionalServices/
    Counselling01/
      Patients/                 # Patient-facing counselling flows
      Therapists/               # Therapist-facing counselling flows

resources/views/modules/mod-XX/...   # Mirrors the controller structure above
routes/web.php                       # Routes grouped per module with `usertype:` middleware
```

### User types

| Code | Role |
|------|------|
| 1    | Standard Patient/User |
| 10 / 11 / 12 / 13 | Business — Local / Regional / National / Global |
| 30   | Therapist (Counselling) |
| 31   | Personal Trainer |
| 32   | Dietitian |
| 90 / 91 / 92 | Admin — Super / System / Finance |

### Conventions worth knowing

- Database columns and Eloquent attributes use **PascalCase** (`UserName`, `Email`, `CreditDate`…) with `ID` as the primary key — not Laravel's default `id`/`snake_case`.
- Many models disable Laravel's automatic timestamps (`public $timestamps = false`) and manage their own date/time columns.
- All interactivity is handled with **Alpine.js** directly inside Blade templates — there is no Vue/React SPA layer.
- Dynamic sidebar menus and route access are driven by the `sys_menu_display_options` table and the `usertype` route middleware.

---

## 🚀 Getting Started

### Requirements

- PHP 8.1+
- Composer
- Node.js & npm
- MySQL
- A Stripe account (test mode is fine)
- A ZegoCloud project (App ID + Server Secret)
- (Optional) A Keycloak realm if you want SSO/social-module integration

### Installation

```bash
git clone <repo-url>
cd justmy-health

composer install
npm install

cp .env.example .env
php artisan key:generate
```

Configure `.env` with at minimum:

```env
APP_URL=http://localhost

DB_CONNECTION=mysql
DB_HOST=127.0.0.1
DB_PORT=3306
DB_DATABASE=justmy_health
DB_USERNAME=root
DB_PASSWORD=

# Stripe
STRIPE_KEY=
STRIPE_SECRET=
STRIPE_WEBHOOK_SECRET=

# ZegoCloud (video)
ZEGOCLOUD_APP_ID=
ZEGOCLOUD_SERVER_SECRET=
ZEGOCLOUD_S3_BUCKET=
ZEGOCLOUD_S3_REGION=
ZEGOCLOUD_AWS_KEY=
ZEGOCLOUD_AWS_SECRET=

# ZegoCloud (in-app chat / ZIM)
ZEGOCHAT_APP_ID=
ZEGOCHAT_SERVER_SECRET=

# Keycloak (optional)
KEYCLOAK_BASE_URL=
KEYCLOAK_REALM=
KEYCLOAK_CLIENT_ID=
KEYCLOAK_CLIENT_SECRET=
KEYCLOAK_ADMIN_CLIENT=
KEYCLOAK_ADMIN_SECRET=
```

Then run:

```bash
php artisan migrate
php artisan storage:link
npm run build      # or: npm run dev
php artisan serve
```

### Stripe webhooks (local development)

```bash
stripe listen --forward-to localhost:8000/stripe/webhook
```

---

## 🧩 Key Integrations

- **Stripe** — `app/Services/StripeService.php` builds Checkout Sessions; `app/Http/Controllers/StripePayment/WebhookController.php` handles `checkout.session.completed` for registration fees and session-credit purchases.
- **ZegoCloud (video)** — `app/Http/Controllers/ZegoCloud/VideoSessionController.php` issues join credentials; the waiting room (therapist) and join page (patient) embed `ZegoUIKitPrebuilt`.
- **ZegoCloud (chat)** — `app/Http/Controllers/ZegoCloud/ZegoChatController.php` issues ZIM tokens via `app/Zego/Token04/ZEGO/ZegoServerAssistant.php`; chat history itself is persisted in `sys_user_message_history`.
- **Keycloak** — `app/Services/KeycloakService.php` handles admin-token retrieval, user provisioning, and direct-grant login used by the social/community SSO redirect.

---

## ⚠️ Known Limitations / Roadmap

- **Video token generation**: production currently relies on ZegoUIKitPrebuilt's `generateKitTokenForTest()` helper, which is **for development only**. This needs to be replaced with a proper signed server-side token to resolve intermittent Zego error codes (20014 / 50119) on the live site.
- **Session credit ledger**: purchases are recorded in `sys_finance_user_type_30_service_credits` and mirrored into a running balance table (`sys_finance_user_type_30_open_session_balance`) on purchase. The corresponding **debit** when a session is booked still needs to be wired up in `PatientsBookSlotsController`.
- **Personal Training** and **Dietitian & Healthy Eating** modules are placeholder "Coming Soon" pages.
- **Business Directory** and several Social Media (mod-03) features are still under active development.

---

## 🤝 Contributing

This is currently developed as a private client product mirrored to a public repository. If you'd like to contribute or report an issue, please open an issue describing the change clearly, including module/controller names where relevant.

## 📄 License

Add your chosen license here (e.g. MIT, proprietary/all-rights-reserved).
