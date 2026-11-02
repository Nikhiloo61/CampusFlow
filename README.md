# CampusFlow — Smart Campus Service & Resource Management System

> **Tagline:** Your Campus, In Sync  
> **Architecture:** Modern Production-Grade MERN Full-Stack Web Application  
> **Frontend:** React 18, Vite, React Router 6, Axios, Recharts, Lucide React, Custom CSS Design System  
> **Backend:** Node.js, Express.js, RESTful Architecture, JWT, bcryptjs, MongoDB, Mongoose ODM  
> **Testing:** Vitest with Supertest (25/25 Integration & Unit Tests Passed)

---

## 1. System Overview

**CampusFlow** is a modern campus operations SaaS platform engineered to unify university physical plant facilities, IT maintenance, academic resource reservations, and administrative oversight under a single synchronized system.

### Key Capabilities

1. **Role-Based Access Control (RBAC):** Distinct dashboards, capabilities, and data access policies for **Students**, **Staff**, and **Administrators**.
2. **Service Request Management:** Complete lifecycle from submission to resolution with unique reference numbering (`REQ-2026-XXXX`), attachments, category mapping, and location indexing.
3. **FlowPriority™ Engine:** A deterministic, explainable, rule-based request prioritization engine that scores tickets from 0 to 100 based on urgency, category baseline impact, location sensitivity, and SLA escalation. Includes administrative override with mandatory audit logging.
4. **Campus Resource Directory & Booking:** Reservation system for computer laboratories, tiered auditoriums, conference rooms, smart projectors, and makerspace equipment.
5. **Mandatory Booking Conflict Prevention:** Database-level concurrency control that mathematically prevents overlapping bookings (`startTime < existing.endTime && endTime > existing.startTime`) and blocks simultaneous collision attempts with HTTP 409 status responses.
6. **Immutable Timelines & Audit Logs:** Chronological timeline events for every ticket transition and an administrative audit trail tracking critical overrides.
7. **In-App Notifications:** Real-time alerts with unread counter badges for ticket state changes, staff assignments, and booking approvals.
8. **Operations Analytics & Reports:** Interactive Recharts charts (weekly ticket flows, category distribution, staff workload balance) and CSV report exports.

---

## 2. Default Development Demo Accounts

The database is pre-seeded with realistic demo accounts across all three user roles:

| Role | Email | Password | Name / Department |
| :--- | :--- | :--- | :--- |
| **Administrator** | `admin@campusflow.edu` | `Admin@123` | Dr. Victor Vance *(Campus Operations & Infrastructure)* |
| **Staff Member** | `staff.sarah@campusflow.edu` | `Staff@123` | Sarah Jenkins *(Facilities & Maintenance)* |
| **Staff Member** | `staff.alex@campusflow.edu` | `Staff@123` | Alex Rivera *(IT & Network Systems)* |
| **Staff Member** | `staff.marcus@campusflow.edu` | `Staff@123` | Marcus Thorne *(Campus Safety & Physical Plant)* |
| **Student** | `student.emma@campusflow.edu` | `Student@123` | Emma Watson *(Computer Science & Engineering)* |
| **Student** | `student.liam@campusflow.edu` | `Student@123` | Liam Chen *(Mechanical Engineering)* |
| **Student** | `student.sophia@campusflow.edu` | `Student@123` | Sophia Davis *(School of Business)* |

*Note: Quick-fill buttons are provided on the Login page for instant one-click authentication.*

---

## 3. FlowPriority™ Scoring Algorithm

The FlowPriority™ engine evaluates four quantifiable operational variables to determine an incident's triage priority score:

$$\text{FlowPriority Score} = \min(100, \max(0, W_{\text{urgency}} + W_{\text{category}} + W_{\text{location}} + W_{\text{escalation}}))$$

### Factor Breakdown:
1. **Urgency Base Weight ($W_{\text{urgency}}$, max 40 pts):**
   - `Emergency`: +40 points
   - `High`: +28 points
   - `Medium`: +16 points
   - `Low`: +6 points

2. **Category Operational Impact ($W_{\text{category}}$, max 30 pts):**
   - Configurable per category in database (Safety & Security: +30, IT & Network: +24, Electrical & Power: +22, Plumbing & Water: +18, Academic Equipment: +15, Facility & Furniture: +12, Cleaning: +8).

3. **Facility & Location Criticality ($W_{\text{location}}$, max 15 pts):**
   - High-impact facilities (Server Room, Data Center, Examination Hall, Main Auditorium, Central Wet Lab): +15 points
   - Academic facilities (Classroom, Lecture Hall, Computer Lab, Library): +10 points
   - Standard campus areas (Dormitory, Student Lounge, Cafeteria): +5 points

