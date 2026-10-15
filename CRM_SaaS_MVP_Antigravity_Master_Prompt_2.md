
# CRM SaaS MVP — Antigravity Master Build Prompt (Laravel + React)
**Document purpose:** A single source of truth for planning, implementing, testing, and preparing a manager-ready MVP of a multi-tenant Telecalling & Lead Management CRM.

**Source baseline:** Telecalling & Lead Management CRM SRS v1.0, dated 10 October 2026, 13 pages.  
**Confirmed technology decisions:** Separate `Frontend/` (React + TypeScript + Vite + Tailwind CSS) and `Backend/` (Laravel PHP REST API + MySQL 8) folders; React PWA for telecallers; use Laravel queues only if asynchronous jobs are genuinely needed.  
**Confirmed product decisions:** Preserve the existing purple-pink mixed gradient visual theme and all existing SRS/MVP features; Owner, Admin, Manager, and Telecaller experiences; no Super Admin dashboard in this release; configurable, database-backed permissions so company administrators can control access by role and, where permitted, individual user.

---

# PART A — MASTER INSTRUCTIONS FOR ANTIGRAVITY

You are acting as a senior full-stack engineer, SaaS architect, product-minded UI engineer, QA engineer, and technical mentor. Build a working MVP of a multi-tenant Telecalling & Lead Management CRM based on the requirements below. The required architecture is a separately deployable React frontend and Laravel REST API backend; do not use Node/Express for the API.

Do not respond with only an architecture proposal or sample snippets. Inspect the current workspace, understand any existing code, make a plan, implement the project in the workspace, run the available checks, fix issues, and provide a concise completion report.

## 1. How to work safely and effectively

1. **Inspect before changing anything.** Examine the current directory, package files, source files, environment files, existing Tailwind setup, database scripts, and Git status if available.
2. If the workspace already contains a React/Tailwind project, preserve and extend it. Do not create a second frontend or overwrite working code unnecessarily.
3. If no project exists, create a clean monorepo structure as specified below.
4. First produce a short implementation checklist and file plan, then proceed with implementation. Do not stop after writing the plan.
5. Work in small, verifiable stages. After each major stage, run the relevant build, type check, lint, test, or migration checks available in the project.
6. Do not claim a feature works unless it has been implemented and checked. Clearly mark any incomplete or blocked feature.
7. Do not fabricate API responses or make buttons appear functional when no action exists. Use a visible loading, success, empty, and error state where appropriate.
8. Never place real secrets, database passwords, JWT secrets, or provider credentials in source code. Create `.env.example` with placeholder values only.
9. Do not run destructive database commands, delete existing user data, or overwrite important files without checking the consequences.
10. If a requirement cannot be completed in the available time, prioritize a stable end-to-end MVP and report the remaining work rather than building disconnected screens.
11. Keep code understandable for a beginner developer. Use descriptive names, a consistent folder structure, and short comments for non-obvious business rules.
12. Avoid unnecessary clarification questions. Make reasonable, clearly documented defaults where the SRS leaves decisions open; identify these as assumptions.

## 2. Product vision

Build a SaaS CRM that allows multiple independent companies to manage leads, telecalling activity, follow-ups, tasks, team performance, notifications, and basic reports through a shared application.

The core workflow is:

**Capture/import lead → validate and check duplicates → assign to a user/team → telecaller opens lead → initiates call → records call outcome and notes → schedules follow-up if needed → lead timeline and reports update → manager/owner monitors performance.**

The system must support separate company workspaces. A user from Company A must never read or modify Company B’s data.

### Product principles informed by the Salesforce CRM overview
Use the referenced Salesforce article as general CRM product inspiration—not as a requirement to copy Salesforce UI, wording, or proprietary features. Apply these principles within the already-approved MVP:
- **Single source of truth:** each lead has a reliable profile and chronological interaction history.
- **Shared context:** permitted users can see relevant lead, call, note, assignment, and follow-up history in one place.
- **Actionable workflow:** make the next follow-up, owner, status, and recent activity clear.
- **Useful reporting:** dashboards derive from real data and explain their metrics.
- **Customisation within scope:** company admins can configure allowed CRM statuses, sources, outcomes, team access, and permissions without code changes where the MVP supports it.
- Do not add Salesforce-only features such as marketing clouds, AI agents, commerce, service cases, advanced automation, or paid integrations to this MVP.

### Dynamic access-control requirement (MVP)
The CRM must not rely only on hard-coded role checks. Implement a database-backed permission system with seeded default role templates and a company-admin UI for managing permitted access. The default roles remain Owner, Admin, Manager, and Telecaller. A role template defines baseline permissions; an authorised Owner/Admin can grant or revoke individual permissions within the policy boundary. If per-user overrides are supported, they must be explicit and auditable. Owner must retain the minimum permissions required to administer the company workspace, and no user can grant permissions above their own authority. Super Admin/platform-wide controls remain out of scope.

## 3. Final technology stack — do not silently change it

### Frontend
- React
- TypeScript
- Vite, if a build tool is needed and one is not already configured
- Tailwind CSS
- React Router
- TanStack Query for server-state fetching/caching if useful
- React Hook Form + Zod for forms/validation if useful
- Lucide React icons
- Reusable components, responsive layouts, accessible labels, and consistent UI states

