# CRM SaaS MVP — Role-Based Access Control (RBAC) & Permissions Matrix

This document provides the canonical specification of the database-driven, configurable Role-Based Access Control system implemented in the CRM SaaS MVP.

---

## 1. Core Principles & Evaluation Precedence

The permission evaluation engine uses strict, deterministic precedence:

```
┌────────────────────────────────────────────────────────┐
│                   User Action Request                  │
└───────────────────────────┬────────────────────────────┘
                            │
                            ▼
           Does a User Override Exist for this User?
           ┌────────────────┴────────────────┐
          YES                               NO
           │                                 │
           ▼                                 ▼
   Is Override = DENY?              Does User's Role have
   ┌───────┴───────┐                the permission enabled
  YES              NO               in the database?
   │                │                        │
   ▼                ▼                        ▼
DENIED (403)   GRANTED (200)           YES ──► GRANTED (200)
                                       NO  ──► DENIED (403)
```

### Precedence Hierarchy:
1. **Explicit User Deny (`is_granted = false`)**: Evaluated first. Always wins, even if the user's role grants the permission.
2. **Explicit User Grant (`is_granted = true`)**: Evaluated second. Grants permission even if the role lacks it.
3. **Role-Based Permission (`role_permissions` table)**: Default baseline dynamically stored and toggled per tenant role.
4. **Tenant Isolation**: Independent of permissions. A user can **never** access or mutate data belonging to another tenant (`tenant_id` mismatch always yields `404 Not Found`).

---

## 2. Operational Roles

| Role | Slug | Hierarchy Level | Primary Scope |
| :--- | :--- | :--- | :--- |
| **Owner** | `owner` | 100 | Complete tenant authority: organization metrics, teams, role matrices, settings, full lead database. |
| **Admin** | `admin` | 80 | Operational management: users, user overrides, lead bulk imports, audit logs, system configurations. |
| **Manager** | `manager` | 50 | Supervisory scope: managed team queue, lead re-distribution, team calling reports, agent conversion leaderboard. |
| **Telecaller** | `telecaller` | 20 | Daily execution scope: personal work queue, one-tap calling (`tel:`), call outcome logging, scheduled follow-ups. |

---

## 3. Master Permission Catalogue (30 Permissions Across 12 Modules)

| Module | Permission Key | Display Name | Description |
| :--- | :--- | :--- | :--- |
| **Dashboard** | `dashboard.view` | View Dashboard | Access role-tailored dashboard and analytical widgets |
| **Leads** | `leads.view_all` | View All Leads | View all company leads across all teams |
| | `leads.view_team` | View Team Leads | View leads assigned to user's managed team |
| | `leads.view_assigned` | View Assigned Leads | View leads directly assigned to the user |
| | `leads.create` | Create Lead | Add a new lead to the CRM database |
| | `leads.edit` | Edit Lead | Update lead contact info, requirements, and status |
| | `leads.delete` | Delete Lead | Soft-delete a lead record |
| | `leads.assign` | Assign Leads | Assign or re-assign leads to telecallers |
| | `leads.export` | Export Leads | Export leads to CSV file |
| | `leads.import` | Import Leads | Bulk import leads from CSV file |
| **Calls** | `calls.view_all` | View All Calls | View organization-wide call history logs |
| | `calls.view_team` | View Team Calls | View call logs for managed team members |
| | `calls.view_assigned` | View Assigned Calls | View personal call logs |
| | `calls.log` | Log Call | Record a call outcome, notes, and duration |
| **Follow-Ups**| `followups.view_all` | View All Follow-ups | View organization-wide scheduled follow-ups |
| | `followups.view_team` | View Team Follow-ups| View scheduled follow-ups for team members |
| | `followups.view_assigned`| View Assigned Follow-ups| View personal follow-up tasks |
| | `followups.manage` | Manage Follow-ups | Create, complete, reschedule, or cancel follow-ups |
| **Teams** | `teams.view` | View Teams | View company teams and rosters |
| | `teams.manage` | Manage Teams | Create teams, assign managers, and update rosters |
| **Users** | `users.view` | View Users | View company users list and profiles |
| | `users.manage` | Manage Users | Create users, change roles, activate/deactivate |
| **Access Control** | `access_control.view` | View Permissions | View role permission matrices and user overrides |
| | `access_control.manage` | Manage Permissions | Dynamically toggle role permissions and apply overrides |
| **Reports** | `reports.view` | View Reports | Access conversion analytics and call performance metrics |
| **Settings** | `settings.view` | View Settings | View dynamic CRM statuses, sources, and outcomes |
| | `settings.manage` | Manage Settings | Update tenant configurations and workflow lists |
| **Audit** | `audit.view` | View Audit Trail | View immutable security audit logs |
| **Notifications** | `notifications.view` | View Notifications | Receive and view in-app alerts and notifications |
| | `notifications.manage` | Manage Notifications | Mark notifications as read or clear alerts |

