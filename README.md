# AcxiomCRM - Enterprise CRM System
**Prepared according to Acxiom Consulting CRM Project Documentation**

---

## 1. Project Overview & Scope
AcxiomCRM is a role-based Customer Relationship Management (CRM) web application designed for enterprise sales operations. It implements the complete customer-sales lifecycle from prospect capture to deal closure, comprehensive multi-level validation, ASP.NET Core Identity authentication and lockout security, audit logging, interactive Chart.js analytics, REST APIs, and exportable business reports.

---

## 2. Default Seeded Credentials

The database automatically initializes upon application startup with predefined roles and users:

| Role | Username / Email | Password | Access Scope |
| :--- | :--- | :--- | :--- |
| **Admin** | `admin@acxiomcrm.com` | `Admin@123` | Full application administration, user/role management, security unlock, immutable audit log, all CRM records and reports. |
| **Manager** | `manager@acxiomcrm.com` | `Manager@123` | Team pipeline monitoring, all CRM customer/deal records, management and productivity reports. |
| **Sales Executive** | `sales@acxiomcrm.com` | `Sales@123` | Assigned customers, leads, opportunities, follow-ups, and individual sales dashboard. |
| **Sales Executive** | `sales2@acxiomcrm.com` | `Sales@123` | Assigned customer and deal records. |

> **Password Complexity Standard:** Minimum 8 characters, at least 1 uppercase letter, 1 lowercase letter, 1 digit, and 1 non-alphanumeric special character.  
> **Lockout Policy:** 5 consecutive failed login attempts trigger an automatic 15-minute account lockout. Administrators can unlock accounts on demand.

---

## 3. Layered Application Architecture
The solution strictly follows the layered architecture defined in the Acxiom Consulting specifications:

```
                  Users / REST Clients
                          │
                          ▼
                  Presentation Layer
         (ASP.NET Core MVC Controllers, Razor Views,
          Bootstrap 5 UI, jQuery Validation, Chart.js)
                          │
                          ▼
                  Application Layer
         (CrmService, DashboardService, AuditService,
          Data Transfer Objects [DTOs], ViewModels)
                          │
                          ▼
                Domain / Business Layer
         (Customer, Lead, Opportunity, FollowUp,
          Activity, AuditLog, Business Validation Rules)
                          │
                          ▼
                  Data Access Layer
         (ApplicationDbContext, Entity Framework Core 9,
          Repository / Parameterized Query Execution)
                          │
                          ▼
                    Database Layer
                 (SQLite: acxiomcrm.db)
```

**Cross-Cutting Concerns:**
- **Security:** ASP.NET Core Identity, cookie management, anti-forgery (`[ValidateAntiForgeryToken]`), role policies.
- **Audit Logging:** Append-only event tracking for Login, Logout, Failed Logins, Account Lockouts, Record Creations, Updates, Deletions, Role Changes, and Lead Conversions.
- **Input & Business Validations:** Client-side unobtrusive validation + Server-side ModelState and domain validation.

---

## 4. Key Functional Modules & Features

### 4.1 Authentication & Authorization
- ASP.NET Core Identity with hashed passwords (PBKDF2/HMAC-SHA256).
- Account lockout with configurable attempt thresholds and duration.
- Cookie authentication with sliding expiration.
- Anti-forgery token verification on all state-changing POST requests.
- Server-side role authorization (`Admin`, `Manager`, `SalesExecutive`).

### 4.2 Dashboard Analytics & Chart.js Visualizations
- **Role-Scoped KPI Cards:**
  1. Total Customers
  2. Total Leads
  3. Open Leads (Excluding Converted and Lost)
  4. Total Opportunities
  5. Open Opportunities
  6. Won Opportunities
  7. Lost Opportunities
  8. Total Pipeline Value & Weighted Pipeline Value
  9. Planned & Overdue Follow-Up Indicators
- **Interactive Chart.js Visualizations:**
  1. *Lead Status Breakdown* (New, Contacted, Qualified, Lost, Converted) - Doughnut Chart
  2. *Opportunity Pipeline Stages* (Qualification, Proposal, Negotiation, Won, Lost) - Bar Chart
  3. *Monthly Won Revenue Trend* - Line Chart
- **Date Filters:** Today, This Week, This Month, All Time.

### 4.3 Customer Management
- Master data capture: `CustomerCode`, `CustomerName`, `Email`, `Phone`, `CompanyName`, `Address`, `City`, `State`, `Status`, `OwnerId`.
- **Duplicate Prevention:** Server and client validation enforcing uniqueness of customer email and phone number.
- Relationship tracking with linked Opportunities, Follow-Ups, and Activities.