### Backend — `Backend/`
- Laravel (current stable version compatible with the selected PHP runtime)
- PHP version supported by that Laravel release
- Laravel REST API routes under `routes/api.php`
- Controllers kept thin; business rules in service/action classes where useful
- Form Request validation, API Resources for consistent JSON responses
- Laravel Policies/Gates plus database-backed permission checks
- Eloquent ORM and versioned Laravel migrations/seeders
- Laravel authentication suitable for a separately deployed SPA (prefer Sanctum with secure cookie/session configuration when deployment topology supports it; document the chosen SPA authentication flow and CSRF/CORS requirements)
- Rate limiting, structured application logs, centralized exception handling, and OpenAPI documentation where practical
- Do not add Node/Express as a backend. Node/npm may still be used to build the React frontend.

### Database
- MySQL 8 with InnoDB
- Versioned migrations and a documented schema
- Transactions for multi-step changes
- Foreign keys and indexes based on actual query patterns
- Connection pooling
- Seed script for demo data

Use Laravel Eloquent ORM and migrations. Keep schema, constraints, indexes, seeders, and generated SQL understandable. Do not make database setup dependent on an undocumented manual process.

### Telecaller app
- React PWA using the same frontend codebase where practical
- Mobile-first, installable experience
- One-tap call initiation through the device’s `tel:` mechanism for the MVP
- Manual call outcome logging unless a verified telephony integration is configured
- Do not claim the app can reliably capture native call duration, recordings, or call status without a supported provider/device mechanism
- Do not implement offline writes unless explicitly required and safely designed

### Background processing
- Laravel queues are **optional until needed**. Start with the database queue driver if it meets the MVP needs; use Redis only when there is a concrete performance/reliability reason.
- For a demo-sized MVP, implement simple reliable flows first. If CSV imports/exports, reminders, or notifications need asynchronous processing, use Laravel Jobs/Queues and document how to run the worker separately.
- Do not require Redis for every API request. BullMQ is not part of this Laravel architecture.

### Deployment principles
- Keep the application as a **modular monolith**, not microservices.
- Frontend should be buildable as static assets.
- Backend should be stateless where practical.
- Use Docker Compose for local development if useful, without making Docker the only way to run the project.
- Include health checks, logs, environment-based configuration, and backup notes.
- Do not introduce Kubernetes, paid services, or always-on infrastructure that is unnecessary for the MVP.

## 4. Scope boundaries

### In MVP — implement
- Authentication and session management
- Multi-tenant company/workspace foundation
- Role-based access for Owner, Admin, Manager, Telecaller
- Role-specific dashboards
- User and team management
- Lead create/read/update and permitted assignment/reassignment
- Lead search, filters, pagination, detail view, and activity timeline
- Duplicate warning using normalized phone and/or email
- CSV import with validation preview and row-level errors, to the extent feasible in the MVP
- Permission-controlled CSV export
- Call initiation and manual call logging/outcomes
- Call notes and history
- Follow-ups and tasks with due/upcoming/overdue states
- In-app notification centre; push notification support only if configuration and time allow
- Basic lead, call, follow-up, and team reports
- Audit events for sensitive and important actions
- Settings for initial statuses, sources, outcomes, and company preferences
- Loading, empty, error, success, and validation states
- Responsive desktop and mobile layouts
- Seed/demo data and setup documentation

### Explicitly deferred — do not build in this release
- Super Admin / platform operator dashboard
- Subscription billing, payment gateway, and plan enforcement
- Contacts, companies as sales objects, deals, and advanced sales pipeline
- Advanced custom fields builder
- WhatsApp/email/campaign integrations
- Marketing automation and advanced workflow builder
- Cloud telephony, IVR, guaranteed call recording, and call transcription
- AI call summaries/scoring/recommendations
- Advanced workforce analytics and field-sales/route optimisation
- Attendance/location tracking unless later approved

The `tenants`/workspace entity is still required for SaaS isolation. Deferring the Super Admin UI does **not** mean deferring tenant-aware architecture.

---

# PART B — USER ROLES AND AUTHORIZATION

## 5. Roles

### Owner
- View company-wide dashboard, leads, calls, follow-ups, reports, and audit events
- Create/edit/deactivate users and manage teams
- Assign/reassign leads
- Configure company CRM settings
- Access permitted exports and sensitive actions
- Manage Admin/Manager/Telecaller access according to defined policies

### Admin
- Manage leads, teams, users, assignments, and approved company settings according to permissions
- Import/export only when permitted
- Access reports within the granted scope
- Sensitive actions must be permission-checked and audited
- Do not assume every Admin has every Owner permission; make permissions explicit

### Manager
- View team-scoped leads and performance
- Assign/reassign leads within the allowed team scope
- Monitor due and overdue follow-ups
- Review permitted call activity and reports
- Cannot manage company-wide roles/settings unless explicitly granted

### Telecaller
- View only own or explicitly shared/assigned leads
- Create/update only fields allowed by policy
- Initiate a call, log an outcome, add a note, and schedule a follow-up
- View permitted call history, tasks, notifications, and own performance
- Cannot bulk export, manage users/teams, or reassign leads by default

## 6. Security rule — backend is the source of truth