4. **Time-Elapsed Escalation ($W_{\text{escalation}}$, max 15 pts):**
   - Unresolved $>12\text{h}$: +3 pts | $>24\text{h}$: +6 pts | $>48\text{h}$: +10 pts
   - Target SLA breached: +8 pts

### Priority Level Classification:
- **Critical:** 80 – 100
- **High:** 60 – 79
- **Medium:** 35 – 59
- **Low:** 0 – 34

---

## 4. Conflict-Free Resource Booking Architecture

Booking conflicts are prevented at the database query layer:

1. **Overlap Query Check:**
   When booking slot $[T_{\text{start}}, T_{\text{end}})$ is requested for resource $R$:
   $$\text{Conflict} \iff \exists B \in \text{Bookings}(R) \text{ where } B.\text{status} \in \{\text{'Approved'}, \text{'Pending'}\} \land (T_{\text{start}} < B.\text{endTime} \land T_{\text{end}} > B.\text{startTime})$$
2. **Concurrency Verification:**
   Immediately following document insertion, a secondary collision check guarantees race conditions between concurrent requests rollback the competing attempt with HTTP 409 Conflict.
3. **Approval Cascade:**
   When an administrator approves a booking, all other overlapping pending requests are automatically declined with recorded explanations and notifications sent to the affected students.

---

## 5. API Reference Endpoints

### Authentication & Profiles (`/api/auth`)
- `POST /api/auth/register` — Register student or staff account
- `POST /api/auth/login` — Validate credentials & issue JWT token
- `GET /api/auth/me` — Retrieve active authenticated session
- `PUT /api/auth/profile` — Update name, phone, department
- `PUT /api/auth/change-password` — Securely update user password

### Service Requests (`/api/requests`)
- `POST /api/requests` — Create request with automated FlowPriority calculation
- `GET /api/requests` — Paginated list with search, status, category, priority filters
- `GET /api/requests/:id` — Request details with complete chronological timeline
- `PUT /api/requests/:id` — Edit eligible details (creator or admin)
- `POST /api/requests/:id/notes` — Add progress note to timeline
- `PUT /api/requests/:id/assign` — Assign staff member (Staff/Admin)
- `PUT /api/requests/:id/status` — Update lifecycle state (Assigned/In Progress/Resolved/Rejected)
- `PUT /api/requests/:id/override-priority` — Admin manual priority override with audit log

### Campus Resources (`/api/resources`)
- `GET /api/resources` — List resources with category/search filters
- `GET /api/resources/:id` — Single resource specification
- `POST /api/resources` — Create new resource (Admin only)
- `PUT /api/resources/:id` — Edit resource details & capacity (Admin only)

### Resource Bookings (`/api/bookings`)
- `GET /api/bookings/check-availability` — Pre-submission conflict query
- `POST /api/bookings` — Submit reservation with concurrency protection
- `GET /api/bookings` — User bookings or campus-wide schedule
- `PUT /api/bookings/:id/approve` — Approve pending booking (Admin only)
- `PUT /api/bookings/:id/reject` — Decline booking with reason (Admin only)
- `PUT /api/bookings/:id/cancel` — Cancel eligible booking (Owner/Admin)

### Dashboards & Reports
- `GET /api/dashboard/student` — Personal stats & live Campus Pulse
- `GET /api/dashboard/staff` — Assigned workload & priority breakdown
- `GET /api/dashboard/admin` — Operations summary & Recharts datasets
- `GET /api/reports/operations` — Filtered operational metrics & CSV export data
- `GET /api/audit` — System audit logs (Admin only)

---

## 6. Installation & Execution Guide

### Prerequisites
- Node.js (v18+ or v24+)
- MongoDB running locally on port 27017 (or MongoDB Atlas connection string)

### 1. Install Dependencies
```powershell
# From CampusFlow root directory
cd c:\TT\CampusFlow

# Install server dependencies
cd server
npm install

# Install client dependencies
cd ..\client
npm install
```

### 2. Seed Database
```powershell
cd c:\TT\CampusFlow\server
node seed/seedData.js
```

### 3. Run Automated Tests
```powershell
cd c:\TT\CampusFlow\server
npm test
```
*Executes all 25 Vitest integration tests verifying auth, priority calculation, conflict prevention, RBAC, and timelines.*

### 4. Start Application

**Terminal 1 (Backend API Server):**
```powershell
cd c:\TT\CampusFlow\server
node server.js
# Listening on http://localhost:5000
```

**Terminal 2 (Frontend Client Application):**
```powershell
cd c:\TT\CampusFlow\client
npm run dev -- --host
# Listening on http://localhost:5173
```

Access the application in your browser at: **`http://localhost:5173`**
