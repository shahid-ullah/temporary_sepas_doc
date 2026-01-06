# SEPAS Backend – Architecture & Design Documentation

## 1. Overview

The **SEPAS (Smart Energy Planning and Analysis System) Backend** is built with **FastAPI** and **PostgreSQL**, designed as a **Modular Monolith** using a **feature-based application architecture**, inspired by Django’s app structure.

Rather than strict Clean Architecture, the system adopts:

- **Feature-Based Modular Architecture (Django-style apps)**
- **Layered Architecture within each module**
- **Strong domain boundaries with high internal cohesion**

This approach ensures the system is:

- Modular and easy to extend
- Testable at module boundaries
- Scalable without premature microservices

---

## 2. Architectural Style

### Primary Architecture

✅ **Modular Monolith**

- Single deployable FastAPI application
- Internally composed of multiple independent application modules

### Secondary Architecture

✅ **Feature-Based Application Architecture**

- Each feature/domain is implemented as a self-contained module under `apps/`
- Similar to Django’s `INSTALLED_APPS` model

### Internal Structure

✅ **Layered Architecture per Module**

Each application module contains:

- API layer (routers)
- Data validation layer (serializers)
- Persistence layer (models)
- Domain-specific utilities and validators

---

## 3. Project Structure

```bash
sepas-backend/
├── alembic.ini
├── apps/                     # Feature-based application modules
│   ├── base_app/             # Shared application logic
│   ├── nrel_app/             # NREL service integration
│   ├── pypsa_engine/         # PyPSA integration & energy modeling
│   └── user_app/             # User management
│
├── utils/                    # Cross-cutting shared utilities
├── migrations/               # Alembic migrations
├── tests/                    # Module-aligned test suites
├── main.py                   # FastAPI application entry point
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

---

## 4. Application Modules (Feature-Based)

Each folder under `apps/` represents a **self-contained application domain**.

### Example Module Structure

```bash
apps/user_app/
├── models.py        # SQLAlchemy ORM models
├── router.py        # FastAPI routers & dependency handlers
├── serializers.py   # Pydantic request/response schemas
├── utils.py         # Domain-specific helpers
```

### Key Characteristics

- Modules do **not depend on each other directly**
- Shared logic lives in `utils/`
- Each module owns its:
  - Database models
  - API routes
  - Validation schemas
  - Business helpers

---

## 5. Layered Architecture (Within a Module)

### Router Layer (`apps/*/router.py`)

- FastAPI `APIRouter`
- Handles:
  - HTTP routing
  - Dependency injection
  - Request/response handling

- No heavy business logic

---

### Serializer Layer (`apps/*/serializers.py`)

- Pydantic models
- Responsible for:
  - Input validation
  - Output serialization

- Acts as DTOs (Data Transfer Objects)

---

### Model Layer (`apps/*/models.py`)

- SQLAlchemy ORM models
- Maps Python objects to database tables
- No HTTP or request logic

---

### Utility & Validation Layer (`apps/*/utils.py`, `validators.py`)

- Domain-specific helpers
- Reusable calculations
- Validation rules
- External service integrations

---

## 6. Shared Infrastructure (`utils/`)

The `utils/` directory contains **cross-cutting concerns** shared across modules:

- Database session management
- Application configuration
- Logging setup
- Global validators
- Shared constants

This prevents duplication and keeps feature modules focused.

---

## 7. Request Lifecycle Example

### Example: Create User

```text
Client
→ FastAPI Router (user_app/router.py)
→ Pydantic Validation (serializers.py)
→ Domain Logic (utils.py)
→ SQLAlchemy Model (models.py)
→ Database Commit
→ Serialized Response
```

---

## 8. Dependency Management & Injection

FastAPI’s dependency injection system (`Depends`) is used for:

- Database sessions
- Authentication context
- Shared services

Dependencies are:

- Request-scoped
- Easily mockable
- Explicitly declared

---

## 9. Testing Strategy

Tests mirror the application structure:

```bash
tests/
├── user_app/
├── nrel_app/
```

### Benefits

- Clear module ownership
- Easy isolation
- Faster debugging
- High confidence refactoring

---

## 10. Design Patterns Used

### ✅ Modular Monolith

- Single deployable unit
- Strong internal boundaries
- Easier operational management than microservices

---

### ✅ Feature-Based Module Pattern

- Code organized by domain, not by technical layer
- High cohesion within modules
- Low coupling between modules

---

### ✅ Layered Architecture (Local)

- Routers → Serializers → Models
- Clear separation of responsibilities
- Easy onboarding for new developers

---

### ✅ DTO Pattern (Pydantic)

- Explicit request/response contracts
- Prevents ORM leakage
- Automatic validation

---

### ✅ Dependency Injection (FastAPI)

- Function-based DI using `Depends`
- Improves testability
- No global state

---

## 11. What This Architecture Is NOT

❌ Not Microservices
❌ Not Strict Clean Architecture
❌ Not Hexagonal / Onion Architecture

> This is an intentional design choice to reduce complexity while preserving modularity.

---

## 12. Future Evolution Path

If needed, the system can evolve toward:

- Extracting modules into independent services
- Introducing async task workers
- Adding GraphQL as an alternative API layer
- Introducing stricter domain-service separation

---

## 13. Summary

The SEPAS backend is a **FastAPI-based modular monolith** that uses:

- Django-inspired feature-based applications
- Local layered architecture per module
- Explicit dependency injection
- Strong domain ownership

This architecture provides an optimal balance between **clarity, scalability, and maintainability**, while avoiding unnecessary complexity.
