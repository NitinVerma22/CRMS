# CRM SaaS MVP — Interactive Demo Walkthrough Script (5–8 Minutes)

This script provides an executive walkthrough of the **CRM SaaS MVP**, demonstrating real database-backed workflows across **Owner**, **Admin**, **Manager**, and **Telecaller** roles.

---

## Preparation
- Ensure Backend server is running: `http://localhost:8000`
- Ensure Frontend dev server is running: `http://localhost:5173`
- Open the login page: `http://localhost:5173/login`
- Default password for all demo accounts: `Password123!`

---

## Act 1: Owner Experience — Full Executive Control (1.5 Minutes)

1. **Sign In**: On the login page, click the quick-login badge **"Owner"** (or enter `owner@apexdynamics.com`).
2. **Review Executive Dashboard**:
   - Observe real-time KPI cards: **Total Leads (9)**, **Calls Today**, **Overdue Follow-ups**, and **Conversion Rate**.
   - Review the **Monthly Call Volume** and **Pipeline Stage Breakdown** distribution charts.
   - Note the **Quick Action Buttons**: "New Lead", "View All Calls", "Access Control".
3. **Inspect Lead Database**:
   - Click **Leads** in the sidebar.
   - Notice that the Owner sees all 9 company leads across all teams.
   - Test search and status filter (`Contacted`, `Qualified`, `Interested`).
   - Click on **"Aarav Mehta"** to inspect the 360° Lead Detail view (Activities timeline, Notes, Call History, and Follow-up tasks).
4. **Live RBAC Matrix Demonstration**:
   - Click **Access Control** in the sidebar.
   - Point out the 4 operational roles and 12 modules containing all 30 granular permissions.
   - Find `leads.export` for the **Manager** role.
   - Click the checkbox to toggle it on or off. Observe the live database update confirmation alert without page reload.

---

## Act 2: Admin Experience — User & Permission Overrides (1.5 Minutes)

1. **Switch User**: Click the user profile dropdown in the top right corner and click **Logout**.
2. **Sign In as Admin**: Click the quick-login badge **"Admin"** (or enter `admin@apexdynamics.com`).
3. **User Management**:
   - Click **Users** in the sidebar.
   - Review the company directory: 5 active users with their respective roles and assigned teams.
4. **Demonstrate User-Level Permission Override**:
   - Find telecaller **"Ananya Patel"** (`telecaller@apexdynamics.com`).
   - Click the **Permissions** button next to her name.
   - In the modal, select `leads.export` (Export Leads).
   - Set Override Type to **"Explicit Grant"**, add reason: *"Authorized for quarterly marketing list export"*, and click **Save Override**.
   - Notice the override is saved with an explicit green badge, showing how user overrides supersede role baselines.
5. **Inspect Immutable Audit Trail**:
   - Click **Audit Logs** in the sidebar.
   - Show the full immutable log recording every login, lead creation, call log, and permission change with timestamp and actor IP.

---

## Act 3: Manager Experience — Team Queue & Assignment (1.5 Minutes)

1. **Switch User**: Logout and click the quick-login badge **"Manager"** (`manager@apexdynamics.com`).
2. **Supervisory Dashboard**:
   - Note the manager-specific widgets: **Team Leads**, **Team Calls Today**, **Pending Follow-ups**, and **Telecaller Leaderboard**.
   - The leaderboard ranks team members by completed calls and conversions.
3. **Team Lead Queue & Lead Assignment**:
   - Click **Leads** in the sidebar.
   - Notice that the Manager only views leads belonging to their managed team (**West Zone Telesales**), demonstrating data scoping.
   - Find the unassigned lead **"Deepak Joshi"**.
   - Click the **Assign** button.
   - Select **"Ananya Patel"** from the telecaller dropdown and confirm.
   - Notice the status immediately changes to assigned and an audit activity is appended.

---

## Act 4: Telecaller Experience — The High-Velocity Call Desk (2 Minutes)

1. **Switch User**: Logout and click the quick-login badge **"Telecaller"** (`telecaller@apexdynamics.com`).
2. **Personalized Touch-Optimized Work Queue**:
   - Notice the telecaller lands directly on the **Telecaller Home** queue (`/telecaller/home`), designed specifically for high-velocity dialing.
   - Observe the **"Up Next"** featured lead card displaying the lead's name, company, phone number, and scheduled follow-up urgency.
3. **One-Tap Calling (`tel:`)**:
   - Click the prominent green **"Call Now"** button (`tel:+919820011223`).
   - On a mobile device, this initiates native phone dialing; on desktop, it prompts the default calling client.
4. **Log Standardized Call Outcome (SRS Aligned)**:
   - Click **"Log Outcome"** on the lead card.
   - In the modal, select one of the 12 standardized CRM outcomes:
     - Select **"Interested"**.
     - Set Call Duration: `180` seconds.
     - Add Call Note: *"Client expressed strong interest in Enterprise SaaS plan. Requested quote for 25 seats."*
     - Check **"Schedule Follow-up"**: Select tomorrow at 11:00 AM.
     - Note: *"Send product brochure and pricing matrix before the call."*
     - Click **Save Call Outcome**.
5. **Verify Immediate Updates**:
   - Notice the lead card automatically transitions and updates the queue counter.
   - Click **Follow-ups** in the sidebar: The new follow-up task is listed with an **"Open"** badge.
   - Click **Calls** in the sidebar: The new 180s call log is recorded in the agent's history.

---

## Act 5: Multi-Tenant Isolation Proof (1 Minute)

1. **Switch to Tenant B**:
   - Logout from Apex Dynamics.
   - Sign in with Tenant B Owner credentials:
     - Email: `owner@horizon.com`
     - Password: `Password123!`
2. **Verify Strict Tenant Isolation**:
   - Review Dashboard and Lead list: Notice **Horizon Telecom** only sees its own isolated leads (`Rajesh Verma`, `Sunita Rao`).
   - None of Apex Dynamics' 9 leads, teams, or call logs are visible or accessible.
   - Every database query strictly filters by `tenant_id`, guaranteeing cross-tenant data privacy.

---

## Summary of Completed Acceptance Criteria

- [x] Multi-tenant data isolation verified via automated tests & UI.
- [x] 4 operational roles (Owner, Admin, Manager, Telecaller) with role-tailored dashboards.
- [x] Dynamic database-driven RBAC matrix with 30 permissions and user overrides.
- [x] High-velocity telecaller queue with one-tap calling (`tel:`) and 12 call outcomes.
- [x] Duplicate phone number detection warning.
- [x] Full audit trail recording all administrative and calling actions.
- [x] Cohesive purple-pink gradient UI theme across desktop and mobile layouts.
