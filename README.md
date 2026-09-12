# demo-Student-Portal-for-Odlar-Yurdu-University
# OYU Portal

**Odlar Yurdu University — an information system for students, teachers, university leadership, and IT auditors.**

**Tech stack:** NestJS + TypeScript + Prisma + PostgreSQL (backend) · Vanilla HTML/JS (frontend) · i18n az/ru/en · Deployed on Railway.

```text
oyu-project-railway/
├── oyu-backend/           # NestJS API (TypeScript + Prisma + PostgreSQL)
│   ├── src/
│   ├── prisma/
│   ├── SECURITY.md        ← document for the IT auditor
│   └── README.md
└── oyu-project/           # Frontend (Express + static HTML)
    ├── public/
    │   ├── login.html
    │   ├── portal.html         (student/teacher)
    │   ├── admin-dashboard.html (leadership)
    │   ├── security.html       (IT auditor)
    │   ├── i18n/               (az/ru/en)
    │   └── js/                 (API, i18n client)
    └── server.js
```

---

## Running the Full Stack Locally

Requirements: Node.js 20+, Docker (for PostgreSQL), npm.

### 1. Start the Database

```bash
cd oyu-backend
cp .env.example .env
docker compose up -d postgres
```

### 2. Install Dependencies and Run Migrations

```bash
npm install
npx prisma generate
npm run prisma:migrate      # name the first migration "init"
npm run seed
```

### 3. Start the Backend

```bash
npm run dev
```

API: `http://localhost:3001`, prefix `/api/v1`

### 4. Start the Frontend

In another terminal:

```bash
cd oyu-project
npm install
node server.js
```

Frontend: `http://localhost:3000` → redirects to `/login.html`

---

## Demo Accounts

| Role       | OYU ID       | Password             | Redirect                      |
| ---------- | ------------ | -------------------- | ----------------------------- |
| STUDENT    | `0122184710` | `Demo@Student1`      | `/portal.html`                |
| STUDENT    | `0122193820` | `Demo@Student2`      | `/portal.html`                |
| STUDENT    | `0122197531` | `Demo@Student3`      | `/portal.html`                |
| TEACHER    | `0987654321` | `Demo@Teacher1`      | `/portal.html` (teacher mode) |
| TEACHER    | `0987654322` | `Demo@Teacher2`      | `/portal.html` (teacher mode) |
| LEADERSHIP | `0100000001` | `Demo@Dean!2026`     | `/admin-dashboard.html`       |
| IT_AUDITOR | `0900000001` | `Demo@Audit!Sec2026` | `/security.html`              |

---

## Interface Languages

The language switcher is located in the top-right corner of every screen.

Supported languages:

* 🇦🇿 **Azerbaijani (az)** — default
* 🇷🇺 **Russian (ru)**
* 🇬🇧 **English (en)**

The selected language is stored in `localStorage` and sent to the API through the `X-Lang` header. The server returns error messages in the selected language.

---

## What to Show During the Demo

### Student

1. Dashboard with:

   * GPA
   * today's schedule
   * latest grades
2. Full weekly schedule displayed as a grid
3. Gradebook with teacher comments

### Teacher

1. Dashboard with group statistics
2. Gradebook:

   * select a group
   * view all grades
   * add a new grade through a modal
3. Every grade modification is recorded in `GradeHistory` + `AuditLog`

### Leadership

1. KPI dashboard:

   * total users
   * students
   * teachers
   * active sessions
   * average GPA
   * logins over the last 7 days
2. Grade distribution:

   * excellent
   * good
   * satisfactory
   * unsatisfactory
3. User search
4. Request management:

   * approve
   * reject

### IT Auditor — Key Presentation Feature

1. **Overview**

   * failed logins in the last 24 hours
   * locked accounts
   * audit log status
2. **Configuration**

   * complete security configuration:
   * password policy
   * JWT
   * rate limiting
   * CORS
   * CSP
   * and other security controls
3. **Audit Log**

   * live stream of actions
   * actor
   * action
   * target
   * IP address
4. **Login Log**

   * login attempts
   * success/failure
   * IP address
   * user agent
   * failure reason
5. **Dependencies**

   * critical security dependencies
   * installed versions

---

# Security — Summary

Full documentation → `oyu-backend/SECURITY.md`

Key security controls:

* **Argon2id** password hashing
* **JWT access tokens (15m) + refresh tokens (7d)**
* Refresh tokens stored in **HttpOnly + SameSite=Strict** cookies
* **Refresh token rotation + reuse detection** — an attack triggers revocation of the entire token family
* **Account lockout** after 5 failed login attempts for 15 minutes
* **Constant-time** password verification to protect against timing oracles
* **Rate limiting:** 10 requests/minute on `/auth/login`
* **Helmet + CSP + HSTS** in production
* **CORS whitelist**
* **Zod `.strict()`** to protect against mass assignment
* **Prisma** — SQL injection protection by design
* **AuditLog + LoginLog** for a complete audit trail
* Dedicated **IT_AUDITOR** role with a separate security dashboard

---

# Production Deployment — Railway

### 1. Create a Railway Project

Create two services:

* `oyu-backend` — Dockerfile-based service
* `oyu-frontend` — simple Node.js service running `node server.js`

Add a PostgreSQL database. Railway will automatically provide the `DATABASE_URL`.

### 2. Backend Environment Variables

Set:

