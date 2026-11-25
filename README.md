# Resolve – Online Complaint & Issue Management System

A clean, minimal, responsive full-stack SaaS web application designed for modern organizational grievance handling and transparent resolution workflows.

---

## 1. Key Technologies

- **Frontend:** React 18, TypeScript, Vite
- **Styling:** CSS3 (Custom SaaS design system, responsive layouts)
- **Icons:** [Lucide React](https://lucide.dev/)
- **Backend:** Node.js, Express.js (REST API)
- **Database:** MySQL / MariaDB (via `mysql2/promise` with parameterized queries)
- **Security:** JWT authentication (`jsonwebtoken`), bcrypt password hashing (`bcryptjs`), Multer file uploads

---

## 2. Design System & Palette

Designed following professional SaaS dashboard principles with minimal visual clutter, soft shadows, and clean borders.

| Element | Hex Code | Purpose |
| :--- | :--- | :--- |
| **Primary Navy** | `#172B4D` | Sidebar, headings, primary buttons, brand accents |
| **Background Cream** | `#F5F0E6` | Main background color across the entire application |
| **Accent Coral** | `#E98270` | Key CTAs, notification highlights, active indicators |
| **White** | `#FFFFFF` | Cards, input fields, containers, high-contrast areas |
| **Secondary Text** | `#697586` | Subtitles, timestamps, metadata labels |

---

## 3. User Roles & Capabilities

### A. End User
- **Register & Authenticate:** Register with email and password; secure JWT session.
- **User Dashboard:** View overview stats (Total Submitted, Pending Review, In Progress, Resolved) and recent complaints.
- **Submit Grievance:** File tickets with Title, Category (`Technical`, `Infrastructure`, `Service`, `Staff`, `Other`), Priority (`Low`, `Medium`, `High`), detailed description, and optional file attachment.
- **Automatic ID Generation:** Every ticket is assigned a unique tracking identifier (e.g., `CMP-202611-XXXX`).
- **My Complaints:** Filter complaints by status or category, perform full-text search, and view paginated records.
- **Complaint Details:** View ticket details, attachments, administrative resolution notes, and complete timeline.
- **Edit & Delete Guard:** Only complaints in `Pending` status may be modified or withdrawn by the owner.

### B. Administrator
- **Secure Admin Portal:** Restricted access; unauthorized users are automatically redirected.
- **Admin Dashboard:** High-level metrics, recent complaints table, and a built-in visual analytics chart (Status Breakdown Meter & Category Volume).
- **Complaint Management:** Search by ticket ID, title, user name, or email address; filter across all categories, priorities, and statuses.
- **Status Updates & Resolution Notes:** Transition complaints between `Pending`, `In Progress`, and `Resolved` with mandatory resolution documentation.
- **Audit History:** Full chronological timeline logging previous status, new status, timestamp, and administrator details for complete accountability.

---

## 4. Database Architecture (MySQL)

Database Name: `resolve_complaints`

```sql
-- 1. Users
CREATE TABLE `users` (
  `id` INT AUTO_INCREMENT PRIMARY KEY,
  `name` VARCHAR(100) NOT NULL,
  `email` VARCHAR(191) NOT NULL UNIQUE,
  `password` VARCHAR(255) NOT NULL,
  `role` ENUM('user', 'admin') NOT NULL DEFAULT 'user',
  `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP
);

