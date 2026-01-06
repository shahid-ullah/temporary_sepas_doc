# 📘 SEPAS Backend – Architecture & Design Documentation

### 1. Overview

    The SEPAS (Smart Energy Planning and Analysis System) Backend is built with FastAPI and PostgreSQL, structured around Clean Architecture principles.

    🔹It follows a layered approach using:

        🔹 Repository Pattern
        🔹 Service Layer Pattern
        🔹 DTO (Data Transfer Object) Pattern
        🔹 Module Pattern (NestJS feature modules)
        🔹 Guard Pattern (Authentication/Authorization)
        🔹 This ensures the system is modular, testable, and scalable.

### 2. Project Structure

```bash
|sepas-backend
|___
    ├── alembic.ini
    ├── apps # Modules (Model, Controllers, Dependency Handlers)
    │   ├── base_app # Module: common application logic
    │   │   ├── __init__.py
    │   │   ├── models.py # common application database tables
    │   │   ├── router.py # common application controllers & Dependency Handlers
    │   │   ├── serializers.py # common application serializer/deserializer
    │   │   └── utils.py # common application utils
    │   ├── nrel_app # Module: NREL service Management
    │   │   ├── __init__.py
    │   │   ├── models.py # NREL service raw data storage tables.
    │   │   ├── router.py # NREL service API's controllers & Dependency Handlers.
    │   │   ├── serializers.py # NREL service data serializer/deserializer
    │   │   └── utils.py # NREL service utils
    │   ├── pypsa_engine # Module: PyPSA https://github.com/PyPSA/PyPSA
    │   │   ├── __init__.py
    │   │   ├── constants.py
    │   │   ├── models.py
    │   │   ├── router.py
    │   │   ├── serializers.py
    │   │   ├── utils.py
    │   │   └── validators.py
    │   └── user_app
    │       ├── __init__.py
    │       ├── models.py
    │       ├── router.py
    │       ├── serializers.py
    │       └── utils.py
    ├── docker-compose.yml
    ├── Dockerfile
    ├── entrypoint.sh
    ├── env.example
    ├── format.ps1
    ├── LICENSE
    ├── list_routes.py
    ├── log.log
    ├── logs
    │   └── app.log
    ├── main.py # Application Entry-point
    ├── migrations
    │   ├── env.py
    │   ├── README
    │   ├── script.py.mako
    │   └── versions
    ├── pyproject.toml
    ├── README.md
    ├── requirements.txt
    ├── ruff.toml
    ├── shell.py
    ├── tags
    ├── tests
    │   ├── nrel_app
    │   │   ├── __init__.py
    │   │   └── test_nrel_solar_wind_fetch.py
    │   ├── test_file_to_df.py
    │   └── user_app
    │       └── test_user_app_routes.py
    ├── utils
    │   ├── __init__.py
    │   ├── components.py
    │   ├── config.py
    │   ├── constants.py
    │   ├── database.py
    │   ├── helpers.py
    │   ├── logging.py
    │   └── validators.py
    └── uv.lock

```

#### Complete Module List

The application consists of **74 modules** listed in the order they appear in `app.module.ts`:

