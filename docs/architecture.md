# System Architecture Document

**Project Name:** Multi-Tenant SaaS Project Management System  
**Date:** October 26, 2025  
**Version:** 1.0  
**Author:** AWS Student / Lead Developer  

---

## 1. System Architecture Overview

The system follows a **containerized three-tier web architecture** designed for:

- Scalability
- Modularity
- Strong tenant isolation
- Cloud-native deployment readiness

The entire stack is orchestrated using **Docker Compose**, ensuring consistent environments across development and production.

---

## 2. High-Level Architecture Diagram

```mermaid
graph LR
    User[Client Browser] --> Frontend[Frontend Container<br/>React + Vite]
    Frontend --> Backend[Backend Container<br/>Node.js + Express]
    Backend --> DB[(PostgreSQL Database)]

    subgraph Docker_Network
        Frontend
        Backend
        DB
    end

    subgraph Security_Layer
        Backend --> JWT[JWT Authentication]
        Backend --> RBAC[RBAC Authorization]
    end
3. System Architecture Components
The system is built using a multi-tenant, containerized architecture with strict separation of concerns:

Client Layer (Frontend)

Application Layer (Backend API)

Data Layer (Database)

4. Component Details
4.1 Client Layer (Frontend)
Technology: React.js (Vite)

Container Port: 3000 → 3000

Responsibilities:

Render user interface

Handle user interactions

Manage authentication state (JWT)

Communicate with Backend REST APIs

Multi-Tenancy Handling
Tenant context is determined by:

Subdomain (tenant1.app.com), or

Tenant-aware login flow

4.2 Application Layer (Backend API)
Technology: Node.js, Express.js

Container Port: 5000 → 5000

Responsibilities:

Business logic execution

JWT authentication

Role-Based Access Control (RBAC)

Tenant data isolation

Tenant Isolation Mechanism
JWT contains tenant_id

Middleware extracts and validates tenant_id

All database queries are scoped by tenant_id

Prevents cross-tenant data access

4.3 Data Layer (Database)
Technology: PostgreSQL 15

Container Port: 5432 → 5432

Responsibilities:

Persistent relational storage

ACID compliance

Referential integrity

Isolation Strategy
Shared Database, Shared Schema

Logical isolation using tenant_id

tenant_id exists in all tenant-owned tables

5. High-Level Multi-Tenant Architecture
mermaid
Copy code
graph LR
    Client[Frontend<br/>React :3000]
        -->|HTTPS + JWT|
    API[Backend API<br/>Node :5000]

    API
        -->|tenant_id scoped SQL|
    DB[(PostgreSQL 15<br/>Shared Database)]

    subgraph Multi_Tenant_Isolation
        API
        DB
    end
6. Database Schema Design (ERD)
The database is normalized to Third Normal Form (3NF) to eliminate redundancy and ensure data consistency.

The tenant_id column serves as the logical partition key for enforcing multi-tenancy.

mermaid
Copy code
erDiagram
    TENANTS ||--o{ USERS : owns
    TENANTS ||--o{ PROJECTS : owns
    TENANTS ||--o{ TASKS : owns
    TENANTS ||--o{ AUDIT_LOGS : records

    USERS ||--o{ PROJECTS : creates
    PROJECTS ||--o{ TASKS : contains
    USERS ||--o{ TASKS : assigned_to

    TENANTS {
        uuid id PK
        string name
        string subdomain UK
        string status
        string subscription_plan
        int max_users
        int max_projects
    }

    USERS {
        uuid id PK
        uuid tenant_id FK
        string email
        string password_hash
        string full_name
        string role
    }

    PROJECTS {
        uuid id PK
        uuid tenant_id FK
        uuid created_by FK
        string name
        string description
        string status
    }

    TASKS {
        uuid id PK
        uuid tenant_id FK
        uuid project_id FK
        uuid assigned_to FK
        string title
        string priority
        string status
        date due_date
    }

    AUDIT_LOGS {
        uuid id PK
        uuid tenant_id FK
        string action
        string entity_type
        uuid entity_id
        string ip_address
    }
7. Schema Details
7.1 tenants (Root Entity)
Primary Key: id (UUID)

Fields:

name

subdomain (Unique)

status

subscription_plan

Constraints:

max_users

max_projects

Note: Root table (no tenant_id)

7.2 users
Primary Key: id (UUID)

Foreign Key:

tenant_id → tenants.id (ON DELETE CASCADE)

Constraints:

UNIQUE (tenant_id, email)

Fields:

email

password_hash

full_name

role

7.3 projects
Primary Key: id (UUID)

Foreign Keys:

tenant_id → tenants.id

created_by → users.id

Index:

sql
Copy code
CREATE INDEX idx_projects_tenant ON projects(tenant_id);
7.4 tasks
Primary Key: id (UUID)

Foreign Keys:

tenant_id → tenants.id

project_id → projects.id

assigned_to → users.id

Index:

sql
Copy code
CREATE INDEX idx_tasks_tenant ON tasks(tenant_id);
7.5 audit_logs
Primary Key: id (UUID)

Foreign Key:

tenant_id → tenants.id

Purpose:

Security auditing

Compliance

Activity tracking

8. API Architecture
The backend exposes 19 RESTful endpoints following standard REST principles.

Standard API Response Format
json
Copy code
{
  "success": true,
  "message": "Operation completed successfully",
  "data": {}
}
9. API Modules
Module A: Authentication
Method	Endpoint	Description	Auth	Role
POST	/api/auth/register-tenant	Register tenant & admin	No	Public
POST	/api/auth/login	Login & receive JWT	No	Public
GET	/api/auth/me	Get current user	Yes	Any
POST	/api/auth/logout	Logout user	Yes	Any

Module B: Tenant Management
Method	Endpoint	Description	Auth	Role
GET	/api/tenants	List all tenants	Yes	super_admin
GET	/api/tenants/:tenantId	Get tenant details	Yes	super_admin
PUT	/api/tenants/:tenantId	Update tenant	Yes	admin

Module C: User Management
Method	Endpoint	Description	Auth	Role
POST	/api/tenants/:tenantId/users	Create user	Yes	tenant_admin
GET	/api/tenants/:tenantId/users	List users	Yes	Any
PUT	/api/users/:userId	Update user	Yes	admin/self
DELETE	/api/users/:userId	Delete user	Yes	tenant_admin

Module D: Project Management
Method	Endpoint	Description	Auth	Role
POST	/api/projects	Create project	Yes	Any
GET	/api/projects	List projects	Yes	Any
PUT	/api/projects/:projectId	Update project	Yes	Owner/Admin
DELETE	/api/projects/:projectId	Delete project	Yes	Owner/Admin

Module E: Task Management
Method	Endpoint	Description	Auth	Role
POST	/api/projects/:projectId/tasks	Create task	Yes	Any
GET	/api/projects/:projectId/tasks	List tasks	Yes	Any
PATCH	/api/tasks/:taskId/status	Update task status	Yes	Any
PUT	/api/tasks/:taskId	Update task	Yes	Any

markdown
Copy code