- Enforce permissions in Laravel middleware, policies/gates, Form Requests, services, and scoped Eloquent queries and database query scope, not only by hiding frontend links.
- Resolve the authenticated user and tenant from verified Laravel authentication/session context. Never accept a tenant identifier or role sent by the browser as proof of authorization.
- Never trust a `tenant_id`, `role`, `owner_id`, or permission supplied by the browser without authorization checks.
- Every tenant-owned read, write, export, report, file operation, and background job must enforce tenant scope.
- Enforce team scope and record ownership as well as tenant scope.
- Return safe errors; do not leak another tenant’s record existence.
- Use modern password hashing such as Argon2id or bcrypt with appropriate parameters.
- Use secure cookies/session controls or a carefully implemented token strategy. Do not store long-lived sensitive tokens in localStorage by default.
- Add login throttling/rate limiting, input validation, safe CORS configuration, Helmet/security headers, and parameterized database access.
- Audit role changes, assignment/reassignment, import/export, record deletion/merge, and sensitive access.
- Never log passwords, authentication tokens, or unnecessary personal data.
- Test cross-tenant and cross-role access explicitly.

---

# PART C — DATABASE AND DOMAIN DESIGN

## 6A. Dynamic permissions and feature access (required MVP capability)

Implement configurable RBAC using permission keys stored in MySQL. Role names are not the permission system by themselves. Use a permission catalogue and role-permission mapping, and user-level overrides so an authorised administrator can control access for a specific user as well as by role.

### Permission model
- `permissions`: stable key, label, module, description, risk level, and whether it is configurable.
- `roles`: tenant-scoped role/template or clearly defined system role definitions.
- `role_permissions`: role-to-permission grants.
- `user_permissions` / `user_permission_overrides`: explicit grant/deny override with actor, reason, and timestamps.
- `users` and tenant membership remain tenant-scoped.
- Use a deterministic precedence rule and document it. Recommended: explicit deny override wins; otherwise explicit grant override; otherwise role grant. Never let a user override tenant isolation or grant a permission they are not allowed to delegate.
- Cache permissions only if useful; invalidate the cache immediately after permission changes or avoid caching for the MVP.

### Permission examples
- `dashboard.view`, `dashboard.view_company`, `dashboard.view_team`, `dashboard.view_own`
- `leads.view`, `leads.view_all`, `leads.view_team`, `leads.view_assigned`, `leads.create`, `leads.update`, `leads.delete`, `leads.assign`, `leads.bulk_assign`, `leads.import`, `leads.export`
- `calls.view`, `calls.log`, `calls.view_team`, `calls.view_all`
- `followups.view`, `followups.create`, `followups.update`, `followups.complete`, `followups.cancel`, `followups.view_team`
- `tasks.view`, `tasks.create`, `tasks.update`, `tasks.assign`
- `users.view`, `users.create`, `users.update`, `users.deactivate`
- `teams.view`, `teams.create`, `teams.update`, `teams.manage_members`
- `reports.view_own`, `reports.view_team`, `reports.view_company`, `reports.export`
- `notifications.view`, `settings.view`, `settings.manage`, `permissions.view`, `permissions.manage`
- `audit.view`

Treat this list as a permission catalogue to seed and refine against actual MVP screens. Do not add out-of-scope modules just because a permission example exists.

### Admin control UI
- Add an Access Control / Role Permissions page for Owner and specifically authorised Admin users.
- Display roles and feature/module permissions in a readable matrix with view/create/update/delete/assign/import/export scopes where relevant.
- Allow saving changes with confirmation, loading state, success/error state, and audit entry.
- Clearly show locked/system-critical permissions that cannot be delegated or removed.
- Provide a user detail permission panel that shows inherited role grants separately from overrides and explains the effective permission.
- Changes should take effect on the next API request (and refresh the frontend permission state); do not require a deployment or source-code edit.
- Navigation and buttons should reflect permissions, but frontend hiding is only usability; Laravel remains the enforcement authority.

### Enforcement rules
- Check permissions on every protected API endpoint, including direct URL/API calls.
- Scope records by tenant, team, and ownership independently of the permission key.
- A `leads.view` grant must not automatically mean `leads.view_all`.
- Exports, imports, assignment, role changes, and settings changes need explicit permissions and audit events.
- Prevent privilege escalation: a Manager cannot grant company-wide access; a Telecaller cannot manage users/permissions; an Admin cannot grant Owner-only or platform-level capabilities unless an explicit policy permits it.
- Write feature tests for each role and critical deny path, including cross-tenant attempts.
- Seed sensible defaults so the demo works immediately, but let the authorised company administrator change them from the UI.

### Dynamic configuration vs feature flags
Permission controls govern who can use an existing feature. They do not create new features or replace subscription/plan entitlements. Billing and plan enforcement are out of scope. Do not build a Super Admin dashboard.

## 7. Data model foundation

Design and document a relational schema. At minimum, consider:

- `tenants` / `workspaces`
- `users`
- `teams`
- `team_members` if a many-to-many membership model is needed
- `roles`, `permissions`, `role_permissions`, and `user_permission_overrides` for database-backed dynamic access control
- `leads`
- `lead_notes` or a suitable note model
- `lead_activities` / `timeline_events`
- `call_logs`
- `follow_ups`
- `tasks` if tasks need a distinct lifecycle
- `notifications`
- `audit_events`
- `import_jobs` and import row errors if CSV processing is tracked
- `tenant_settings` or tenant-level configuration tables

