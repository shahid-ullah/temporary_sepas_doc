# SEPAS Backend – Architecture & Developer Guide

## 1. Purpose

This document provides a **high-level overview** of the **SEPAS (Smart Energy Planning and Analysis System) backend**, the architectural principles behind its design, and clear guidance on how developers should **understand, extend, and maintain** the system safely.

The goal of this documentation is to ensure:

- Architectural consistency
- Clear domain boundaries
- Predictable extension patterns
- Long-term maintainability

## 2. Tech Stack

- **Language:** Python 3.12+
- **Framework:** FastAPI
- **Database:** PostgreSQL (SQLAlchemy ORM + Alembic)
- **API Style:** REST
- **Authentication:** JWT (JSON Web Tokens)
- **Validation:** Pydantic
- **Testing:** Pytest
- **Documentation:** Swagger / OpenAPI (`/docs`)
- **Deployment:** Docker, Docker Compose
- **File Storage:** AWS S3
- **Linting & Formatting:** Ruff

## 3. Architectural Overview

SEPAS is implemented as a **Modular Monolith** using a **feature-based application architecture**, inspired by Django’s app structure.

Instead of enforcing strict Clean Architecture, the system adopts a **pragmatic, domain-oriented design** that balances clarity, scalability, and simplicity.

### High-Level Request Flow

```text
Client
→ FastAPI Router
→ Pydantic Validation
→ Domain Logic (module utils)
→ SQLAlchemy ORM
→ PostgreSQL
→ Serialized Response
```

## 4. Project Structure

```text
sepas-backend/
├── apps/                 # Feature-based application modules
│   ├── user_app/
│   ├── nrel_app/
│   ├── pypsa_engine/
│   └── base_app/
│
├── utils/                # Shared cross-cutting utilities
├── migrations/           # Alembic/Database migrations
├── tests/                # Module-aligned test suites
├── main.py               # Application entry point
├── Dockerfile
├── docker-compose.yml
└── requirements.txt
```

## 5. Feature-Based Application Modules

Each folder under `apps/` represents a **self-contained domain module**.

### Example Module Structure

```text
apps/user_app/
├── models.py        # SQLAlchemy ORM models
├── router.py        # FastAPI routes & dependencies
├── serializers.py   # Pydantic schemas (DTOs)
├── utils.py         # Domain logic & helpers
```

### Design Characteristics

- Each module owns its **routes, models, and validation**
- Modules do not directly depend on each other
- Shared logic is extracted into `utils/`
- Clear boundaries reduce coupling and simplify testing

## 6. Layered Design (Within a Module)

Each module follows a **local layered architecture**:

### Router Layer (`router.py`)

- Defines HTTP endpoints
- Handles dependency injection
- Orchestrates request flow
- Contains minimal business logic

### Serializer / DTO Layer (`serializers.py`)

- Defines request and response schemas
- Performs validation using Pydantic
- Prevents ORM leakage into APIs

### Model Layer (`models.py`)

- SQLAlchemy ORM models
- Database persistence mapping
- No FastAPI or HTTP dependencies

### Domain Logic (`utils.py`, `validators.py`)

- Business rules
- Computation and transformations
- Reusable across endpoints

## 7. Shared Infrastructure (`utils/`)

The `utils/` directory contains **cross-cutting concerns** used by all modules:

- Database session management
- Configuration loading
- Logging setup
- Shared validators
- Common helpers and constants

This acts as a **shared kernel**, kept intentionally small and stable.

## 8. Authentication & Security

- Authentication is handled using **JWT**
- Auth dependencies are injected via FastAPI `Depends`
- Protected routes explicitly declare authentication requirements
- No hidden global state

## 9. Testing Strategy

Tests mirror the feature-based structure:

```text
tests/
├── user_app/
├── nrel_app/
```

### Benefits

- Clear ownership of tests
- Easy mocking of dependencies
- Fast feedback during development
- Safer refactoring

## 10. Adding a New Feature (Recommended Workflow)

1. Create a new module under `apps/`
2. Define database models in `models.py`
3. Generate a migration using Alembic
4. Add Pydantic schemas in `serializers.py`
5. Implement routes in `router.py`
6. Add domain logic in `utils.py`
7. Register the router in `main.py`
8. Add tests under `tests/<module_name>/`

This workflow keeps features isolated and consistent.

## 11. API Documentation

- Swagger UI is available at `/docs`
- OpenAPI schema is auto-generated
- All request/response models are documented via Pydantic

## 12. What This Architecture Is (and Is Not)

### ✅ Is

- Modular Monolith
- Feature-based
- Domain-oriented
- FastAPI-native

### ❌ Is Not

- Microservices
- Strict Clean Architecture
- Over-abstracted service/repository layers
- Framework-driven boilerplate

These are **intentional trade-offs** to reduce complexity while preserving scalability.

## 13. Future Improvements

Potential future enhancements include:

- Enhanced RBAC (role & permission-based access)
- Caching layer (Redis)
- Rate limiting
- Event-driven integrations
- Selective module extraction into services if needed

## 14. Summary

The SEPAS backend is a **FastAPI-based modular monolith** designed around **feature-based application modules**.

Key strengths of this design:

- Clear domain boundaries
- Predictable extension patterns
- High testability
- Low operational complexity
- Long-term maintainability

This architecture provides a **solid foundation** for evolving SEPAS as system requirements grow.
