# Spec File Templates

Templates with few-shot examples for each spec file. The `design` skill references this file when drafting specs under `spec/`.

**Rules:**
- **Always create**: `glossary.md`, `constraints.md`, `architecture-decisions.md`, `overview.md`, `non-functional-requirements.md`, `use-cases.md`, `flows.md`
- **Create when applicable**: `data-model.md` (persistence), `api-design.md` (API), `deployment.md` (infra), `ui/` (frontend)
- **Add custom files as needed**: e.g., `security-model.md`, `event-bus.md`
- **Use mermaid** for all diagrams
- **Each file must be self-contained** — readable on its own
- **Reference atdd.md Features** — spec files should trace back to the Features defined in atdd.md

---

## glossary.md

Shared vocabulary. Prevents "what do you mean by X?" across teams.

```markdown
# Glossary

Ubiquitous language for this project. All spec files, code, and communication
should use these terms consistently.

| Term | Definition | Context |
|---|---|---|
| Tenant | An organization that has its own isolated data and configuration | Multi-tenancy model |
| Workspace | A collaborative space within a Tenant where members work together | Tenant → Workspace → Member |
| Member | A user who belongs to one or more Workspaces with assigned roles | Not "user" — "user" is ambiguous |
| Plan | A subscription tier (Free, Pro, Enterprise) that determines feature access | Billing context, not "plan" as in project plan |
| Seat | A billable unit — one Member occupies one Seat | Billing: Seats × Price = Invoice |
| Invitation | A pending request for someone to join a Workspace | Status: pending → accepted / expired |
| RBAC | Role-Based Access Control — permissions assigned through roles, not directly | Roles: owner, admin, member, viewer |
```

---

## constraints.md

Technology, regulatory, and organizational constraints that limit design choices.

```markdown
# Constraints

Hard constraints that the design must respect. These are not negotiable
without stakeholder approval.

## Technology Constraints

| Constraint | Reason | Impact |
|---|---|---|
| Must use PostgreSQL 16+ | Company standard, DBA team only supports Postgres | No MongoDB, DynamoDB, etc. |
| Must deploy on AWS EKS | Existing infrastructure, ops team expertise | Architecture must be container-friendly |
| Must use company SSO (Okta SAML) | Security policy — no custom auth for internal tools | Auth flow must integrate OIDC/SAML |
| Node.js 22 LTS for backend | Team expertise, existing tooling | No Rust/Go/Java for backend services |

## Regulatory Constraints

| Constraint | Regulation | Impact |
|---|---|---|
| User data must be deletable within 30 days | GDPR Art. 17 (Right to Erasure) | Soft-delete + scheduled hard-delete pipeline |
| Audit log for all data access | SOC 2 Type II | Every read/write to PII must be logged |
| Data residency in EU for EU customers | GDPR | Multi-region deployment or data routing |

## Organizational Constraints

| Constraint | Reason | Impact |
|---|---|---|
| Max 3 developers for MVP | Team capacity | Scope must be achievable with small team |
| Must integrate with existing billing service | Finance team mandate | Cannot build custom billing |
| Launch deadline: Q3 2026 | Business commitment | Phasing must fit timeline |
```

---

## architecture-decisions.md

Architecture Decision Records (ADRs). Captures the "why" behind decisions.