---

## 4. Default Baseline Role Permissions Matrix

The table below details the baseline configuration populated by `PermissionSeeder.php`:

| Permission Key | Owner | Admin | Manager | Telecaller |
| :--- | :---: | :---: | :---: | :---: |
| `dashboard.view` | **Yes** | **Yes** | **Yes** | **Yes** |
| `leads.view_all` | **Yes** | **Yes** | No | No |
| `leads.view_team` | **Yes** | **Yes** | **Yes** | No |
| `leads.view_assigned` | **Yes** | **Yes** | **Yes** | **Yes** |
| `leads.create` | **Yes** | **Yes** | **Yes** | **Yes** |
| `leads.edit` | **Yes** | **Yes** | **Yes** | **Yes** |
| `leads.delete` | **Yes** | **Yes** | No | No |
| `leads.assign` | **Yes** | **Yes** | **Yes** | No |
| `leads.export` | **Yes** | **Yes** | No | No |
| `leads.import` | **Yes** | **Yes** | **Yes** | No |
| `calls.view_all` | **Yes** | **Yes** | No | No |
| `calls.view_team` | **Yes** | **Yes** | **Yes** | No |
| `calls.view_assigned` | **Yes** | **Yes** | **Yes** | **Yes** |
| `calls.log` | **Yes** | **Yes** | **Yes** | **Yes** |
| `followups.view_all` | **Yes** | **Yes** | No | No |
| `followups.view_team` | **Yes** | **Yes** | **Yes** | No |
| `followups.view_assigned` | **Yes** | **Yes** | **Yes** | **Yes** |
| `followups.manage` | **Yes** | **Yes** | **Yes** | **Yes** |
| `teams.view` | **Yes** | **Yes** | **Yes** | No |
| `teams.manage` | **Yes** | **Yes** | No | No |
| `users.view` | **Yes** | **Yes** | **Yes** | No |
| `users.manage` | **Yes** | **Yes** | No | No |
| `access_control.view` | **Yes** | **Yes** | No | No |
| `access_control.manage` | **Yes** | **Yes** | No | No |
| `reports.view` | **Yes** | **Yes** | **Yes** | No |
| `settings.view` | **Yes** | **Yes** | No | No |
| `settings.manage` | **Yes** | **Yes** | No | No |
| `audit.view` | **Yes** | **Yes** | No | No |
| `notifications.view` | **Yes** | **Yes** | **Yes** | **Yes** |
| `notifications.manage` | **Yes** | **Yes** | **Yes** | **Yes** |

---

## 5. User-Level Overrides

When an organization needs to grant a specific telecaller elevated capabilities (e.g. allowing a senior telecaller to export leads) or restrict an administrator (e.g. revoking export privileges for security compliance), an authorized user can record an override in the `user_permission_overrides` table.

### API Endpoints
- `GET /api/v1/access-control/users/{user}/overrides` — Retrieve all active overrides for a user.
- `POST /api/v1/access-control/users/{user}/overrides` — Create or update an override (`permission_id`, `is_granted`, `reason`).
- `DELETE /api/v1/access-control/users/{user}/overrides/{permission}` — Remove an override, falling back to role baseline.

All changes are audited and immediately reflect in the user's `effective_permissions` array returned at `/api/v1/auth/me`.