-- 2. Complaints
CREATE TABLE `complaints` (
  `id` INT AUTO_INCREMENT PRIMARY KEY,
  `complaint_id` VARCHAR(50) NOT NULL UNIQUE,
  `user_id` INT NOT NULL,
  `title` VARCHAR(200) NOT NULL,
  `category` ENUM('Technical', 'Infrastructure', 'Service', 'Staff', 'Other') NOT NULL,
  `priority` ENUM('Low', 'Medium', 'High') NOT NULL,
  `description` TEXT NOT NULL,
  `attachment_url` VARCHAR(255) NULL,
  `status` ENUM('Pending', 'In Progress', 'Resolved') NOT NULL DEFAULT 'Pending',
  `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  `updated_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP,
  FOREIGN KEY (`user_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
);

-- 3. Complaint Updates (Audit & Resolution History)
CREATE TABLE `complaint_updates` (
  `id` INT AUTO_INCREMENT PRIMARY KEY,
  `complaint_id` INT NOT NULL,
  `admin_id` INT NOT NULL,
  `previous_status` ENUM('Pending', 'In Progress', 'Resolved') NOT NULL,
  `new_status` ENUM('Pending', 'In Progress', 'Resolved') NOT NULL,
  `resolution_notes` TEXT NOT NULL,
  `created_at` TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
  FOREIGN KEY (`complaint_id`) REFERENCES `complaints` (`id`) ON DELETE CASCADE,
  FOREIGN KEY (`admin_id`) REFERENCES `users` (`id`) ON DELETE CASCADE
);
```

---

## 5. Prerequisites & Requirements

- **Node.js** v18+ (tested on Node.js v25)
- **npm** v9+
- **MySQL** or **MariaDB** (e.g., via XAMPP, Laragon, or standalone MySQL Server running on port `3306`)

---

## 6. Installation & Quick Start

### Step 1: Start MySQL Server
Ensure your MySQL service is started (e.g., click "Start" next to MySQL in the XAMPP Control Panel).

### Step 2: Set Up Backend Environment & Database
```bash
# Navigate to backend
cd backend

# Install dependencies
npm install

# Initialize database schema (creates resolve_complaints database and tables)
npm run init-db

# Seed initial admin and demo user accounts
npm run seed-admin

# (Optional) Seed realistic sample complaints
npm run seed-samples

# Start backend server
npm start
```
The backend API will run at `http://localhost:5000`.

### Step 3: Set Up Frontend
Open a new terminal window:
```bash
# Navigate to frontend
cd frontend

# Install dependencies
npm install

# Start development server
npm run dev
```
The frontend will run at `http://localhost:5173`.

---

## 7. Default Credentials for Testing

| Role | Email | Password |
| :--- | :--- | :--- |
| **Administrator** | `admin@resolve.com` | `Admin@123` |
| **Demo User** | `user@resolve.com` | `User@123` |

> *Tip: The Login page includes 1-click **"Demo Admin"** and **"Demo User"** buttons to auto-populate credentials for quick demonstration.*

---

## 8. Creating Additional Administrator Accounts

To create custom administrator accounts, you can use the interactive script or run a query directly in MySQL:

### Option A: Using the Seed Script
Edit `backend/database/seed-admin.js` to define your desired credentials and run:
```bash
cd backend
npm run seed-admin
```

### Option B: Direct SQL Insertion with bcrypt
Hash a password (using any bcrypt tool or Node.js REPL) and execute:
```sql
USE `resolve_complaints`;
INSERT INTO `users` (`name`, `email`, `password`, `role`)
VALUES ('Admin Name', 'customadmin@domain.com', '<hashed_bcrypt_password>', 'admin');
```

---

## 9. REST API Reference

### Authentication
- `POST /api/auth/register` – Register new user
- `POST /api/auth/login` – Authenticate user and receive JWT
- `GET /api/auth/me` – Retrieve currently logged-in user details *(Auth required)*

### Complaints (User)
- `POST /api/complaints` – File a new complaint with optional attachment *(Auth required)*
- `GET /api/complaints` – Get paginated complaints belonging to current user *(Auth required)*
- `GET /api/complaints/user/dashboard` – Get user-specific stats & recent tickets *(Auth required)*
- `GET /api/complaints/:id` – View details and timeline of a complaint *(Auth required)*
- `PUT /api/complaints/:id` – Edit title, category, priority, description *(Owner only, Pending only)*
- `DELETE /api/complaints/:id` – Delete complaint ticket *(Owner only, Pending only)*

### Administration (Admin Only)
- `GET /api/admin/dashboard` – Global system statistics, status & category breakdown, recent complaints
- `GET /api/admin/complaints` – Search & filter across all organization complaints with pagination
- `PATCH /api/admin/complaints/:id/status` – Update complaint status and append resolution note to timeline
