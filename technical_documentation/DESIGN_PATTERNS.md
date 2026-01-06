# SEPAS Backend – Design Patterns

## 1. Overview

The **SEPAS Backend** follows a **FastAPI-based Modular Monolith architecture** built using **feature-based application modules.**

Rather than enforcing strict Clean Architecture, SEPAS applies **pragmatic, domain-oriented design patterns** that emphasize:

- High cohesion within features
- Explicit domain ownership
- Clear request lifecycle
- Ease of testing and extension

### Core Design Patterns Used

- Feature-Based Module Pattern
- Modular Monolith
- Local Layered Architecture
- DTO Pattern (Pydantic)
- Dependency Injection (FastAPI `Depends`)
- Router Pattern
- Shared Kernel (Common Utilities)

---

## 2. Feature-Based Module Pattern (Primary Pattern)

### Location: `apps/*`

### Intent

- Organize code by **business domain**, not technical layer
- Encapsulate models, routes, validation, and helpers per feature
- Reduce cross-module coupling
- Enable future extraction into microservices if needed

### Structure

```bash
apps/user_app/
├── models.py        # ORM models
├── router.py        # FastAPI routes & dependencies
├── serializers.py   # Pydantic schemas (DTOs)
├── utils.py         # Domain logic & helpers
```

### Benefits

High cohesion
Clear ownership
Easier onboarding

## 3. Modular Monolith Pattern

### Intent

- Maintain a **single deployable unit**
- Avoid premature microservices
- Keep operational complexity low

### Characteristics

- One FastAPI application
- Independent internal modules
- Shared infrastructure (DB, logging, config)

> This architecture balances **scalability and simplicity** while keeping the system evolvable.

---

## 4. Local Layered Architecture (Within Each Module)

Each application module follows a **local layered structure**:

```text
Router → Serializer → Model → Utility
```

### Router Layer (`router.py`)

- FastAPI `APIRouter`
- Handles:
  - HTTP requests
  - Dependency injection
  - Authentication hooks

- No persistence or heavy business logic

```python
@router.post("/users")
def create_user(
    payload: UserCreate,
    db: Session = Depends(get_db),
):
    return create_user_logic(db, payload)
```

---

### Serializer / DTO Pattern (`serializers.py`)

### Intent

- Define explicit request/response contracts
- Validate input data
- Prevent ORM leakage into API responses

```python
class UserCreate(BaseModel):
    email: EmailStr
    password: str


class UserResponse(BaseModel):
    id: int
    email: EmailStr
```

Clear API contracts
Automatic validation
Framework-agnostic schemas

---

### Model Layer (`models.py`)

### Intent

- Map Python objects to database tables
- Encapsulate persistence logic

```python
class User(Base):
    __tablename__ = "users"

    id = Column(Integer, primary_key=True)
    email = Column(String, unique=True)
```

Models:

- Do not import FastAPI
- Do not handle request logic

---

### Utility / Domain Logic Layer (`utils.py`, `validators.py`)

### Intent

- Encapsulate domain-specific logic
- Keep routers thin
- Reuse logic across routes

```python
def hash_password(password: str) -> str:
    return pwd_context.hash(password)
```

---

## 5. Dependency Injection Pattern (FastAPI)

### Intent

- Explicitly declare dependencies
- Improve testability
- Avoid global state

### Example

```python
def get_db():
    db = SessionLocal()
    try:
        yield db
    finally:
        db.close()
```

```python
@router.get("/users")
def list_users(db: Session = Depends(get_db)):
    return db.query(User).all()
```

### Benefits

Request-scoped dependencies
Easy mocking in tests
No hidden wiring

---

## 6. Router Pattern

### Location: `apps/*/router.py`

### Intent

- Group endpoints by domain
- Keep HTTP concerns isolated
- Enable easy versioning

```python
router = APIRouter(prefix="/users", tags=["Users"])
```

Routers are registered centrally in `main.py`.

---

## 7. Shared Kernel Pattern

### Location: `utils/`

### Intent

- Centralize cross-cutting concerns
- Avoid duplication across modules

### Contents

- Database session management
- Logging configuration
- Application settings
- Shared validators
- Global helpers

This acts as a **Shared Kernel**, carefully kept small and stable.

---

## 8. What This Architecture Avoids (Intentionally)

Strict Clean Architecture
Service / Repository over-abstraction
Microservices sprawl
Framework-driven structure

> Patterns are applied **only when they add value**.

---

## 9. Testing-Aligned Design

Tests mirror feature modules:

```bash
tests/
├── user_app/
├── nrel_app/
```

### Benefits

- Clear scope per test
- Easy mocking of dependencies
- Domain-focused assertions

---

## 10. Benefits of This Design

- **Maintainability** – changes stay within modules
- **Testability** – FastAPI DI enables easy mocking
- **Clarity** – code organized by domain, not layers
- **Scalability** – modules can be extracted later
- **Pragmatism** – avoids unnecessary abstractions

---

## 11. Summary

The SEPAS backend applies a **feature-based modular design** implemented in FastAPI.

Key takeaways:

- Feature-based modules are the primary organizing unit
- Each module uses a local layered structure
- Pydantic DTOs define API contracts
- FastAPI dependency injection ensures flexibility
- The system is a modular monolith, not microservices

This design provides a **clean, scalable, and pragmatic foundation** for long-term development.
