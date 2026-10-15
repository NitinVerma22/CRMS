# CRM SaaS MVP — Production Deployment Guide

This guide describes how to deploy the **CRM SaaS MVP** into production with independent backend (Laravel REST API) and frontend (Vite React SPA) hosting.

---

## 1. Architecture & Deployment Strategy

```
                       ┌────────────────────────┐
                       │   End User (Browser)   │
                       └───────────┬────────────┘
                                   │
                    HTTPS Requests │
            ┌──────────────────────┴──────────────────────┐
            ▼                                             ▼
┌───────────────────────────────┐             ┌───────────────────────────────┐
│     Static Frontend Host      │             │      Backend API Server       │
│  (Vercel, Netlify, S3/CDN)    │             │  (Laravel Forge, VPS, Docker) │
│                               │             │                               │
│ - Vite React SPA              │  API Calls  │ - Laravel 11 REST API         │
│ - Built assets (dist/)        ├────────────►│ - PHP 8.2+ / FPM / Nginx      │
│ - VITE_API_BASE_URL (Env)     │  (/api/v1)  │ - Laravel Sanctum Auth        │
└───────────────────────────────┘             └───────────────┬───────────────┘
                                                              │
                                                              ▼
                                              ┌───────────────────────────────┐
                                              │      Managed MySQL 8.0+       │
                                              │    (AWS RDS / DigitalOcean)   │
                                              └───────────────────────────────┘
```

The frontend and backend are completely decoupled:
1. **Frontend**: Pure static bundle (`HTML/CSS/JS`) deployable to any edge CDN or static hosting platform.
2. **Backend**: Stateless PHP application server with database connections and Laravel Sanctum bearer token handling.
3. **Database**: Managed MySQL 8.0 database with connection pooling and automated backups.

---

## 2. Backend Deployment (Laravel 11)

### 2.1 Server Requirements
- **OS**: Ubuntu 22.04 LTS or 24.04 LTS
- **Web Server**: Nginx
- **PHP**: 8.2 or 8.3 with `php8.2-fpm`, `php8.2-mysql`, `php8.2-mbstring`, `php8.2-xml`, `php8.2-curl`, `php8.2-zip`, `php8.2-bcmath`
- **Database**: MySQL 8.0+
- **Process Manager**: Supervisor (for queue workers, if required in future phases)

### 2.2 Production Environment Variables (`.env`)
Configure the following in your production `.env` file:

```env
APP_NAME="Apex Dynamics CRM"
APP_ENV=production
APP_KEY=base64:GENERATE_VIA_PHP_ARTISAN_KEY_GENERATE
APP_DEBUG=false
APP_URL=https://api.yourcrmdomain.com
FRONTEND_URL=https://app.yourcrmdomain.com

# Database (Production MySQL)
DB_CONNECTION=mysql
DB_HOST=your-rds-endpoint.amazonaws.com
DB_PORT=3306
DB_DATABASE=crm_production
DB_USERNAME=crm_db_user
DB_PASSWORD=YOUR_STRONG_DATABASE_PASSWORD

# Session and Sanctum
SESSION_DRIVER=database
SESSION_LIFETIME=120
SESSION_ENCRYPT=true
SESSION_DOMAIN=.yourcrmdomain.com
SANCTUM_STATEFUL_DOMAINS=app.yourcrmdomain.com

# Cache & Queues
CACHE_STORE=database
QUEUE_CONNECTION=database
LOG_CHANNEL=stack
LOG_LEVEL=error
```

### 2.3 CORS & Security Configuration
In `Backend/config/cors.php`, ensure `allowed_origins` includes your production frontend domain:
```php
'paths' => ['api/*', 'sanctum/csrf-cookie'],
'allowed_methods' => ['*'],
'allowed_origins' => [
    env('FRONTEND_URL', 'https://app.yourcrmdomain.com'),
],
'allowed_headers' => ['*'],
'exposed_headers' => [],
'max_age' => 0,
'supports_credentials' => true,
```