```markdown
# Architecture Decisions

Key decisions made during design. Each record captures context, decision,
alternatives considered, and consequences.

## ADR-001: REST over gRPC for public API

**Status:** Accepted
**Date:** 2026-05-12

**Context:**
We need a public API for third-party integrations. Our team has deep REST
experience. Clients will be browser-based SPAs and mobile apps.

**Decision:**
Use REST (OpenAPI 3.1) for all public-facing APIs.

**Alternatives considered:**
- **gRPC**: Better performance, strong typing. Rejected — browser support
  requires grpc-web proxy, adds complexity for external developers.
- **GraphQL**: Flexible queries. Rejected — adds caching complexity,
  overpowered for our use case (well-defined resources, not graph-shaped data).

**Consequences:**
- (+) Easy onboarding for third-party developers
- (+) Rich tooling ecosystem (Swagger, Postman)
- (-) No streaming — will need WebSocket for real-time features
- (-) Over/under-fetching for some endpoints

---

## ADR-002: Event-driven communication between services

**Status:** Accepted
**Date:** 2026-05-12

**Context:**
Billing service needs to know when a Member is added (to update seat count).
Notification service needs to know when an Invitation is sent. Direct API
calls create tight coupling.

**Decision:**
Use async event bus (AWS SNS + SQS) for inter-service communication.
Services publish domain events; consumers subscribe independently.

**Alternatives considered:**
- **Direct HTTP calls**: Simple. Rejected — creates runtime coupling,
  cascading failures.
- **Shared database**: Easy. Rejected — violates service boundary,
  schema coupling.

**Consequences:**
- (+) Loose coupling between services
- (+) Easy to add new consumers without modifying publisher
- (-) Eventual consistency — must handle stale reads
- (-) Debugging async flows is harder (need correlation IDs)
```

---

## overview.md

Architecture entry point. C4 Context + Container level, solution strategy, tech stack.

```markdown
# Architecture Overview

## System Context

```mermaid
C4Context
    title System Context Diagram
    Person(user, "End User", "Uses the web application")
    Person(admin, "Admin", "Manages tenant settings")
    System(system, "Workspace Platform", "Multi-tenant collaboration platform")
    System_Ext(sso, "Okta SSO", "Authentication provider")
    System_Ext(billing, "Billing Service", "Existing company billing")
    System_Ext(email, "SendGrid", "Email delivery")

    Rel(user, system, "Uses", "HTTPS")
    Rel(admin, system, "Manages", "HTTPS")
    Rel(system, sso, "Authenticates via", "SAML/OIDC")
    Rel(system, billing, "Reports seat changes", "Events")
    Rel(system, email, "Sends emails via", "API")
` ``

## Container Diagram

```mermaid
C4Container
    title Container Diagram
    Person(user, "End User")

    Container_Boundary(platform, "Workspace Platform") {
        Container(spa, "Web App", "React, TypeScript", "Single-page application")
        Container(api, "API Server", "Node.js, Express", "REST API, business logic")
        Container(worker, "Event Worker", "Node.js", "Processes async events")
        ContainerDb(db, "Database", "PostgreSQL 16", "Tenant, workspace, member data")
        ContainerQueue(queue, "Event Bus", "SNS + SQS", "Domain events")
    }

    Rel(user, spa, "Uses", "HTTPS")
    Rel(spa, api, "Calls", "REST/JSON")
    Rel(api, db, "Reads/Writes", "SQL")
    Rel(api, queue, "Publishes events")
    Rel(worker, queue, "Consumes events")
` ``

## Tech Stack

| Layer | Technology | Rationale |
|---|---|---|
| Frontend | React 19, TypeScript, Tailwind CSS | Team expertise, component ecosystem |
| Backend | Node.js 22 LTS, Express, TypeScript | Company standard (see constraints.md) |
| Database | PostgreSQL 16, Prisma ORM | Company standard, JSONB for flexible metadata |
| Event Bus | AWS SNS + SQS | Managed, reliable, existing infra (see ADR-002) |
| Auth | Okta SAML/OIDC | Company SSO mandate (see constraints.md) |
| Deployment | AWS EKS, Docker | Existing infrastructure |

## Cross-Cutting Concerns

### Authentication & Authorization
- Okta SSO handles authentication (SAML 2.0)
- RBAC with four roles: owner, admin, member, viewer
- JWT tokens with 15min expiry, refresh via secure cookie

### Error Handling
- Structured error responses: `{ code, message, details }`
- Business errors: 4xx with error codes (`SEAT_LIMIT_REACHED`)
- System errors: 5xx, logged with correlation ID, generic message to client

### Observability
- Structured JSON logging (correlation ID on every request)
- OpenTelemetry traces for cross-service flows
- Health check endpoints for each service
```

---

## non-functional-requirements.md

Measurable quality targets. Prevents teams from making contradictory assumptions.

