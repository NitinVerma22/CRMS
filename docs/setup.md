# CRM SaaS MVP — Local Setup Guide

This guide provides step-by-step instructions for setting up and running the Telecalling & Lead Management CRM SaaS MVP on a local development workstation.

---

## 1. Prerequisites

Ensure your development machine has the following installed:

- **PHP**: 8.2 or higher (with extensions: `pdo`, `pdo_sqlite` or `pdo_mysql`, `mbstring`, `openssl`, `tokenizer`, `xml`, `ctype`, `json`, `curl`)
- **Composer**: 2.5 or higher
- **Node.js**: 18.x or 20.x LTS
- **npm**: 9.x or higher
- **Database**:
  - **MySQL 8.0+** (Recommended for production emulation) OR
  - **SQLite 3** (Default for zero-configuration, instant local development)

---

## 2. Project Architecture Overview

```
CRM/
├── Backend/        # Laravel 11 / PHP REST API
│   ├── app/        # Models, Controllers, Policies, Middlewares
│   ├── database/   # Migrations and Seeders
│   ├── routes/     # api.php (v1 prefix)
│   └── tests/      # Feature and Unit test suites
├── Frontend/       # Vite + React 19 + TypeScript + Tailwind CSS
│   ├── src/        # Components, Views, Lib, Contexts
│   └── dist/       # Optimized static production build
└── docs/           # Architecture, Deployment, RBAC & Demo guides
```

---

## 3. Backend Setup (Laravel 11)

### Step 3.1: Navigate to Backend
```bash
cd Backend
```

### Step 3.2: Install PHP Dependencies
```bash
composer install
```

### Step 3.3: Environment Configuration
Copy `.env.example` to `.env`:
```bash
cp .env.example .env
```

Generate your application key:
```bash
php artisan key:generate
```

### Step 3.4: Database Configuration (MySQL 8 Primary)

#### Primary Target: MySQL 8.0+
1. Create the MySQL database using utf8mb4 encoding:
   ```sql
   CREATE DATABASE crm_saas_mvp CHARACTER SET utf8mb4 COLLATE utf8mb4_unicode_ci;
   ```
2. Configure `.env` with your local MySQL credentials:
   ```env
   DB_CONNECTION=mysql
   DB_HOST=127.0.0.1
   DB_PORT=3306
   DB_DATABASE=crm_saas_mvp
   DB_USERNAME=root
   DB_PASSWORD=your_mysql_password
   ```

#### Optional: Zero-Setup SQLite Fallback
For quick, isolated local evaluation without a running MySQL service:
```env
DB_CONNECTION=sqlite
DB_DATABASE=database/database.sqlite
```

### Step 3.5: Run Migrations and Seed Demo Data

#### For a Fresh Setup:
Execute migrations and seed all roles, permissions, demo users, leads, and isolation fixtures:
```bash
# Run migrations to create all 26 schema tables
php artisan migrate

# Seed complete CRM dataset
php artisan db:seed
```

*(Note: In production environments, never run `migrate:fresh` or destructive commands without explicit backup verification.)*

### Step 3.6: Start Backend Dev Server
```bash
php artisan serve --host=127.0.0.1 --port=8000
```
Backend API will be accessible at `http://127.0.0.1:8000/api/v1`.

---

## 4. Frontend Setup (React + Vite + Tailwind CSS)

### Step 4.1: Navigate to Frontend
Open a new terminal window:
```bash
cd Frontend
```

### Step 4.2: Install Node Dependencies
```bash
npm install
```

### Step 4.3: Environment Configuration
Ensure `.env` matches your backend URL (defaults to `http://localhost:8000/api/v1`):
```bash
cp .env.example .env
```
Contents of `.env`:
```env
VITE_API_BASE_URL=http://localhost:8000/api/v1
```

### Step 4.4: Start Vite Dev Server
```bash
npm run dev
```
The application will be live at `http://localhost:5173`.

---

## 5. Demo Credentials

All seeded accounts use the password: `Password123!`

| Role | Email | Purpose / Experience |
| :--- | :--- | :--- |
| **Owner** | `owner@apexdynamics.com` | Full tenant access, metrics, team management, role permissions matrix |
| **Admin** | `admin@apexdynamics.com` | User management, user permission overrides, lead management, audit logs |
| **Manager** | `manager@apexdynamics.com` | Team queue, lead re-assignment, agent leaderboard, team follow-ups |
| **Telecaller** | `telecaller@apexdynamics.com` | Daily call queue, one-tap calling (`tel:`), call logging, scheduled follow-ups |
| **Telecaller 2** | `telecaller2@apexdynamics.com` | Secondary caller for testing team workloads and assignment |
| **Tenant B Owner** | `owner@horizon.com` | Used to verify tenant isolation (cannot see Apex Dynamics leads) |

> **Tip**: The login page (`http://localhost:5173/login`) provides one-click quick-fill buttons for instant switching between Owner, Admin, Manager, and Telecaller accounts.

---

## 6. Running Tests

### Backend Automated Tests
Run the feature tests covering authentication, tenant isolation, RBAC denial, user overrides, and duplicate lead checks:
```bash
cd Backend
php artisan test tests/Feature/CrmSecurityAndIsolationTest.php
```

### Frontend Production Build Check
Verify TypeScript compilation and Vite bundling:
```bash
cd Frontend
npm run build
```