### 4.4 Lead Management & Lead-to-Customer Conversion
- Lead capture: `LeadCode`, `LeadName`, `Email`, `Phone`, `CompanyName`, `Source`, `Status`, `Priority`, `ExpectedValue`, `AssignedTo`.
- Statuses: `New`, `Contacted`, `Qualified`, `Unqualified`, `Converted`, `Lost`.
- **Conversion Workflow:** Qualified leads can be converted into Customer accounts and Opportunity deals in a single workflow. The status updates to `Converted` and is logged in the Audit Trail.

### 4.5 Opportunity Management & Pipeline
- Tracking: `OpportunityName`, `Customer`, `Lead`, `Amount`, `Stage`, `Probability`, `ExpectedCloseDate`, `AssignedTo`.
- Stages: `Qualification`, `Proposal`, `Negotiation`, `Won`, `Lost`.
- **Derived Weighted Pipeline Value:** `Amount * Probability / 100`.
- **Business Validations:**
  - `Amount > 0` required for active sales opportunities (cannot be zero or negative).
  - `Probability` must be between 0 and 100 inclusive.
  - `Expected Close Date` cannot be in the past for active opportunities.

### 4.6 Follow-Up Management
- Tracking: `FollowUpDate`, `Subject`, `FollowUpType` (Call, Meeting, Email, Visit), `Status` (Planned, Completed, Missed, Cancelled), `AssignedTo`.
- **Date Rule:** A new/planned follow-up date cannot be earlier than today.
- Automatic overdue task detection.
- Completion and rescheduling workflows.

### 4.7 Activity Management
- Touchpoint tracking: Call, Meeting, Email, Task.
- Linkable to Customers and Leads with date and description logging.

### 4.8 User & Role Administration (Admin Only)
- User creation with primary role assignment.
- Role changes and active/inactive status toggle.
- **Account Lockout Management:** View failed login counts and unlock locked accounts.
- Administrative password reset enforcing security standards.

### 4.9 System Audit Log
- Append-only, tamper-protected record of all critical operations.
- Captures: `Timestamp`, `UserId`, `UserName`, `Action`, `EntityName`, `RecordId`, `OldValue`, `NewValue`, `Details`, `IpAddress`, `Result`.
- Search and filter by User, Module, Action, and Date range.

### 4.10 Reporting Engine (with CSV Export)
1. **Pipeline Report:** Stage-wise and owner-wise deal value and weighted revenue.
2. **Conversion Report:** Lead conversion velocity, won ratios, and converted deal volume.
3. **Customer Report:** Account master data and status distribution.
4. **Lead Report:** Channel sources and expected value metrics.
5. **Follow-Up Report:** SLA compliance and overdue task tracking.
6. **User Activity Report:** Cross-module productivity per team member.
- One-click CSV export available for all reports.

### 4.11 Secured REST API (Swagger Enabled)
Exposes REST endpoints utilizing DTOs and standard HTTP status codes (200, 201, 400, 401, 403, 404, 409):
- `POST /api/auth/login` - Authenticate API client and receive role claims
- `POST /api/auth/logout` - Session termination
- `GET /api/customers` - List/search customers
- `GET /api/customers/{id}` - Get customer details
- `POST /api/customers` - Create customer (with duplicate conflict checks)
- `PUT /api/customers/{id}` - Update customer
- `DELETE /api/customers/{id}` - Deactivate customer
- `GET /api/leads` - List/search leads
- `POST /api/leads` - Create lead
- `GET /api/opportunities` - List/search opportunities
- `POST /api/opportunities` - Create opportunity with business validations
- `GET /api/followups` - List follow-up activities
- `POST /api/followups` - Create follow-up
- `GET /api/reports/pipeline` - Stage-wise and weighted pipeline intelligence
- Interactive Swagger UI available at: `/swagger`

---

## 5. How to Run the Application

### Prerequisites
- [.NET 9 SDK](https://dotnet.microsoft.com/download) (or .NET 8 SDK / .NET 10 preview)

### Steps to Run
1. Open PowerShell or a terminal inside the project directory:
   ```powershell
   cd AcxiomCRM
   ```
2. Build the application:
   ```powershell
   dotnet build
   ```
3. Run the application:
   ```powershell
   dotnet run
   ```
4. Open your browser and navigate to:
   - Web Application: `http://localhost:5000` (or `https://localhost:5001`)
   - Swagger REST API: `http://localhost:5000/swagger`
5. Log in with any of the seeded credentials (e.g. `admin@acxiomcrm.com` / `Admin@123`).