```markdown
# Non-Functional Requirements

Measurable quality targets. Each requirement must have a metric and a threshold.

## Performance

| Requirement | Metric | Target | Measurement |
|---|---|---|---|
| API response time | p95 latency | < 200ms | Load test (k6) |
| Page load time | Largest Contentful Paint | < 1.5s | Lighthouse CI |
| Database query time | p95 query duration | < 50ms | PostgreSQL slow query log |
| Event processing | End-to-end event latency | < 5s | CloudWatch metrics |

## Scalability

| Requirement | Target | Notes |
|---|---|---|
| Concurrent users | 10,000 | Per-tenant isolation must not degrade |
| Tenants | 1,000 | Schema-per-tenant not viable, use row-level isolation |
| Data volume | 100GB per tenant | Partitioning strategy needed for large tenants |

## Availability

| Requirement | Target |
|---|---|
| Uptime SLA | 99.9% (8.7h downtime/year) |
| RTO (Recovery Time Objective) | < 1 hour |
| RPO (Recovery Point Objective) | < 5 minutes |

## Security

| Requirement | Standard | Notes |
|---|---|---|
| Data at rest encryption | AES-256 | RDS encryption enabled |
| Data in transit encryption | TLS 1.3 | All endpoints HTTPS |
| OWASP Top 10 compliance | OWASP 2021 | Verified via OWASP ZAP scan |
| Dependency vulnerability scan | CVE database | Snyk or Dependabot in CI |
```

---

## use-cases.md

Use case descriptions organized by Feature (matching atdd.md).

```markdown
# Use Cases

Organized by Feature from atdd.md. Each use case describes actor interactions,
flows, and postconditions.

## Feature: Workspace Management

### UC-01: Create Workspace

**Actors:** Admin
**Preconditions:** Admin is authenticated, Tenant has available seat capacity
**Triggers:** Admin clicks "Create Workspace"

**Main Flow:**
1. Admin enters workspace name and description
2. System validates name uniqueness within Tenant
3. System creates Workspace with Admin as owner
4. System publishes `workspace.created` event
5. System redirects Admin to new Workspace dashboard

**Alternative Flows:**
- **2a. Name already exists:** System shows error "Workspace name already taken"
- **3a. Seat limit reached:** System shows error "Upgrade plan to add more workspaces"

**Postconditions:**
- Workspace exists in database with status `active`
- Admin has `owner` role in the Workspace
- Billing service received seat update event

**Related ATDD items:**
- Feature: Workspace Management → "Admin can create a new workspace"

---

### UC-02: Invite Member to Workspace

**Actors:** Admin, Invitee (external)
**Preconditions:** Admin has `admin` or `owner` role in Workspace

**Main Flow:**
1. Admin enters invitee's email and selects a role
2. System creates Invitation with status `pending`, expiry 7 days
3. System sends invitation email via SendGrid
4. Invitee clicks link, authenticates via SSO
5. System changes Invitation status to `accepted`
6. System creates Member record with assigned role
7. System publishes `member.joined` event

**Alternative Flows:**
- **2a. Email already a member:** System shows "Already a member of this workspace"
- **4a. Invitee not in SSO directory:** SSO rejects, show "Contact your IT admin"
- **4b. Invitation expired:** Show "This invitation has expired. Ask for a new one."

**Postconditions:**
- Member exists with assigned role
- Invitation status is `accepted`
- Billing service received seat update event
```

---

## flows.md

Sequence diagrams, state diagrams, and event flows. Replaces the narrower `sequence-diagrams.md`.

```markdown
# Flows

Sequence diagrams, state machines, and event flows for critical paths.
All diagrams use mermaid.

## Sequence: Invite Member Flow

```mermaid
sequenceDiagram
    actor Admin
    participant API as API Server
    participant DB as PostgreSQL
    participant Bus as Event Bus
    participant Email as SendGrid
    actor Invitee

    Admin->>API: POST /workspaces/:id/invitations
    API->>DB: Check seat capacity
    DB-->>API: OK (seats available)
    API->>DB: Create Invitation (pending)
    API->>Bus: Publish invitation.created
    Bus->>Email: Send invitation email
    API-->>Admin: 201 Created

    Invitee->>API: GET /invitations/:token/accept
    API->>DB: Validate token + expiry
    API->>DB: Create Member, update Invitation
    API->>Bus: Publish member.joined
    API-->>Invitee: Redirect to workspace