### 2.4 Deployment Script Commands
Run these commands in your CI/CD pipeline or deployment hook:

```bash
# 1. Pull latest release code
git pull origin main

# 2. Install production dependencies without dev packages
composer install --no-dev --optimize-autoloader

# 3. Run database migrations safely
php artisan migrate --force

# 4. Cache configurations, routes, and events for maximum performance
php artisan config:cache
php artisan route:cache
php artisan view:cache

# 5. Set correct storage permissions
sudo chown -R www-data:www-data storage bootstrap/cache
sudo chmod -R 775 storage bootstrap/cache
```

### 2.5 Nginx Configuration Example
```nginx
server {
    listen 80;
    listen [::]:80;
    server_name api.yourcrmdomain.com;
    return 301 https://$host$request_uri;
}

server {
    listen 443 ssl http2;
    listen [::]:443 ssl http2;
    server_name api.yourcrmdomain.com;

    ssl_certificate /etc/letsencrypt/live/api.yourcrmdomain.com/fullchain.pem;
    ssl_certificate_key /etc/letsencrypt/live/api.yourcrmdomain.com/privkey.pem;

    root /var/www/crm/Backend/public;
    index index.php index.html;

    add_header X-Frame-Options "SAMEORIGIN";
    add_header X-Content-Type-Options "nosniff";
    add_header X-XSS-Protection "1; mode=block";

    location / {
        try_files $uri $uri/ /index.php?$query_string;
    }

    location ~ \.php$ {
        fastcgi_pass unix:/var/run/php/php8.2-fpm.sock;
        fastcgi_param SCRIPT_FILENAME $realpath_root$fastcgi_script_name;
        include fastcgi_params;
    }

    location ~ /\.(?!well-known).* {
        deny all;
    }
}
```

---

## 3. Frontend Deployment (Vite + React)

### 3.1 Build Environment Configuration
The frontend communicates with the backend via the build-time environment variable `VITE_API_BASE_URL`.

Create `.env.production` in `Frontend/`:
```env
VITE_API_BASE_URL=https://api.yourcrmdomain.com/api/v1
```

### 3.2 Compiling the Production Bundle
```bash
cd Frontend
npm ci
npm run build
```
This generates the optimized bundle in `Frontend/dist/`:
- `dist/index.html`
- `dist/assets/index-*.css` (Tailwind compiled CSS)
- `dist/assets/index-*.js` (Minified React code chunks)

### 3.3 Deploying to Vercel / Netlify / Cloudflare Pages

#### Option A: Vercel
1. Set Root Directory to `Frontend`.
2. Build Command: `npm run build`.
3. Output Directory: `dist`.
4. Environment Variables:
   - `VITE_API_BASE_URL`: `https://api.yourcrmdomain.com/api/v1`
5. Ensure SPA rewrites are enabled via `vercel.json`:
   ```json
   {
     "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
   }
   ```

#### Option B: Netlify
Create `Frontend/public/_redirects`:
```
/*    /index.html   200
```
Build Command: `npm run build`
Publish directory: `dist`

#### Option C: Nginx Static Hosting
```nginx
server {
    listen 443 ssl http2;
    server_name app.yourcrmdomain.com;
    root /var/www/crm/Frontend/dist;
    index index.html;

    location / {
        try_files $uri $uri/ /index.html;
    }
}
```

---

## 4. Post-Deployment Verification Checklist

1. **SSL Certificate**: Verify both `https://app.yourcrmdomain.com` and `https://api.yourcrmdomain.com` resolve over HTTPS.
2. **Health Check**: Request `https://api.yourcrmdomain.com/api/v1/auth/me` with invalid token; expect HTTP `401 Unauthorized`.
3. **Login Flow**: Sign in with an active tenant owner account; verify token issuance and dashboard metrics loading.
4. **Tenant Isolation**: Confirm that users in Tenant A receive HTTP `404 Not Found` when requesting resources belonging to Tenant B.
5. **CORS Headers**: Verify the browser console shows no CORS origin or pre-flight rejection errors.