For every tenant-owned entity, define tenant scope consistently. Add `created_at`, `updated_at`, and where appropriate `created_by`, `updated_by`, status, and soft-deactivation/deletion fields.

Suggested lead fields include:
- ID
- tenant ID
- name
- normalized phone and original phone
- email and normalized email when present
- source/campaign where available
- status
- owner/user ID
- team ID where applicable
- notes/attributes as appropriate
- created/updated timestamps

Suggested call log fields include:
- tenant ID, lead ID, user ID
- call direction if supported
- initiation/start/end timestamps when available
- duration only when reliably available
- call status/outcome
- notes
- provider/reference fields only if a real provider is integrated

Suggested follow-up fields include:
- tenant ID, lead ID, owner ID
- type/title
- due timestamp
- status: open/completed/cancelled, with overdue derived from due time and open status where appropriate
- completion/reschedule metadata
- notes

### Database design requirements
- Use UTC for stored timestamps and display in the tenant/user timezone.
- Use transactions for multi-step changes such as recording outcomes and creating follow-ups.
- Prevent duplicate/retried integration or job processing when relevant through idempotency controls.
- Use efficient pagination and indexes based on actual query patterns.
- Avoid loading all leads/call logs into application memory.
- Do not use unbounded queries.
- Do not store passwords in plaintext.
- Do not silently merge duplicates; flag likely matches and require permitted user action.
- Provide migration and seed commands and document how to reset only development/demo data safely.
- Document a future path to archive/partition large activity tables if measured scale eventually requires it.

## 8. Multi-tenant data isolation

Use a shared database/shared tables with `tenant_id` for the initial SaaS implementation unless a concrete technical blocker is found.

Requirements:
1. Tenant context comes from authenticated identity/session, not from a freely editable browser field.
2. Tenant scope is applied centrally and consistently.
3. Composite indexes should begin with `tenant_id` when matching query patterns justify it.
4. Use tenant-aware foreign keys/constraints where practical.
5. Background jobs must carry a tenant identifier and revalidate authorization/context.
6. Reports, exports, file downloads, notifications, and audit views must be tenant-scoped.
7. Add automated tests proving that users from Tenant A cannot read or mutate Tenant B records.
8. Do not create a database/server per customer by default.

---

# PART D — SCREEN AND WORKFLOW REQUIREMENTS

## 9. Common application shell

Create a professional CRM interface with:
- Desktop sidebar and top navigation
- Workspace/company identity and current user menu
- Role-appropriate navigation items
- Responsive mobile navigation
- Notification indicator
- Profile/logout actions
- Consistent page title, breadcrumbs where helpful, action buttons, filters, and tables/cards
- Accessible keyboard focus, labels, readable contrast, and clear errors
- Purple-pink gradient used selectively for brand moments and primary actions; data-heavy screens should remain readable

Do not display a Super Admin link or Super Admin dashboard in this release.

## 10. Authentication and onboarding

Implement:
- Sign-in page
- Logout
- Session expiry/unauthorized handling
- Password reset flow only if it can be implemented safely; otherwise provide a documented development-only reset procedure and mark production email delivery as pending
- Tenant-aware user membership
- Role-based landing page after sign-in
- Protected routes
- Clear unauthorized and session-expired states

For demo purposes, seed an Owner, Admin, Manager, and Telecaller under a demo tenant. Provide demo credentials in local-only documentation, not hard-coded into production code. Make it clear seeded credentials must be changed and are not production defaults.

If self-service tenant registration is not fully implemented, use a documented seed/admin provisioning process to create demo workspaces and users. Do not invent a Super Admin panel to support provisioning.

## 11. Role-specific dashboards

### Owner dashboard
- Total/new leads
- Assigned/unassigned leads
- Leads by status
- Calls attempted and outcomes where logged
- Follow-ups due today and overdue
- Team performance
- Date-range filter
- Recent activity/audit highlights when permitted

### Admin dashboard
- Lead flow and assignment overview
- Team/user summary
- Import status
- Follow-up backlog
- Relevant reports within permissions

### Manager dashboard
- Team performance
- Assigned/unassigned team leads
- Calls/outcomes by team member
- Due/overdue follow-ups
- Team/date filters

### Telecaller home
- My assigned leads
- Due and overdue follow-ups
- Daily call progress/targets only if targets are configured
- Next eligible lead
- Recent activity and notifications
- Mobile-first quick actions

KPI values must be based on actual database data, not static hard-coded dashboard numbers. Seed demo records so the manager can see useful data.

## 12. Lead management

Implement:
- Lead list with server-side pagination
- Search by name, phone, email, and supported identifiers
- Filters for status, source, owner, team, creation date, and next follow-up where available
- Create/edit lead form with validation
- Lead detail page/drawer
- Assignment/reassignment action based on role and scope
- Status changes with configured/valid transitions
- Notes with actor and timestamp
- Chronological timeline showing creation, notes, calls, outcomes, follow-ups, status changes, and assignment changes
- Duplicate warnings based on normalized phone and/or email
- Bulk assignment/status update only for authorized roles
- Restricted and audited delete/merge actions; do not silently merge records