` ``

## State: Invitation Lifecycle

```mermaid
stateDiagram-v2
    [*] --> Pending: Admin sends invitation
    Pending --> Accepted: Invitee accepts
    Pending --> Expired: 7 days pass
    Pending --> Revoked: Admin cancels
    Expired --> [*]
    Revoked --> [*]
    Accepted --> [*]
` ``

## Event Flow: Member Joined

```mermaid
flowchart LR
    API[API Server] -->|member.joined| Bus[Event Bus]
    Bus --> Billing[Billing Service<br/>Update seat count]
    Bus --> Notify[Notification Service<br/>Welcome email]
    Bus --> Audit[Audit Service<br/>Log access grant]
` ``
```

---

## data-model.md

Entity-relationship diagram and schema descriptions. Create when the system has persistence.

```markdown
# Data Model

Entity relationships and schema design for PostgreSQL.

## ER Diagram

```mermaid
erDiagram
    TENANT ||--o{ WORKSPACE : has
    TENANT ||--o{ SUBSCRIPTION : has
    WORKSPACE ||--o{ MEMBERSHIP : has
    WORKSPACE ||--o{ INVITATION : has
    USER ||--o{ MEMBERSHIP : "belongs via"
    USER ||--o{ INVITATION : "receives"

    TENANT {
        uuid id PK
        string name
        string slug UK
        timestamp created_at
    }

    WORKSPACE {
        uuid id PK
        uuid tenant_id FK
        string name
        string description
        enum status "active | archived"
        timestamp created_at
    }

    USER {
        uuid id PK
        string email UK
        string name
        string sso_subject UK
        timestamp created_at
    }

    MEMBERSHIP {
        uuid id PK
        uuid workspace_id FK
        uuid user_id FK
        enum role "owner | admin | member | viewer"
        timestamp joined_at
    }

    INVITATION {
        uuid id PK
        uuid workspace_id FK
        string email
        enum role "admin | member | viewer"
        enum status "pending | accepted | expired | revoked"
        string token UK
        timestamp expires_at
        timestamp created_at
    }

    SUBSCRIPTION {
        uuid id PK
        uuid tenant_id FK
        enum plan "free | pro | enterprise"
        int seat_limit
        timestamp current_period_start
        timestamp current_period_end
    }
` ``

## Access Patterns

| Query | Table(s) | Index |
|---|---|---|
| Get all workspaces for a tenant | WORKSPACE | `idx_workspace_tenant_id` |
| Get all members of a workspace | MEMBERSHIP + USER | `idx_membership_workspace_id` |
| Look up user by SSO subject | USER | `uk_user_sso_subject` |
| Find pending invitations by email | INVITATION | `idx_invitation_email_status` |
| Check seat count for tenant | MEMBERSHIP + WORKSPACE | Aggregate via tenant_id |

## Multi-Tenancy Strategy

Row-level isolation using `tenant_id` on all tenant-scoped tables.
PostgreSQL Row-Level Security (RLS) policies enforce isolation at the database layer.
```

---

## api-design.md

API endpoint definitions. Create when the system exposes an API.

```markdown
# API Design

REST API (OpenAPI 3.1). All endpoints require authentication via Bearer JWT
unless marked as public.

## Workspaces

### POST /api/v1/workspaces
Create a new workspace.

**Auth:** `owner` or `admin` role in Tenant
**Request:**
```json
{
  "name": "Engineering",
  "description": "Engineering team workspace"
}
` ``

**Response (201):**
```json
{
  "id": "ws_abc123",
  "name": "Engineering",
  "description": "Engineering team workspace",
  "created_at": "2026-05-12T10:00:00Z"
}
` ``

**Errors:**
| Code | Status | Condition |
|---|---|---|
| `WORKSPACE_NAME_TAKEN` | 409 | Name already exists in Tenant |
| `SEAT_LIMIT_REACHED` | 403 | Tenant subscription seat limit hit |
| `UNAUTHORIZED` | 401 | Missing or invalid JWT |
| `FORBIDDEN` | 403 | Insufficient role |

---

### POST /api/v1/workspaces/:id/invitations
Invite a member to a workspace.

**Auth:** `owner` or `admin` role in Workspace
**Request:**
```json
{
  "email": "jane@example.com",
  "role": "member"
}
` ``

**Response (201):**
```json
{
  "id": "inv_xyz789",
  "email": "jane@example.com",
  "role": "member",
  "status": "pending",
  "expires_at": "2026-05-19T10:00:00Z"
}
` ``

**Errors:**
| Code | Status | Condition |
|---|---|---|
| `ALREADY_MEMBER` | 409 | Email is already a workspace member |
| `SEAT_LIMIT_REACHED` | 403 | Tenant seat limit hit |
| `INVALID_ROLE` | 400 | Role not in allowed list |
```

---

## deployment.md

Infrastructure and deployment topology. Create when ops/SRE alignment matters.

```markdown
# Deployment

How the system runs in production.

## Deployment Topology

```mermaid
flowchart TB
    subgraph AWS["AWS (eu-west-1)"]
        subgraph EKS["EKS Cluster"]
            API["API Server<br/>3 replicas"]
            Worker["Event Worker<br/>2 replicas"]
        end
        ALB["Application<br/>Load Balancer"]
        RDS["RDS PostgreSQL 16<br/>Multi-AZ"]
        SNS["SNS Topics"]
        SQS["SQS Queues"]
        S3["S3<br/>Static assets"]
        CF["CloudFront CDN"]
    end

    User((User)) --> CF --> S3
    User --> ALB --> API
    API --> RDS
    API --> SNS
    SNS --> SQS
    SQS --> Worker
    Worker --> RDS
` ``

## Environment Configuration

| Variable | Description | Example |
|---|---|---|
| `DATABASE_URL` | PostgreSQL connection string | `postgresql://...` |
| `OKTA_ISSUER` | SSO identity provider URL | `https://company.okta.com` |
| `SNS_TOPIC_ARN` | Domain events topic | `arn:aws:sns:eu-west-1:...` |
| `SENDGRID_API_KEY` | Email service key | `SG.xxx` (secret) |
| `LOG_LEVEL` | Logging verbosity | `info` |

## Scaling Strategy

| Component | Strategy | Trigger |
|---|---|---|
| API Server | Horizontal Pod Autoscaler | CPU > 70% |
| Event Worker | KEDA (SQS queue depth) | Queue depth > 100 |
| Database | Vertical scaling + read replicas | Connection count, CPU |
```

---

## ui/ directory

HTML mockups and behavior descriptions. Create when there's a frontend.

### ui/dashboard.html
Static HTML file showing the page layout and structure. Use semantic HTML, minimal inline styling for layout indication.

### ui/dashboard.md

```markdown
# Dashboard Page

## Purpose
Landing page after login. Shows workspace list and recent activity.

## Layout
- **Header:** Logo, workspace switcher, user menu (profile, logout)
- **Sidebar:** Navigation — Dashboard, Members, Settings
- **Main content:** Workspace cards grid (name, member count, last activity)
- **Empty state:** "Create your first workspace" CTA when no workspaces exist

## Interactions
| Element | Action | Result |
|---|---|---|
| Workspace card | Click | Navigate to workspace detail |
| "Create Workspace" button | Click | Open create workspace modal |
| Workspace switcher | Click | Dropdown showing other tenant workspaces |

## States
- **Loading:** Skeleton cards while fetching workspaces
- **Empty:** Illustration + "Create your first workspace" button
- **Populated:** Grid of workspace cards, sorted by last activity
- **Error:** "Failed to load workspaces. Retry" banner

## Responsive Breakpoints
| Breakpoint | Layout |
|---|---|
| Desktop (≥1024px) | 3-column card grid, sidebar visible |
| Tablet (768-1023px) | 2-column card grid, sidebar collapsible |
| Mobile (<768px) | 1-column card list, bottom nav replaces sidebar |
```
