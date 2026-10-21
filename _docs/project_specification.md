# 📋 Project Specification: Self-Hosted Household Task & Schedule Manager

## 🎯 Vision & Overview
A lightweight, self-hosted web application for managing household tasks, schedules, and routines. It features flexible time-blocked tasks, appointments, deadlines, sub-tasks, and a secure root-admin-led invitation system with 6-digit PIN authentication. Designed for low-resource hosting (such as a Mini-PC or Raspberry Pi) with zero external SaaS dependencies outside of a personal Gmail SMTP fallback for invites.

---

## 1. Technology Stack

* **Frontend:** React, Vite, TypeScript, Tailwind CSS (SPA)
* **Backend:** Fastify (Node.js framework), TypeScript
* **Database:** SQLite via `better-sqlite3` (Single-file database)
* **Authentication:** Bcrypt (PIN hashing), `jwt-simple` (HttpOnly session cookies), `@fastify/rate-limit` for brute-force protection.
* **Email Service:** Nodemailer connected to a personal Gmail account using an App Password.

---

## 2. Core Feature Requirements

### 🔐 2.1 Authentication & User Management
* **Root Administrator:** The initial user created on setup. Has full rights to invite members and manage the system.
* **Email Invitations:** Root admin inputs an email; the system generates a secure token and sends an invitation link via Gmail SMTP.
* **PIN Setup:** Invited users open the link, validate the token, and set a secure **6-digit numeric PIN**.
* **Secure Login:** Users log in using their email and 6-digit PIN via an on-screen keypad. Rate-limiting protects against brute-force attacks.

### 📅 2.2 Flexible Task Management & Scheduling
* **Task Types:**
  * **Time-Block:** Tasks with dedicated `start_datetime` and `end_datetime` durations.
  * **Appointment:** Fixed time-bound events with `start_datetime` and `end_datetime`.
  * **Deadline:** Tasks with a target `due_datetime` (no fixed working duration required).
* **Sub-Tasks:** Nested checklists inside tasks with individual completion toggles.
* **Priorities & Categories:** High, Medium, Low priority flags and custom category tags.
* **Recurrence Engine:** Support for Hourly, Daily, Weekly, Monthly, and Yearly recurring schedules.

---

## 3. Top-Level Data Schemas (SQLite)

### Users
* `id` (TEXT, Primary Key / UUID)
* `email` (TEXT, Unique, Not Null)
* `name` (TEXT, Not Null)
* `pin_hash` (TEXT, Not Null) — *Bcrypt hash of 6-digit PIN*
* `is_admin` (INTEGER, Default 0)
* `created_at` (TEXT, Timestamp)

### Invites
* `id` (TEXT, Primary Key / UUID)
* `email` (TEXT, Unique, Not Null)
* `token` (TEXT, Unique, Not Null)
* `expires_at` (TEXT, Timestamp)
* `created_by` (TEXT, Foreign Key -> `users.id`)

### Tasks
* `id` (TEXT, Primary Key / UUID)
* `title` (TEXT, Not Null)
* `description` (TEXT, Nullable)
* `assignee_id` (TEXT, Foreign Key -> `users.id`, Nullable)
* `created_by_id` (TEXT, Foreign Key -> `users.id`)
* `task_type` (TEXT, Enum: `TIME_BLOCK`, `APPOINTMENT`, `DEADLINE`)
* `start_datetime` (TEXT, Timestamp, Nullable)
* `end_datetime` (TEXT, Timestamp, Nullable)
* `due_datetime` (TEXT, Timestamp, Nullable)
* `is_all_day` (INTEGER, Default 0)
* `priority` (TEXT, Enum: `LOW`, `MEDIUM`, `HIGH`)
* `category` (TEXT, Nullable)
* `recurrence_type` (TEXT, Enum: `NONE`, `HOURLY`, `DAILY`, `WEEKLY`, `MONTHLY`, `YEARLY`)
* `recurrence_interval` (INTEGER, Default 1)
* `status` (TEXT, Enum: `PENDING`, `IN_PROGRESS`, `COMPLETED`)
* `completed_at` (TEXT, Timestamp, Nullable)
* `created_at` (TEXT, Timestamp)

### Subtasks
* `id` (TEXT, Primary Key / UUID)
* `task_id` (TEXT, Foreign Key -> `tasks.id`, Cascade Delete)
* `title` (TEXT, Not Null)
* `is_completed` (INTEGER, Default 0)
* `sort_order` (INTEGER, Default 0)

---

## 4. Software Development Life Cycle (SDLC) Roadmap

### Phase 1: Environment & Database Setup
* Initialize Git repository, Node/TypeScript configuration, and project folder structures.
* Create SQLite initialization scripts and schema migration files.
* Set up environment variables (`.env`) for Gmail SMTP credentials and JWT secrets.

### Phase 2: Backend Core & Authentication API
* Implement Fastify server framework setup and CORS configuration.
* Build authentication routes: Root admin bootstrap, invite generation, email dispatch via Nodemailer, token validation, and 6-digit PIN hashing/login with rate-limiting.
* Implement JWT cookie session handling middleware.

### Phase 3: Task & Sub-task API Endpoints
* Build CRUD endpoints for Tasks supporting time-blocks, appointments, deadlines, and recurrence rules.
* Implement sub-task management linked to parent tasks.
* Add filtering capabilities (by assignee, date range, status).

### Phase 4: Frontend SPA Foundation (React + Vite + Tailwind)
* Set up Vite, Tailwind CSS, and global routing.
* Build shared layout, navigation, and authentication views (including the custom 6-digit `PinPad` component).
* Implement state management and API client wrappers.

### Phase 5: Frontend Feature Implementation
* Build task creation/editing modal supporting the three distinct timing modes.
* Implement interactive sub-task checklists and status toggles.
* Build calendar/timeline views to visualize time blocks and deadlines.
* Create the Admin Invite dashboard.

### Phase 6: Testing, Hardening & Deployment
* Conduct end-to-end testing of the invitation flow and PIN login.
* Verify SQLite performance and resource utilization.
* Package application for self-hosting deployment (e.g., Docker container or systemd service setup on a mini-PC).