Initial lead statuses based on the SRS:
- New
- Assigned
- Contacted
- Interested
- Follow-up
- Qualified
- Converted
- Lost
- Invalid

Store statuses in a configurable or seedable structure where practical. A converted/lost lead’s open follow-ups should follow a documented MVP policy.

## 13. Lead assignment

Implement:
- Manual assignment to an eligible user/team
- Assignment history with previous owner, new owner, actor, timestamp, and optional reason
- Assignment notification in the app
- Deactivated users must not receive new assignments
- Round-robin assignment is a SHOULD requirement in the SRS; include it only if the core MVP is stable and there is time. If omitted, mark it as a documented follow-up, not as complete.

## 14. Calling and call logging

For MVP:
- Provide one-tap call initiation via the mobile device’s `tel:` link.
- After returning from a call, let the telecaller manually record the call attempt/outcome and note.
- Allow permitted users to view call history for a lead and their allowed team/user scope.
- Show the outcome in the lead timeline.
- Capture call duration/status only if reliably supplied; otherwise leave unknown rather than inventing data.
- Do not promise native call recording or automatic native dialler logs.

Initial call outcomes from the SRS:
- Connected
- Interested
- Follow-up Required
- Call Later
- Busy
- No Answer
- Not Reachable
- Wrong Number
- Not Interested
- Demo/Meeting Scheduled
- Converted
- Lost

Outcome-driven rules:
- Follow-up Required should require a follow-up date/time.
- Lost should require a lost reason if the configuration requires it.
- Save call log, lead status/timeline update, and follow-up creation safely, using a transaction when appropriate.

## 15. Follow-ups, tasks, and calendar

Implement:
- Create follow-up linked to a lead and owner
- Due date/time and optional note
- Today, upcoming, overdue, completed, and cancelled views
- Complete, reschedule, and cancel actions
- Task type/title and assignee
- In-app reminder/notification where feasible
- Manager escalation after a configurable threshold only if the base reminders are stable
- Calendar view is a SHOULD requirement; if not feasible for the demo, make a clear list-based task workflow first and document calendar as pending

Overdue status should be derived consistently from open status and due time. Do not mark a task as completed merely because the due date passed.

## 16. CSV import/export

CSV import flow:
1. Upload CSV.
2. Validate file type/size and parse safely.
3. Show column mapping or a clear supported template.
4. Preview rows and validation errors.
5. Detect likely duplicates.
6. Confirm import.
7. Show imported, skipped, and failed row counts with row-level error details.
8. Make retry behavior safe and avoid accidental duplicate imports.

CSV export:
- Only authorized users can export.
- Export must be tenant- and permission-scoped.
- Log the export event.
- Exclude fields the user is not permitted to access.
- For large exports, use a background job when needed.

## 17. Notifications

Implement an in-app notification centre for:
- New lead assignment
- Reassignment
- Follow-up reminders
- Overdue alerts
- Relevant manager/owner alerts

Track read/unread state. Push delivery is dependent on browser/device configuration and can be deferred if not ready. Never show a false “delivered” state if the provider does not confirm delivery.

## 18. Reports and analytics

MVP reports:
- Leads by status, source, owner, team, and date range
- Call attempts/outcomes by user/day where reliable data exists
- Follow-ups due, completed, missed, and overdue
- Team performance within permitted scope

Use consistent metric definitions:
- New leads: leads created in the selected period, interpreted using workspace timezone
- Calls attempted: logged attempts in the period; document whether failed attempts are included
- Connected calls: only count when explicitly and reliably marked
- Connect rate: connected calls divided by attempts; show N/A if denominator is zero
- Overdue follow-ups: open follow-ups whose due time is earlier than now, using the correct timezone
- Average duration: calculate only from calls with valid duration
- Conversion rate: document the cohort/date basis and denominator

Dashboard/report queries must use real database data. Avoid expensive repeated full-table scans. For the MVP, indexed SQL aggregation is acceptable; add summary tables/caching only when profiling justifies it.

## 19. Audit trail

Record important actions:
- Login/security events where appropriate
- Lead creation/update/status change
- Assignment/reassignment
- User and role changes
- Import/export
- Delete/merge
- Sensitive settings changes

Store actor, tenant, action, object type/ID, timestamp, and relevant before/after metadata where appropriate. Do not put secrets or unnecessary personal data in audit logs.

---

# PART E — UI DESIGN SYSTEM

## 20. Brand and visual direction

Theme: **Purple + pink mixed gradient**, modern B2B SaaS, polished but practical for long work sessions.

Suggested palette:
- Deep purple: `#6D28D9`
- Violet: `#A855F7`
- Pink: `#EC4899`
- Light purple surface: `#F5F3FF`
- Main text: `#18181B`
- Secondary text: `#71717A`
- Page background: `#FAFAFC`
- Borders: `#E4E4E7`
- Success: `#16A34A`
- Warning: `#D97706`
- Error: `#DC2626`

Suggested brand gradient: `linear-gradient(120deg, #6D28D9 0%, #A855F7 48%, #EC4899 100%)`.

