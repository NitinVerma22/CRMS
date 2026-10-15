# CRM SaaS MVP — Telecalling & Lead Management Platform

A multi-tenant, high-velocity Telecalling and Lead Management CRM SaaS platform built with **Laravel 11**, **MySQL 8** (with SQLite local zero-setup fallback), and **React 19 + TypeScript + Vite + Tailwind CSS**.

Engineered strictly following the **CRM SaaS MVP — Antigravity Master Build Guide**.

---

## 🚀 Key Highlights & Architectural Strengths

- **Multi-Tenant Isolation**: Hardened tenant isolation enforced at the database layer through global scopes and middleware (`EnsureTenantContext`). All requests are strictly bound to the authenticated tenant.
- **Dynamic Database-Driven RBAC**: Live permissions management with 30 granular permissions across 12 modules. Supports role baselines and explicit user-level overrides (`explicit deny` > `explicit grant` > `role grant`).
- **High-Velocity Telecaller Desk**: Touch-friendly call queue, one-tap calling (`tel:`), 12 standardized call outcomes (aligned with SRS), automatic follow-up scheduling, and lead timeline tracking.
- **Role-Tailored Experiences**: Dedicated views for **Owner** (Executive overview, teams, live RBAC matrix), **Admin** (User directory, overrides, audit logs), **Manager** (Supervisory queue, lead assignment, agent leaderboard), and **Telecaller** (High-dialing queue, personal logs).
- **Modern Purple-Pink Gradient Aesthetic**: Cohesive, responsive dark/light UI design featuring the exact `#6D28D9` → `#A855F7` → `#EC4899` gradient palette.
- **Independent Deployability**: Decoupled `Backend/` (Laravel REST API with Sanctum) and `Frontend/` (Vite SPA) connected via configurable environment variables.

---

## 📁 Repository Structure

```
CRM/
├── Backend/                    # Laravel 11 REST API
│   ├── app/
│   │   ├── Http/Controllers/Api/V1/ # REST Controllers (Auth, Leads, Calls, RBAC, etc.)
│   │   ├── Http/Middleware/         # Tenant Context & Permission Guards
│   │   └── Models/                  # Eloquent Models with tenant isolation
│   ├── database/
│   │   ├── migrations/              # 11 structured database migrations
│   │   └── seeders/                 # PermissionSeeder & CrmDemoSeeder
│   ├── routes/                      # api.php (v1 endpoints)
│   ├── tests/Feature/               # Automated security & isolation test suite
│   ├── .env.example                 # MySQL 8 & SQLite configurations
│   └── composer.json
├── Frontend/                   # React 19 + TypeScript + Vite SPA
│   ├── src/
│   │   ├── components/              # Buttons, Badges, Modals, StatCards, Layouts
│   │   ├── views/                   # Dashboard, Leads, Telecaller, RBAC, Calls, Teams
│   │   ├── lib/                     # API client, AuthContext, <Can> permission gate
│   │   └── App.tsx                  # Protected routing and layout wrapper
│   ├── dist/                        # Production build artifacts
│   ├── .env.example                 # VITE_API_BASE_URL configuration
│   └── package.json
├── docs/                       # Project Documentation
│   ├── setup.md                     # Step-by-step local development setup
│   ├── deployment.md                # Production hosting, Nginx, CORS, and build instructions
│   ├── permissions-matrix.md        # Complete dynamic RBAC and permissions catalogue
│   └── demo-script.md               # 5–8 minute executive demo walkthrough
└── CRM_SaaS_MVP_Antigravity_Master_Prompt_2.md # Single source of truth specifications
```

---

## ⚡ Quick Start (Local Development)

### 1. Backend Setup
```bash
cd Backend
composer install
cp .env.example .env
php artisan key:generate

# Run migrations and seed complete demo dataset (Users, Roles, Leads, Calls, Follow-ups)
php artisan migrate
php artisan db:seed

# Start backend server on port 8000
php artisan serve --host=127.0.0.1 --port=8000
```
Backend API will be active at: `http://127.0.0.1:8000/api/v1`

### 2. Frontend Setup
In a new terminal:
```bash
cd Frontend
npm install
cp .env.example .env

# Start frontend dev server on port 5173
npm run dev
```
Frontend web application will be active at: `http://localhost:5173`

---

## 🔑 Demo Accounts

All demo accounts share the password: **`Password123!`**