1. `UserModule` - User management
2. `LoggerModule` - Logging functionality
3. `AuthModule` - Authentication & authorization
4. `DomainModule` - Domain management
5. `ObjectiveModule` - Objective management
6. `CapabilityModule` - Capability management
7. `InitiativeModule` - Initiative management
8. `TechClassModule` - Technology class management
9. `TechSubClassModule` - Technology sub-class management
10. `TechnologyModule` - Technology management
11. `SectorModule` - Sector management
12. `ProjectModule` - Project CRUD operations
13. `ProjectCapabilityModule` - Project capability management
14. `ProjectTechnologyModule` - Project technology management
15. `ProjectDomainModule` - Project domain management
16. `ProjectObjectiveModule` - Project objective management
17. `ProjectInitiativeModule` - Project initiative management
18. `ProjectTechnologyClassModule` - Project technology class management
19. `ProjectTechnologySubClassModule` - Project technology sub-class management
20. `OptimizationMatrixModule` - Optimization matrix
21. `ImportDomainModule` - Domain import functionality
22. `ImportTechnologiesModule` - Technology import functionality
23. `ExportTechnologiesModule` - Technology export functionality
24. `ExportDomainModule` - Domain export functionality
25. `ImportCapabilityModule` - Capability import functionality
26. `ImportInitiativesModule` - Initiative import functionality
27. `ExportCapabilityModule` - Capability export functionality
28. `NotesModule` - Notes management
29. `VersionModule` - Version management
30. `CalculateXlsxDataModule` - Excel data calculation
31. `SpiderDiagramModule` - Spider diagram visualization
32. `RoleModule` - Role management
33. `ParameterGroupModule` - Parameter group management
34. `ProjectKpiModule` - Project KPI management
35. `KpiImpactMatrixModule` - KPI impact matrix
36. `YearDurationModule` - Year duration management
37. `ValuePackYearModule` - Value pack year management
38. `ProjectCollaboratorModule` - Project collaborator management
39. `ParameterUnitModule` - Parameter unit management
40. `ParameterPeriodModule` - Parameter period management
41. `KpiModule` - KPI management
42. `ProjectKpiListModule` - Project KPI list management
43. `HourlyRateModule` - Hourly rate management
44. `HourlyRateCoreModule` - Core hourly rate management
45. `ProjectParameterUnitModule` - Project parameter unit management
46. `ProjectParameterPeriodModule` - Project parameter period management
47. `CostModule` - Cost management
48. `PhaseModule` - Phase management
49. `CostSheetModule` - Cost sheet management
50. `ProjectKpiSheetModule` - Project KPI sheet management
51. `InitiativeCostSheetModule` - Initiative cost sheet management
52. `RolloutTableModule` - Rollout table management
53. `ProjectOverviewSheetModule` - Project overview sheet
54. `ProjectBudgetSheetModule` - Project budget sheet management
55. `ProjectSensitivityAnalysisSheetModule` - Sensitivity analysis sheet
56. `CurrencyModule` - Currency management
57. `ExternalAuthModule` - External authentication
58. `ExternalCompanyModule` - External company management
59. `ExternalCompanyProjectsModule` - External company projects
60. `ExternalUserModule` - External user management
61. `ExternalValuePackYearModule` - External value pack year
62. `ExternalOptimizationMatrixModule` - External optimization matrix
63. `ExternalProjectTechnologyModule` - External project technology
64. `ExternalHourlyRateModule` - External hourly rate
65. `ExternalRolloutTableModule` - External rollout table
66. `ExternalProjectBudgetSheetModule` - External project budget sheet
67. `ExternalProjectSensitivityAnalysisSheetModule` - External sensitivity analysis
68. `ExternalOverviewSheetModule` - External overview sheet

**Note:** `DatabaseModule` is a Global module (not imported in `app.module.ts`) that provides Prisma Client wrapper and is available throughout the application.

### 3. Layered Architecture

#### Controller Layer (`src/modules/*/controllers/`)

    🔹 Framework-specific (NestJS controllers).

    🔹 Handles HTTP requests, validation, exception mapping.

    🔹 Uses Swagger decorators for API documentation.

    🔹 Delegates work to services.

    🔹 Example: `project.controller.ts`

#### Service Layer (`src/modules/*/services/`)

    🔹 Implements business logic (e.g., project creation, capability assessment).

    🔹 Calls repositories for persistence.

    🔹 Handles complex workflows and business rules.

    🔹 Example: `ProjectService`

#### Repository Layer (`src/modules/*/repositories/`)

    🔹 Abstracts database access using Prisma Client.

    🔹 Provides CRUD methods, hides Prisma queries from services.

    🔹 Example: `ProjectRepository`

#### Domain Models (`prisma/schema.prisma`)

    🔹 Prisma schema defines entities that represent business objects: Project, Capability, Technology, User, etc.

    🔹 Type-safe database models generated by Prisma.

#### DTOs / Schemas (`src/modules/*/dto.ts`)

    🔹 Define input/output contracts for API.

    🔹 Prevent leaking Prisma models into APIs.

    🔹 Uses class-validator for validation.

    🔹 Example: `CreateProjectDto, UpdateProjectDto, ProjectResponseDto`

#### Common (`src/common/`)

    🔹 Cross-cutting concerns:

        🔹 App configuration (multer.config.ts)

        🔹 Security utilities (auth.constants.ts)

        🔹 JWT authentication (via AuthGuard)

        🔹 Custom decorators (validate.decorator.ts)

        🔹 Validation pipes (validation.pipe.ts)

        🔹 Utility functions (file-upload.util.ts, response-handler.util.ts, etc.)

#### Database Module (`src/modules/database/`)

    🔹 Global module that provides Prisma Client wrapper.

    🔹 Handles database connection lifecycle (onModuleInit, onModuleDestroy).

    🔹 Exported as `DatabaseService` and available throughout the application.

    🔹 Used by all repository classes for database access.

#### Guards (`src/modules/auth/auth.guard.ts`)

    🔹 Authentication and authorization.

    🔹 JWT token validation.

    🔹 Role-based access control.

### 4. Design Patterns Used

#### ✅ Clean Architecture

    🔹 Separation of concerns.

    🔹 Inner layers do not depend on outer layers.

    🔹 Database and NestJS can be swapped with minimal changes.