Design rules:
- Use gradients selectively for primary buttons, active accents, onboarding and small brand elements.
- Do not use intense gradients behind dense tables or body text.
- Use clean neutral surfaces, rounded cards, restrained shadows, consistent spacing, and strong visual hierarchy.
- Status colors must be consistent across lead tables, forms, badges, and dashboards.
- Keep the telecaller interface mobile-first with large touch targets and minimal steps.
- Include skeleton/loading, empty, error, and success states.
- Make layouts responsive for common laptop, tablet, and Android mobile widths.
- Do not use excessive animation that slows the app or harms usability.
- Use a coherent component system rather than styling each page independently.

---

# PART F — ARCHITECTURE, PERFORMANCE, AND SCALABILITY

## 21. Backend module structure

Use a modular monolith with clear modules, for example:
- auth
- tenants/workspaces
- users
- teams
- roles/permissions
- leads
- assignments
- calls
- follow-ups/tasks
- notifications
- reports
- imports/exports
- audit
- settings

Each Laravel module should separate API routes, controllers, Form Requests, policies, services/actions, Eloquent models, and resources where appropriate. Avoid one enormous controller or route file and avoid premature microservices.

## 22. API standards

- Use Laravel `routes/api.php` and consistent versioned REST paths (for example `/api/v1/...`) and response/error shapes.
- Validate path parameters, query parameters, and request bodies.
- Apply authentication, tenant scope, and permission checks to protected endpoints.
- Use pagination and bounded limits.
- Provide clear 400/401/403/404/409/422/429/500 behavior where applicable.
- Use safe transaction handling for multi-step business operations.
- Add request IDs and useful structured logs.
- Keep API documentation aligned with the actual endpoints.
- Never return internal stack traces or secrets to the client.

## 23. Cost-aware performance rules

- Serve the frontend as static assets where practical.
- Start with one modular API deployment and one MySQL instance sized to the actual pilot; do not claim it can handle unlimited traffic.
- Add Laravel queues and Redis only when jobs require reliable asynchronous execution.
- Use indexes and inspect query plans for slow list/report queries.
- Do not fetch every record and filter in the browser.
- Use Laravel/PDO connection handling appropriately for the chosen PHP deployment model; avoid unnecessary repeated connections within a single request/job.
- Keep endpoints stateless where practical.
- Cache only data that benefits from caching, with clear invalidation rules.
- Avoid polling every few seconds without a concrete need.
- Keep file uploads and exports out of synchronous requests when they become expensive.
- Add pagination to all potentially large lists.
- Keep external integrations optional and failure-tolerant.
- Document scaling triggers: API CPU/memory, DB CPU/slow queries/connections, queue delay, and latency.
- Do not introduce Kubernetes, microservices, read replicas, or costly managed products without measured need.

## 24. Security and privacy baseline

- HTTPS in production.
- Secure password hashing.
- Protected session/token handling.
- Tenant isolation and server-side authorization.
- Input validation and parameterized SQL.
- Rate limiting and secure headers.
- CORS restricted to configured origins.
- Secrets only in environment/secret manager.
- Sensitive actions audited.
- Minimal personal data in logs.
- File type/size checks and safe CSV parsing.
- Backup and restore documentation.
- Production call recording disabled unless provider capability, consent/notice, access, and retention policies are explicitly approved.

---

# PART G — PROJECT STRUCTURE AND DEVELOPER EXPERIENCE

## 25. Required repository structure

Adapt to the existing workspace if one is already established; do not duplicate it.

```text
CRM-SaaS/
  Backend/                         # Deploy independently: Laravel REST API
    app/
      Http/Controllers/Api/V1/
      Http/Requests/
      Http/Resources/
      Middleware/
      Models/
      Policies/
      Services/
      Actions/                      # only where useful
    bootstrap/
    config/
    database/
      migrations/
      seeders/
      factories/
    routes/
      api.php
      web.php                       # only Laravel-required web/auth routes
    tests/Feature/
    tests/Unit/
    storage/
    .env.example
    composer.json
  Frontend/                        # Deploy independently: static React app/PWA
    public/
    src/
      app/
      components/
      features/
        auth/
        dashboard/
        leads/
        calls/
        follow-ups/
        tasks/
        users/
        teams/
        reports/
        notifications/
        settings/
        access-control/
      layouts/
      routes/
      lib/
        api/
        auth/
        permissions/
      styles/
      types/
    .env.example
    package.json
    vite.config.ts
  docs/
    architecture.md
    api.md
    permissions-matrix.md
    setup.md
    deployment.md
    demo-script.md
  README.md
```


This is a suggested structure, not a reason to restructure an existing working project without need.

## 26. Developer commands and setup

Provide documented commands for:
- Installing dependencies
- Starting frontend development server
- Starting the Laravel backend with `php artisan serve` (or the configured local web server)
- Configuring MySQL in `Backend/.env`
- Running `php artisan migrate` and `php artisan db:seed` from `Backend/`
- Building the frontend with `npm run build` from `Frontend/`
- Running type checks, lint, and tests
- Building the production frontend (`npm run build`) and preparing Laravel backend (`composer install --no-dev --optimize-autoloader`, cache/config checks as documented)
- Running any optional Laravel queue worker (`php artisan queue:work`)
- Running Docker Compose if supplied