| Role | Email | Key Features & Access |
| :--- | :--- | :--- |
| **Owner** | `owner@apexdynamics.com` | Executive dashboard, all leads, teams, dynamic Role-Permissions Matrix |
| **Admin** | `admin@apexdynamics.com` | User management, user permission overrides, lead management, audit trail |
| **Manager** | `manager@apexdynamics.com` | Managed team leads, telecaller lead re-assignment, agent leaderboard |
| **Telecaller** | `telecaller@apexdynamics.com` | Daily call queue, one-tap calling (`tel:`), call logging, personal follow-ups |
| **Telecaller 2** | `telecaller2@apexdynamics.com` | Secondary caller for testing team workloads and assignment |
| **Tenant B Owner** | `owner@horizon.com` | Multi-tenant isolation test user (sees only Horizon Telecom data) |

> **Note**: The login screen includes quick-fill buttons for instant role switching without manual typing.

---

## 🧪 Automated Testing & Verification

### 1. Backend Test Suite
Run the automated security and isolation test suite:
```bash
cd Backend
php artisan test tests/Feature/CrmSecurityAndIsolationTest.php
```
**Results**:
- `test_user_can_login_and_receive_token_and_effective_permissions`: **Passed**
- `test_tenant_isolation_prevents_tenant_a_user_from_accessing_tenant_b_leads`: **Passed** (Returns 404 for cross-tenant access)
- `test_telecaller_only_receives_assigned_leads_in_lead_list`: **Passed** (Strict data scoping)
- `test_user_permission_override_explicit_deny_wins_over_role_grant`: **Passed** (Explicit deny precedence)
- `test_duplicate_lead_phone_warning_returned`: **Passed** (Returns 409 duplicate warning)

### 2. Frontend Production Build
```bash
cd Frontend
npm run build
```
**Results**:
- Built cleanly in `12.2s` with zero TypeScript errors and zero styling warnings.
- Output: `dist/index.html` (1.07 kB), `dist/assets/index-*.css` (31.54 kB), `dist/assets/index-*.js` (413.38 kB).

---

## 📋 Comprehensive Feature Checklist

### Completed MVP Features
- [x] Multi-tenant organization scoping and tenant middleware.
- [x] Database-backed Dynamic RBAC with 30 granular permissions.
- [x] Precedence evaluation: `Explicit Deny` > `Explicit Grant` > `Role Grant`.
- [x] Live UI Role Permissions Matrix with real-time toggle endpoints.
- [x] User-level permission override management modal.
- [x] 4 distinct operational role experiences: Owner, Admin, Manager, Telecaller.
- [x] Lead management: List, filter, search, pagination, create modal with duplicate phone check.
- [x] 360° Lead Detail view: Notes, Call History, Follow-ups, and Activity Timeline tabs.
- [x] Lead Assignment modal for managers and owners.
- [x] CSV bulk lead import and export flows.
- [x] Telecaller Home queue with "Up Next" prioritization card.
- [x] One-tap calling (`tel:`) protocol integration.
- [x] Call outcome modal with 12 standardized CRM outcomes, notes, and duration.
- [x] Automated follow-up task creation with datetime pickers.
- [x] Follow-ups list view with status filtering (`Open`, `Completed`, `Cancelled`).
- [x] Real-time dashboard KPI widgets and pipeline stage analytics.
- [x] Manager Supervisory view with Agent conversion leaderboard.
- [x] Teams and User management directories.
- [x] Immutable security Audit Log with actor IP and timestamps.
- [x] Tenant Settings view for custom statuses, sources, and call outcomes.
- [x] Cohesive purple-pink gradient design theme (`#6D28D9` → `#A855F7` → `#EC4899`).

### Explicitly Excluded Non-MVP Boundaries (Per Master Guide)
- No Super Admin / Platform Admin dashboard in this release.
- No SaaS subscription billing, payment gateways, or Stripe integrations.
- No AI calling agents, voice synthesis, or conversational bots.
- No external cloud telephony provider / IVR SDK integrations (telephony relies on native OS one-tap `tel:` links).

---

## 📚 Detailed Documentation

- [Setup Guide](file:///docs/setup.md) — Detailed instructions for local setup with MySQL 8 or SQLite.
- [Deployment Guide](file:///docs/deployment.md) — Production hosting, Nginx, CORS, and deployment scripts.
- [Permissions Matrix](file:///docs/permissions-matrix.md) — RBAC hierarchy, permission catalog, and override rules.
- [Demo Walkthrough Script](file:///docs/demo-script.md) — 5–8 minute manager walkthrough script for all 4 roles.
