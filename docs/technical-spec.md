# Technical Specification

**Project Name:** Multi-Tenant SaaS Project Management System  
**Date:** October 26, 2025  
**Version:** 1.0  
**Author:** AWS Student / Lead Developer  

---

## 1. Project Structure

The project follows a **Monorepo architecture**, containing both the **Backend API** and the **Frontend React application**, orchestrated using **Docker Compose** at the root level.

---

## 1.1 Root Directory Structure

```text
/Multi-Tenant-SaaS-Platform
├── docker-compose.yml       # Orchestration for DB, Backend, Frontend
├── submission.json          # Credentials for automated evaluation
├── README.md                # Entry point documentation
├── .gitignore               # Git ignore rules
├── docs/                    # Architecture, PRD, Research documents
├── backend/                 # Node.js / Express API
└── frontend/                # React (Vite) application
1.2 Backend Structure (/backend)
The backend is built using Node.js, Express, and Prisma, following a modular and scalable architecture.

text
Copy code
backend/
├── .env.example             # Environment variable template
├── Dockerfile               # Backend container definition
├── package.json             # Backend dependencies
├── prisma/
│   ├── schema.prisma        # Database schema
│   └── migrations/          # Migration history
├── seeds/
│   └── seed.js              # Database seed script
└── src/
    ├── controllers/         # Business logic (Auth, Tenant, Project)
    ├── middleware/          # Auth, RBAC, Error handling
    ├── routes/              # API route definitions
    └── utils/               # Utilities (hashing, JWT helpers)
1.3 Frontend Structure (/frontend)
The frontend is developed using React with Vite for fast development and optimized builds.

text
Copy code
frontend/
├── Dockerfile               # Frontend container definition
├── package.json             # Frontend dependencies
├── public/                  # Static assets
└── src/
    ├── context/             # Global state (AuthContext)
    ├── pages/               # UI pages (Login, Dashboard, Register)
    ├── App.js               # Root component & routing
    └── index.js             # React DOM entry point
2. Development Setup Guide
2.1 Prerequisites
Ensure the following tools are installed:

Docker Desktop — Version 4.0+

Node.js — Version 18 LTS

Git — Version 2.0+

2.2 Environment Variables
Create a .env file inside the backend/ directory
(or rely on defaults defined in docker-compose.yml).

Required Variables
ini
Copy code
# Server
PORT=5000
NODE_ENV=development

# Database (Docker Internal Network)
DATABASE_URL=postgresql://postgres:postgres@database:5432/saas_db?schema=public

# Security
JWT_SECRET=your_secure_random_secret_key_min_32_chars
JWT_EXPIRES_IN=24h

# CORS
FRONTEND_URL=http://localhost:3000
2.3 Installation Steps
Clone the Repository
bash
Copy code
git clone <repository_url>
cd Multi-Tenant-SaaS-Platform
Optional: Install Dependencies Locally
This is optional but recommended for IDE support.

Backend
bash
Copy code
cd backend
npm install
Frontend
bash
Copy code
cd ../frontend
npm install
2.4 Running the Application (Docker – Recommended)
The entire stack runs using Docker Compose, ensuring proper networking between services.

Build and Start Containers
From the root directory:

bash
Copy code
docker-compose up -d --build
Verify Running Services
bash
Copy code
docker-compose ps
You should see:

database

backend

frontend

Automatic Initialization
On startup, the backend container automatically executes:

prisma migrate deploy

node seeds/seed.js

⏳ Wait 30–60 seconds for the database to initialize.

2.5 Accessing the Application
Frontend: http://localhost:3000

Backend API: http://localhost:5000

Health Check: http://localhost:5000/api/health

2.6 Testing & Verification
Manual API Testing (Postman / Curl)
Use credentials from submission.json.

Health Check Example
bash
Copy code
curl http://localhost:5000/api/health
Expected Response:

json
Copy code
{
  "status": "ok",
  "database": "connected"
}
Database Inspection
Connect to the PostgreSQL container:

bash
Copy code
docker exec -it database psql -U postgres -d saas_db
Example query:

sql
Copy code
SELECT * FROM tenants;