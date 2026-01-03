# Research Document: Multi-Tenant SaaS Platform Architecture

**Project Name:** Multi-Tenant SaaS Project Management System  
**Date:** October 26, 2025  
**Author:** AWS Student / Lead Developer  
**Status:** Approved for Implementation  

---

## 1. Multi-Tenancy Architecture Analysis

Multi-tenancy is the architectural capability of a single software instance to serve multiple independent customer organizations (**tenants**) while maintaining logical data isolation.

In SaaS systems, the **database design strategy** is the most critical architectural decision, as it directly affects:

- Scalability
- Security & isolation
- Operational complexity
- Infrastructure cost

This section analyzes the three primary multi-tenancy models.

---

## 1.1 Shared Database, Shared Schema  
*(Discriminator Column / Pool Model)*

All tenants share a single database and schema.  
Data isolation is enforced using a discriminator column such as `tenant_id`.

### Mechanism
- Every tenant-owned table contains `tenant_id`
- All queries include `WHERE tenant_id = ?`
- Isolation is enforced at the application layer

### Pros
- Lowest infrastructure cost
- Instant tenant onboarding
- Simple DevOps (single database)
- Easy cross-tenant analytics

### Cons
- High risk if `tenant_id` is missed in queries
- Difficult per-tenant backups
- Noisy-neighbor performance risk

---

## 1.2 Shared Database, Separate Schemas  
*(Schema-per-Tenant / Bridge Model)*

Each tenant has its own database schema  
(e.g., `tenant_a.users`, `tenant_b.users`).

### Mechanism
- Database search path set dynamically per request
- Schema-level isolation enforced by the database

### Pros
- Stronger isolation than shared schema
- Easier per-tenant backups
- Supports limited tenant customization

### Cons
- Schema migration complexity grows with tenant count
- Schema catalog bloat
- Slower deployments at scale

---

## 1.3 Separate Databases  
*(Database-per-Tenant / Silo Model)*

Each tenant is assigned a fully isolated database instance.

### Mechanism
- Central tenant catalog maps tenant → database connection

### Pros
- Maximum security and isolation
- No noisy-neighbor issues
- Ideal for regulated industries

### Cons
- Very high infrastructure cost
- Complex DevOps and monitoring
- Poor fit for freemium SaaS models

---

## 1.4 Comparison Summary

| Feature | Shared Schema | Separate Schema | Separate Database |
|------|---------------|-----------------|------------------|
| Isolation | Low | Medium | High |
| Cost Efficiency | High | Medium | Low |
| Onboarding Speed | Instant | Fast | Slow |
| DevOps Effort | Low | Medium | High |
| Data Leak Risk | High | Medium | Very Low |
| Maintenance | Easy | Hard | Very Hard |
| Scalability | Vertical | Vertical | Horizontal |

---

## 1.5 Chosen Approach: Shared Database + Shared Schema

### Justification

1. **MVP Focus**  
   Prioritizes core SaaS functionality over infrastructure orchestration.

2. **ORM Compatibility (Prisma)**  
   Prisma works best with a single schema using a discriminator pattern.

3. **Dockerized Deployment**  
   Enables simple `docker-compose` based environments.

4. **Risk Mitigation**  
   Middleware automatically injects `tenant_id`, eliminating developer error.

---

## 2. Technology Stack Justification

### Backend: Node.js + Express

**Reasons**
- Non-blocking I/O for high concurrency
- Middleware-friendly architecture
- Single language across full stack

**Rejected Alternatives**
- Django (synchronous defaults, rigid ORM)
- Spring Boot (heavy runtime and configuration)

---

### Frontend: React (Vite)

**Reasons**
- Component reusability
- Fast rendering via Virtual DOM
- Rich ecosystem (Router, Axios)

**Rejected**
- Angular (steep learning curve)

---

### Database: PostgreSQL

**Reasons**
- Strong relational integrity
- Cascade deletes
- JSONB flexibility
- Future-ready for Row-Level Security (RLS)

**Rejected**
- MongoDB (weaker relational guarantees)

---

### Authentication: JWT

**Reasons**
- Stateless and horizontally scalable
- Works across subdomains
- No shared session store required

**Rejected**
- Server-side sessions (Redis dependency)

---

### Deployment: Docker & Docker Compose

**Reasons**
- Environment parity
- One-command deployment
- Simple service orchestration

---

## 3. Security Considerations

### 3.1 Middleware-Based Tenant Isolation

- JWT validated on every request
- `tenant_id` injected server-side
- Client-supplied tenant identifiers are never trusted

---

### 3.2 Secure Password Storage

- Passwords hashed using **Bcrypt**
- Salt rounds ≥ 10
- Protects against rainbow table attacks

---

### 3.3 Role-Based Access Control (RBAC)

| Role | Capabilities |
|----|-------------|
| Super Admin | Global system access |
| Tenant Admin | Full tenant control |
| User | Restricted access |

RBAC guards execute **before** database access.

---

### 3.4 API Hardening

- Strict CORS policies
- Rate limiting for authentication endpoints
- Helmet security headers

---

### 3.5 Input Validation & ORM Safety

- Request payloads validated using schemas
- Prisma parameterized queries
- Eliminates SQL Injection risks

---

## 4. Data Isolation Strategy (Row-Level Pattern)

- `tenant_id` present in all tenant-owned tables
- All queries are scoped using JWT-derived tenant context

```sql
SELECT *
FROM tasks
WHERE id = :taskId
  AND tenant_id = :jwtTenantId;
Unauthorized access returns 404 Not Found, ensuring foreign tenant data remains invisible.

5. Authentication & Authorization Flow
Step 1: User Login
User logs in using credentials and tenant subdomain.

Step 2: Tenant & User Validation
Backend resolves tenant by subdomain and validates user within that tenant.

Step 3: JWT Issuance
On success, a signed JWT is issued:

json
Copy code
{
  "userId": "<uuid>",
  "tenantId": "<uuid>",
  "role": "<role>"
}
Step 4: Authenticated Requests
Client sends JWT with each protected request:

http
Copy code
Authorization: Bearer <token>
Step 5: Token Validation
JWT signature verified

Claims attached to request context

Step 6: Authorization (RBAC)
Role-based guards validate permissions

Unauthorized requests are blocked early

Enforces least-privilege access

6. Authentication & Authorization Sequence Diagram
mermaid
Copy code
sequenceDiagram
    autonumber
    participant User as Client
    participant API as Backend API
    participant Auth as Auth Middleware
    participant RBAC as RBAC Guard
    participant DB as PostgreSQL

    User->>API: POST /auth/login (credentials + subdomain)
    API->>DB: Resolve tenant
    API->>DB: Validate user
    API->>API: Compare password (Bcrypt)

    API-->>User: JWT issued

    User->>API: Request with Bearer JWT
    API->>Auth: Verify JWT
    Auth-->>API: Claims attached

    API->>RBAC: Role validation
    RBAC-->>API: Access granted

    API->>DB: Query with tenant_id
    DB-->>API: Scoped data
    API-->>User: 200 OK