Create `.env.example` with clearly named placeholders such as:
**Backend/.env.example**: `APP_NAME`, `APP_ENV`, `APP_KEY` placeholder/instructions, `APP_DEBUG`, `APP_URL`, `FRONTEND_URL`, `SANCTUM_STATEFUL_DOMAINS` if Sanctum cookie auth is selected, `SESSION_DOMAIN` if needed, `DB_CONNECTION=mysql`, `DB_HOST`, `DB_PORT`, `DB_DATABASE`, `DB_USERNAME`, `DB_PASSWORD`, mail placeholders only if reset emails are implemented, and queue settings only if queues are enabled.

**Frontend/.env.example**: `VITE_API_BASE_URL` pointing to the deployed Laravel API. Never put secrets in Vite variables because they are exposed to the browser.

Do not expose actual credentials. Ensure `.env` is gitignored.

---

# PART H — IMPLEMENTATION PLAN FOR A SAME-DAY MVP

## 27. Prioritized build sequence

### Stage 0 — Workspace inspection and setup
- Inspect the repository and preserve existing setup.
- Confirm frontend/backend locations and Tailwind configuration.
- Write the implementation checklist.
- Establish environment configuration and database migration approach.

**Exit condition:** Project structure and run commands are known.

### Stage 1 — Visual foundation and app shell
- Set purple-pink theme tokens.
- Build reusable buttons, inputs, badges, cards, modals, tables, alerts, loading states.
- Create desktop sidebar, header, responsive navigation, and role-aware layout.

**Exit condition:** The app has a consistent, responsive CRM shell.

### Stage 2 — Authentication and tenant foundation
- Implement sign-in/session handling and protected routes.
- Create tenant-aware user membership and role policy.
- Seed demo tenant and demo users.
- Enforce backend permission and tenant middleware.

**Exit condition:** Different demo roles can sign in and see only authorized screens/data.

### Stage 3 — Leads and teams
- Create lead list, filters, pagination, create/edit form, detail view, notes/timeline.
- Implement team/user management and lead assignment.
- Implement duplicate warnings.
- Add audit events for important mutations.

**Exit condition:** A manager/admin can create and assign a lead, and the telecaller sees only assigned/permitted leads.

### Stage 4 — Telecaller calling and follow-ups
- Build mobile-first PWA work queue.
- Add `tel:` call initiation.
- Implement manual call outcome, notes, follow-up creation, task lists, due/upcoming/overdue states.
- Update lead timeline safely after actions.

**Exit condition:** Telecaller completes the main daily workflow on a mobile viewport.

### Stage 5 — CSV, notifications, and basic reports
- Implement CSV validation/import and scoped export as feasible.
- Add in-app notifications.
- Implement basic role-scoped dashboards and report filters.
- Add queue/worker only if asynchronous jobs are required.

**Exit condition:** Demo dashboards reflect seeded and newly created records.

### Stage 6 — QA and demo readiness
- Run build/type checks and fix errors.
- Test tenant isolation and role permissions.
- Test lead creation/assignment/call outcome/follow-up.
- Test empty, loading, validation, unauthorized, and error states.
- Verify mobile responsiveness.
- Prepare demo script and known limitations.

**Exit condition:** The demo follows a complete end-to-end workflow without misleading placeholder behavior.

### Priority if time is extremely limited
1. App shell + theme
2. Login/demo users + role-specific routes
3. Tenant-safe lead list/create/detail/assignment
4. Telecaller call outcome + follow-up
5. Seeded dashboards
6. Basic tests and demo documentation
7. CSV/report/notification enhancements according to remaining time

Do not sacrifice tenant isolation or authentication safety to add more screens.

---

# PART I — ACCEPTANCE CRITERIA

## 28. MVP must pass these checks

### Authentication and access
- [ ] Owner, Admin, Manager, and Telecaller can sign in using seeded demo accounts.
- [ ] Each role lands on the correct dashboard.
- [ ] Unauthorized UI routes are blocked.
- [ ] Backend APIs independently enforce permissions using Laravel policies/gates and database-backed permission checks.
- [ ] A Tenant A user cannot access Tenant B data by changing an ID or request parameter.
- [ ] Telecaller cannot export company-wide leads or reassign leads by default.

### Dynamic permissions
- [ ] Permissions are seeded in MySQL and are not implemented solely as hard-coded frontend role checks.
- [ ] An authorised Owner/Admin can change role permissions in the UI without code changes.
- [ ] Permission changes are enforced by Laravel APIs on the next request and are audited.
- [ ] Users cannot grant permissions above their own authority or bypass tenant/team/record scope.
- [ ] UI navigation/actions match effective permissions, while API enforcement remains authoritative.

### Leads
- [ ] Authorized user can create, search, filter, paginate, open, and update a lead.
- [ ] Lead ownership and assignment history are retained.
- [ ] Duplicate phone/email is flagged according to the defined normalization policy.
- [ ] Notes, call outcomes, and assignment events appear in the timeline.
- [ ] Empty and error states are clear.

### Calls and follow-ups
- [ ] Telecaller can initiate a device call from mobile.
- [ ] Telecaller can save a manual call outcome and note.
- [ ] Follow-up Required prompts for a due date/time.
- [ ] Follow-ups appear in due/upcoming/overdue lists.
- [ ] Failed saves are shown as failures; the UI never falsely reports an unsaved activity as saved.

### Teams and reports
- [ ] Owner sees company-scoped metrics.
- [ ] Manager sees only permitted team metrics.
- [ ] Telecaller sees permitted own performance/activity.
- [ ] Metrics use documented definitions and real database data.