#### ✅ Repository Pattern

    🔹 Decouples database operations from business logic.

    🔹 Enables testing with fake/in-memory repositories.

    🔹 Uses Prisma Client for type-safe database access.

#### ✅ Service Layer Pattern

    🔹 Encapsulates business rules and workflows.

    🔹 Controllers only delegate work to services.

#### ✅ DTO Pattern (Data Transfer Objects)

    🔹 Uses class-validator for request/response validation.

    🔹 Prevents direct exposure of database models.

#### ✅ Dependency Injection (NestJS DI)

    🔹 Injects services, repositories, and DB via NestJS DI system.

    🔹 Promotes loose coupling.

    🔹 Uses interfaces for abstraction.

#### ✅ Module Pattern

    🔹 Feature-based modules following the exact folder structure (see Complete Module List above).

    🔹 Each module encapsulates related functionality (controller, service, repository, DTOs).

    🔹 Modules can import and export providers for reusability.

    🔹 Global modules (DatabaseModule, UserModule) are available throughout the application.

    🔹 Total of 74 modules covering all application features, listed in the order they appear in `app.module.ts`.

#### ✅ Guard Pattern

    🔹 Authentication guards protect routes.

    🔹 Authorization guards check permissions.

### 5. Validation of Clean Architecture

#### Dependency Rule ✅

    🔹 Services do not import controllers or HTTP-specific code.

    🔹 Controllers depend on Services, not vice versa.

    🔹 Repositories depend on Prisma Client, but Services only on Repositories.

#### Testability ✅

    🔹 Services can be tested with mock repositories.

    🔹 Controllers can be tested with TestClient + mocks.

    🔹 Dependency injection enables easy mocking.

#### Replaceability ✅

    🔹 DB can be swapped (PostgreSQL → MySQL) via Prisma.

    🔹 HTTP adapter can be swapped (Fastify → Express).

    🔹 Services remain unchanged when swapping infrastructure.

### 6. Example Flow (Project CRUD)

```bash
    Request → POST /api/consultant/projects (Controller Layer)
    AuthGuard → Validates JWT token
    Validation → CreateProjectDto (class-validator)
    Service → ProjectService.createProject()
    Repository → ProjectRepository.create() (Prisma Client)
    Database → PostgreSQL stores record
    Response → ProjectResponseDto returned to client
```

### 7. Sequence Diagram (Mermaid)

```bash
    sequenceDiagram
        participant C as Client
        participant G as AuthGuard
        participant A as Controller Layer (ProjectController)
        participant S as Service Layer (ProjectService)
        participant R as Repository (ProjectRepository)
        participant P as Prisma Client
        participant D as Database (PostgreSQL)

        C->>A: POST /api/consultant/projects {CreateProjectDto}
        A->>G: Validate JWT token
        G-->>A: Token valid
        A->>S: validate & call createProject()
        S->>R: call repository.create()
        R->>P: prisma.project.create()
        P->>D: INSERT INTO projects
        D-->>P: return new project row
        P-->>R: return Project entity
        R-->>S: return Project entity
        S-->>A: return ProjectResponseDto
        A-->>C: HTTP 201 {ProjectResponseDto}
```

### 8. Tools for Validation

    🔹 ESLint + Prettier → code quality and formatting

    🔹 Jest → unit & integration testing

    🔹 TypeScript → type safety

    🔹 Swagger / OpenAPI → auto-generate API documentation (available at `/api`)

    🔹 class-validator → DTO validation

    🔹 Prisma → type-safe database access

    🔹 Fastify → High-performance HTTP server (alternative to Express)

    🔹 AWS S3 SDK → File storage integration

    🔹 Winston → Logging framework

    🔹 XLSX → Excel file processing

### 9. Future Extensions

    🔹 GraphQL (NestJS GraphQL module) as alternative delivery layer.

    🔹 Background Workers (Bull/BullMQ + Redis) for async tasks, reports, notifications.

    🔹 Enhanced RBAC (Role-Based Access Control) middleware for fine-grained permissions.

    🔹 Audit Logging for compliance.

    🔹 Caching layer (Redis) for performance optimization.

    🔹 Rate limiting for API protection.

    🔹 WebSocket support for real-time features.

### 10. Diagrams

#### Layered Architecture

### 11. Summary

#### The SEN backend applies Clean Architecture through:

    🔹 Repositories → persistence (Prisma Client)

    🔹 Services → business logic

    🔹 DTOs → input/output validation

    🔹 NestJS controllers → delivery mechanisms

    🔹 Guards → authentication/authorization

    🔹 Modules → feature organization

    🔹 This makes the system modular, maintainable, and scalable.
