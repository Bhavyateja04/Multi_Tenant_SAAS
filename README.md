# Multi-Tenant SaaS Platform

## Overview

A comprehensive **B2B SaaS Project Management Platform** built to demonstrate **Multi-Tenancy**, **Strict Data Isolation**, and **Role-Based Access Control (RBAC)**.

The system enables multiple organizations (**Tenants**) to operate within a single application instance while maintaining complete logical separation of data. Each tenant manages its own users, projects, and tasks without visibility into other organizations.

### Role Hierarchy
- **Super Admin** – Manages the entire SaaS platform and all tenants
- **Tenant Admin** – Manages users, projects, and settings for their organization
- **Standard User** – Works on assigned projects and tasks

---

## Key Features

- **Multi-Tenant Architecture**  
  Logical data isolation using `tenant_id` across all resources

- **Secure Authentication**  
  JWT-based authentication with Bcrypt password hashing

- **Role-Based Access Control (RBAC)**  
  Fine-grained permissions for Super Admins, Tenant Admins, and Users

- **Project Management**  
  Create, update, and track tenant-scoped projects

- **Task Orchestration**  
  Assign tasks with priorities, due dates, and status tracking

- **Team Management**  
  Tenant Admins can invite, remove, and manage users

- **Super Admin Dashboard**  
  Global visibility into all tenants and system usage

- **Responsive UI**  
  Modern React dashboard optimized for desktop and mobile

---

## Technology Stack

### Frontend
- **Framework:** React.js (v18)
- **State Management:** React Context API
- **Routing:** React Router DOM (v6)
- **HTTP Client:** Axios
- **Styling:** CSS3 (Flexbox & Grid)

### Backend
- **Runtime:** Node.js (v18)
- **Framework:** Express.js
- **ORM:** Prisma
- **Security:** JWT, Bcrypt, CORS
- **Validation:** Express Validator

### Database & DevOps
- **Database:** PostgreSQL (v15)
- **Containerization:** Docker & Docker Compose
- **Environment:** Linux (Alpine) containers

---

## Architecture Overview

The application follows a **Client–Server architecture**:

- React frontend communicates with the backend via REST APIs
- Backend enforces tenant isolation using `tenant_id` on every request
- PostgreSQL serves as a shared database with logical isolation
- All services are orchestrated using Docker Compose

---

## Installation & Setup

### Prerequisites
- Docker & Docker Compose (**Recommended**)
- Node.js v18+ (for manual/local setup)
- PostgreSQL (for manual/local setup)

---

## Method 1: Docker (Recommended)

Automatically sets up **Database**, **Backend**, and **Frontend**.

### 1. Clone the Repository
```bash
git clone <your-repo-url>
cd Multi-Tenant-SaaS-Platform
2. Configure Environment
Create a .env file in the root directory (or verify docker-compose.yml):

properties
Copy code
POSTGRES_USER=postgres
POSTGRES_PASSWORD=postgres
POSTGRES_DB=saas_db
3. Launch the Application
bash
Copy code
docker-compose up -d --build
4. Access the Application
Frontend: http://localhost:3000

Backend Health Check: http://localhost:5000/api/health

The system automatically runs database migrations and seed data on startup.

Method 2: Local Development (Manual)
<details> <summary>Click to expand</summary>
1. Database Setup
Ensure PostgreSQL is running on port 5432.

2. Backend Setup
bash
Copy code
cd backend
npm install
cp .env.example .env
# Update .env with local DB credentials
npx prisma migrate dev --name init
npm run seed
npm start
3. Frontend Setup
bash
Copy code
cd frontend
npm install
npm start
</details>
Environment Variables
See .env.example in the backend directory.

Variable	Description	Default (Docker)
PORT	Backend server port	5000
DATABASE_URL	PostgreSQL connection string	postgresql://...@database:5432/saas_db
JWT_SECRET	JWT signing secret	Set your own
FRONTEND_URL	CORS allowed origin	http://localhost:3000

API Overview
Authentication
POST /api/auth/register-tenant – Register a new organization

POST /api/auth/login – User login

Projects & Tasks
GET /api/projects – List tenant projects

POST /api/projects – Create a project

POST /api/projects/:id/tasks – Add task to project

PATCH /api/tasks/:id/status – Update task status

Management
GET /api/tenants – (Super Admin) List all tenants

GET /api/tenants/:id/users – (Tenant Admin) List users

POST /api/tenants/:id/users – (Tenant Admin) Add user

Testing Credentials (Seed Data)
Super Admin
Email: superadmin@system.com

Password: Admin@123

Tenant Admin (Demo Company)
Email: admin@demo.com

Password: Admin@123

Subdomain: demo

License
This project is intended for educational and demonstration purposes.