### CSV and audit
- [ ] Import validation errors are visible.
- [ ] Exports are permission-controlled and tenant-scoped.
- [ ] Sensitive operations generate audit events.

### Technical readiness
- [ ] Frontend production build succeeds.
- [ ] Laravel dependency/autoload checks and automated tests pass; no PHP syntax errors.
- [ ] Database migrations and seed scripts work from documented instructions.
- [ ] No secrets are committed.
- [ ] Main screens are responsive.
- [ ] Setup and demo instructions are complete.
- [ ] Known incomplete features are documented honestly.

---

# PART J — MANAGER DEMO SCRIPT

## 29. Suggested 5–8 minute demo

1. **Show the product shell:** Explain that the product is a multi-tenant CRM SaaS with a shared codebase and isolated company workspaces.
2. **Sign in as Owner:** Show company-level dashboard, lead KPIs, follow-up backlog, and team overview.
3. **Sign in as Admin/Manager:** Show role-specific access, lead assignment, and team-scoped data.
4. **Create or open a lead:** Demonstrate search, filters, lead details, and duplicate warning if demo data supports it.
5. **Assign the lead:** Assign it to a telecaller and show assignment history/notification if implemented.
6. **Sign in as Telecaller:** Open the mobile PWA layout, open the assigned lead, use call initiation, save a manual outcome and note.
7. **Schedule a follow-up:** Show the follow-up appearing in the task list and lead timeline.
8. **Return to dashboard:** Show updated counts/reports based on actual data.
9. **Briefly explain scalability/security:** Tenant isolation, server-side role enforcement, MySQL indexing, modular monolith, and optional Laravel queues/Redis only where appropriate.
10. **Be transparent:** State which features are implemented, which are basic, and which are deferred.

Do not demo fake telephony integration, fake notification delivery, or hard-coded KPI values as if they were production features.

---

# PART K — REQUIRED FINAL OUTPUT FROM ANTIGRAVITY

## 30. When implementation is complete, report

1. What was implemented.
2. Current repository structure and important files.
3. Exact commands to install, configure, run, seed, test, and build.
4. Database migration/seed status.
5. Demo accounts and passwords for local demonstration only.
6. Which role-based flows were verified.
7. Build/type-check/test results.
8. Which acceptance criteria passed and failed.
9. Any incomplete feature, limitation, or manual configuration.
10. Any security issue or blocker that prevents production use.
11. A concise next-step plan after the manager demo.

Do not state “production ready” merely because the development server starts. Distinguish “demo-ready MVP” from “production-ready SaaS”.

## Final instruction

Start by inspecting the current workspace and SRS-related files available in the project. Then implement the MVP in the existing workspace, following this document as the source of truth. Preserve the existing features and purple-pink theme exactly; use separate `Frontend/` React and `Backend/` Laravel folders so each can be deployed independently. Keep the four operational roles and defer the Super Admin dashboard. Implement dynamic database-backed permissions with safe company-admin controls. Use Salesforce only for general CRM principles such as unified customer history, shared context, actionable workflows, customisation, and reporting—not for copying its design or expanding this MVP into unrelated product lines. Reference: https://www.salesforce.com/in/crm/what-is-crm/ Optimize for a coherent, secure, end-to-end demo while laying a maintainable foundation for future SaaS scale.


---

# PART L — SEPARATE DEPLOYMENT CHECKLIST

## 31. Frontend and backend must deploy independently

- `Frontend/` builds to static assets and can be hosted on a static hosting provider/CDN. Configure `VITE_API_BASE_URL` at build time to the deployed Laravel API URL.
- `Backend/` is a Laravel application deployed to a PHP-compatible host/container with the required PHP extensions and Composer dependencies. Configure its own environment variables and MySQL connection.
- MySQL must be reachable from the backend environment only; never expose database credentials or direct database access to the browser.
- Configure Laravel CORS for the exact production frontend origin(s), not `*` with credentials. If using Sanctum SPA cookie authentication, configure stateful domains, session cookie domain, HTTPS, credentials, and CSRF bootstrap correctly for the chosen domain topology. If using a different auth flow, document the security trade-offs and token storage strategy.
- Configure HTTPS for both deployments. Confirm frontend-to-backend API connectivity, preflight requests, login/logout, session expiry, and permission refresh in production-like testing.
- Run migrations as a deliberate deployment step after backup/approval; do not automatically run destructive database resets in production.
- Frontend rollback and backend rollback must be possible independently, while keeping API contract compatibility during deployment.
- Document exact build/deploy commands, health-check endpoint, logs, database backup/restore, and environment variables in `docs/deployment.md`.

## 32. Final architecture guardrails

- Do not move business logic into the React frontend.
- Do not connect React directly to MySQL. All data access goes through Laravel REST APIs.
- Do not put private credentials in `VITE_*` variables.
- Do not hard-code permission assignments in JSX as the source of truth. Fetch the effective permission set from the authenticated API and enforce it again on the backend.
- Do not change the approved theme, layout pattern, feature scope, role names, or MVP workflows unless the user explicitly approves a later change.
- Do not build a Super Admin panel, subscription billing, advanced marketing automation, AI agents, or other deferred features.