```text
JWT_ACCESS_SECRET=<generated secret>
JWT_REFRESH_SECRET=<generated secret>
FRONTEND_ORIGIN=https://<frontend-domain>
COOKIE_SECURE=true
NODE_ENV=production
```

Generate strong JWT secrets with:

```bash
openssl rand -base64 48
```

### 3. Frontend Environment Variable

```text
OYU_API_BASE=https://<backend-domain>
```

Alternatively, inject the API base URL through an HTML template.

### 4. Database Migration

On the first deployment, Railway automatically runs:

```bash
prisma migrate deploy
```

through the configured deployment command.

### 5. Run the Seed Once

```bash
railway run npm run seed
```

---

# Feature Status

## Auth & Security

* ✅ Login
* ✅ Refresh token rotation + reuse detection
* ✅ Logout
* ✅ Account lockout
* ✅ Constant-time password verification
* ✅ Argon2id
* ✅ JWT access/refresh tokens
* ✅ Helmet + CSP
* ✅ HSTS in production
* ✅ Rate limiting
* ✅ CORS
* ✅ Audit log
* ✅ IT Auditor dashboard
* ✅ Configuration security view
* ✅ Login log
* ✅ Failed login monitoring
* ✅ Security dependency monitoring

## Student

* ✅ Dashboard:

  * GPA
  * active courses
  * today's schedule
  * latest grades
* ✅ Schedule:

  * complete weekly schedule
* ✅ Grades:

  * filtering
  * color coding
* ✅ **Assignments:**

  * assignment list
  * submission
* ✅ Attendance:

  * attendance history
* ✅ **Materials:**

  * grouped by course
  * external links
* ✅ **Requests:**

  * create requests
  * view personal requests
* ✅ **Transcript:**

  * cumulative GPA
  * semester breakdown
  * course grades
  * grade points
  * current/archive semester badges

## Teacher

* ✅ Dashboard:

  * groups
  * students
  * courses
  * schedule
* ✅ Journal:

  * view grades by group
  * add grades
* ✅ **Assignments:**

  * create assignments
  * view submissions
  * grade submissions through a prompt
* ✅ **Attendance Marking:**

  * select group
  * select course
  * select date
  * mark each student
* ✅ **Materials:**

  * create links
  * delete materials

## Leadership

* ✅ KPI dashboard
* ✅ Grade distribution
* ✅ Group statistics
* ✅ User search
* ✅ Request management
* ✅ Approve/reject requests

## i18n

* ✅ 3 languages:

  * Azerbaijani (`az`)
  * Russian (`ru`)
  * English (`en`)
* ✅ Both server-side and client-side i18n
* ✅ Language selector in the top-right corner
* ✅ Selected language stored in `localStorage`
* ✅ `X-Lang` header sent with API requests
* ✅ Localized server error messages

---

# Roadmap — Future Iterations

* 🟡 **2FA (TOTP) UI** — schema is ready
* 🟡 **File upload (Cloudflare R2)** — currently only the filename is stored
* 🟡 **Email (Resend)** — password reset and notifications
* 🟡 **Password change / forgot-password flow**
* 🟡 **Transcript PDF generation**
* 🟡 **Announcements / Notifications bell**
* 🟡 **Payments** — AZN payment gateway
* 🟡 **Multi-semester GPA / CGPA**
* 🟡 **Course catalog with syllabus**
* 🟡 **Academic calendar**

---

# ⚠ Important: Updating the Database Schema

The schema has been modified several times. Run the migrations sequentially:

```bash
cd oyu-backend

npx prisma migrate dev --name add_submission_notes
npx prisma migrate dev --name add_academic_year_term

npm run seed
```

Or, if this is a completely new installation with no previous migrations:

```bash
npx prisma migrate dev --name init
npm run seed
```

---

# Seed Data

The seed creates:

* **20 archived courses** covering the previous 5 semesters:

  * MATH101
  * PROG101/102
  * PHYS101/102
  * CS201–304
  * ENG101–202
  * and others
* **19 historical final grades** for student `0122184710`, representing a complete academic path from semesters 1–5
* **6 current-semester grades** for `SPRING 2025-2026`:

  * CS401–405
  * ENG301
* 2 assignments
* 3 materials
* attendance records
* 1 request

---

# Academic Transcript

A new **Transcript** section is available for students in `/portal.html`.

It includes:

### Cumulative GPA (CGPA)

The student's cumulative GPA is calculated using a **4.0 grading scale**.

### Semester Breakdown

Each semester displays:

* Semester GPA
* Earned credits
* Course list
* Letter grade:

  * A
  * B
  * C
  * D
  * F
* Grade points

### Sorting

Semesters are sorted from:

**Current semester → earliest semester**

### Semester Badges

The interface distinguishes between:

* **Current semester**
* **Archived semester**

---

# Semester Filters

The **My Grades** and **Attendance** pages now include semester filters.

Students can view:

* a specific academic year and term
* or all semesters

Example:

```text
Academic Year: 2024-2025
Term: FALL
```

---

# API

### Full Transcript

```http
GET /api/v1/me/transcript
```

Returns the complete transcript with GPA aggregation.

### Grades by Semester

```http
GET /api/v1/me/grades?academicYear=2024-2025&term=FALL
```

Returns grades filtered by academic year and semester.

### Attendance by Semester

```http
GET /api/v1/me/attendance?academicYear=2025-2026&term=SPRING
```

Returns attendance filtered by academic year and semester.

```

This version is ready to use as English project documentation / README content.
```
