# 📘 SEN Backend – Architecture Guide

### 1. Purpose

    This document provides a high-level overview of the SEN (Strategic Enterprise Navigator) backend architecture, the principles behind its design, and how developers should extend or modify the system safely.

### 2. Tech Stack

    🔹 Language: TypeScript 5.6+

    🔹 Framework: NestJS 10+

    🔹 Database: PostgreSQL (Prisma ORM + Prisma Migrate)

    🔹 API: REST (Fastify adapter)

    🔹 Auth: JWT (JSON Web Tokens) with Guards

    🔹 Testing: Jest

    🔹 Validation: class-validator, Zod

    🔹 Documentation: Swagger/OpenAPI

    🔹 Deployment: Docker, Docker Compose

    🔹 File Storage: AWS S3

### 3. Clean Architecture Layers

```bash
   Client → Controller Layer → Service Layer → Repository Layer → Prisma Client → PostgreSQL
```

#### Controller Layer (`src/modules/*/controllers/`)

    🔹 REST controllers (NestJS controllers).

    🔹 Handles HTTP requests/responses.

    🔹 Validates inputs using DTOs and decorators.

    🔹 Delegates to services.

    🔹 Never contains business logic.

#### Service Layer (`src/modules/*/services/`)

    🔹 Encapsulates business rules.

    🔹 Talks to repositories, raises business errors.

    🔹 Can be tested independently of controllers or DB.

#### Repository Layer (`src/modules/*/repositories/`)

    🔹 Handles persistence using Prisma Client.

    🔹 Provides CRUD methods.

    🔹 Services depend on these abstractions.

#### Domain Models (`prisma/schema.prisma`)

    🔹 Prisma schema defines database models.

    🔹 Represents core SEN business objects (Project, Capability, Technology, etc.).

#### DTOs (`src/modules/*/dto.ts`)

    🔹 Data Transfer Objects used for request/response validation.

    🔹 Uses class-validator decorators.

    🔹 Prevent ORM leakage into APIs.

#### Common (`src/common/`)

    🔹 Cross-cutting concerns (config, constants, decorators, pipes, utils).

    🔹 Guards for authentication/authorization.

    🔹 Utility functions.

### 4. Design Principles

    🔹 Clean Architecture: independent layers, no inward dependencies.

    🔹 Repository Pattern: isolate DB logic.

    🔹 Service Layer Pattern: centralize business rules.

    🔹 DTO Pattern: define clear input/output contracts.

    🔹 Dependency Injection: NestJS built-in DI system.

    🔹 Module Pattern: feature-based modules for organization.

    🔹 Guard Pattern: authentication and authorization.

### 5. Request Flow Example (Create Project)

    1. REST API receives request POST /api/consultant/projects.

    2. AuthGuard validates JWT token.

    3. Controller validates payload using DTO (CreateProjectDto).

    4. Service Layer (ProjectService) applies business rules.

    5. Repository persists entity via Prisma Client.

    6. Prisma maps to PostgreSQL.

    7. Response DTO is returned.

### 6. Adding a New Feature (Example: New Module)

    1.  Add Prisma model → `prisma/schema.prisma`.

    2.  Create migration →
    ```bash
        npx prisma migrate dev --name add_new_feature
    ```

    3.  Generate Prisma Client →
    ```bash
        npx prisma generate
    ```

    4.  Create repository → `src/modules/new-module/new-module.repository.ts`.

    5.  Create service → `src/modules/new-module/new-module.service.ts`.

    6.  Add DTOs → `src/modules/new-module/new-module.dto.ts`.

    7.  Add controller → `src/modules/new-module/new-module.controller.ts`.

    8.  Create module → `src/modules/new-module/new-module.module.ts`.

    9.  Register module in `app.module.ts`.

### 7. Diagrams

#### Layered Architecture

### 8. Validation Tools

    🔹 ESLint + Prettier → code quality and formatting.

    🔹 Jest → test services and controllers.

    🔹 TypeScript → type safety.

    🔹 Swagger → auto-generated API docs at `/api`.

    🔹 class-validator → DTO validation.

### 9. Future Improvements

    🔹 GraphQL support (NestJS GraphQL module).

    🔹 Background workers (Bull/BullMQ + Redis) for async tasks.

    🔹 Enhanced RBAC (role + permission middleware).

    🔹 Audit logging for compliance.

    🔹 Caching layer (Redis).

    🔹 Rate limiting.
