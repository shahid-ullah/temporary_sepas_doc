# 📘 SEN Backend – Design Patterns

### 1. Overview

    The SEN backend is designed with Clean Architecture principles to ensure separation of concerns, testability, and scalability.
    Key design patterns applied include:

    🔹 Repository Pattern

    🔹 Service Layer Pattern

    🔹 Data Transfer Object (DTO) Pattern

    🔹 Dependency Injection (NestJS DI)

    🔹 Module Pattern (NestJS feature modules)

    🔹 Guard Pattern (Authentication/Authorization)

    🔹 Factory Pattern (via Prisma & DTOs)

### 2. Repository Pattern

#### 📍 Location: `src/modules/*/repositories/`

#### Intent

    🔹 Abstracts database access.

    🔹 Keeps Prisma operations separate from business logic.

    🔹 Makes persistence layer replaceable (PostgreSQL → MySQL, or Prisma → TypeORM).

#### Example

```typescript
// project.repository.ts

@Injectable()
export class ProjectRepository implements ProjectRepositoryInterface {
  constructor(
    private readonly db: DatabaseService,
    private logger: Logger,
  ) {}

  async findById(id: number): Promise<Project | null> {
    return await this.db.project.findUnique({
      where: { id },
    });
  }

  async create(data: Prisma.ProjectCreateInput): Promise<Project> {
    return await this.db.project.create({
      data,
    });
  }

  async update(id: number, data: Prisma.ProjectUpdateInput): Promise<Project> {
    return await this.db.project.update({
      where: { id },
      data,
    });
  }
}
```

✅ Benefit: Business logic (ProjectService) never deals with Prisma directly.

### 3. Service Layer Pattern

#### 📍 Location: `src/modules/*/services/`

#### Intent

    🔹 Encapsulates business rules and workflows.

    🔹 Coordinates repositories, applies validation, raises business errors.

    🔹 Keeps controllers thin.

#### Example

```typescript
// project.service.ts

@Injectable()
export class ProjectService {
  constructor(
    @Inject('ProjectRepositoryInterface')
    private readonly projectRepository: ProjectRepositoryInterface,
    private readonly logger: Logger,
  ) {}

  async createProject(data: CreateProjectDto): Promise<Project> {
    // Business logic: check if project name already exists
    const existing = await this.projectRepository.findByName(data.name);
    if (existing) {
      throw new ConflictException('Project with this name already exists');
    }

    // Business logic: apply default values
    const projectData = {
      ...data,
      status: data.status || 'DRAFT',
      createdAt: new Date(),
    };

    return await this.projectRepository.create(projectData);
  }
}
```

✅ Benefit: The same service can be reused by different controllers or future GraphQL resolvers.

### 4. DTO (Data Transfer Object) Pattern

#### 📍 Location: `src/modules/*/dto.ts`

#### Intent

    🔹 Decouples external API contracts from internal models.

    🔹 Prevents Prisma entities from leaking into API.

    🔹 Provides strong validation using class-validator.

#### Example

```typescript
// project.dto.ts

import { IsString, IsOptional, IsEnum, MinLength } from 'class-validator';
import { ApiProperty } from '@nestjs/swagger';

export class CreateProjectDto {
  @ApiProperty({ description: 'Project name' })
  @IsString()
  @MinLength(3)
  name: string;

  @ApiProperty({ description: 'Project description', required: false })
  @IsOptional()
  @IsString()
  description?: string;

  @ApiProperty({ description: 'Project status', enum: ProjectStatus })
  @IsEnum(ProjectStatus)
  status: ProjectStatus;
}

export class ProjectResponseDto {
  id: number;
  name: string;
  description: string;
  status: ProjectStatus;
  createdAt: Date;
  updatedAt: Date;
}
```

✅ Benefit: REST API contracts are clearly defined and validated.

### 5. Dependency Injection

#### 📍 Location: Used throughout NestJS modules.

#### Intent

    🔹 Injects dependencies (DB service, authenticated user, services) instead of hardcoding them.

    🔹 Improves testability & loose coupling.

    🔹 Uses NestJS built-in DI container.

#### Example

```typescript
// project.module.ts

@Module({
  imports: [DatabaseModule],
  controllers: [ProjectController],
  providers: [
    {
      provide: 'ProjectRepositoryInterface',
      useClass: ProjectRepository,
    },
    {
      provide: 'ProjectServiceInterface',
      useClass: ProjectService,
    },
  ],
  exports: ['ProjectServiceInterface'],
})
export class ProjectModule {}

// project.controller.ts

@Controller('/api/consultant/projects')
export class ProjectController {
  constructor(
    @Inject('ProjectServiceInterface')
    private readonly projectService: ProjectServiceInterface,
  ) {}

  @Post()
  async create(@Body() createDto: CreateProjectDto) {
    return await this.projectService.createProject(createDto);
  }
}
```

✅ Benefit: Swap implementations (e.g., mock repository) in tests easily.

### 6. Module Pattern

#### 📍 Location: `src/modules/*/`

#### Intent

    🔹 Organizes code by feature.

    🔹 Encapsulates related functionality (controller, service, repository).

    🔹 Enables module imports/exports for reusability.

#### Example

```typescript
// project.module.ts

@Module({
  imports: [CostModule, DatabaseModule],
  controllers: [ProjectController],
  providers: [
    ProjectService,
    ProjectRepository,
    // ... other providers
  ],
  exports: ['ProjectServiceInterface', 'ProjectRepositoryInterface'],
})
export class ProjectModule {}
```

✅ Benefit: Clear feature boundaries, easy to understand and maintain.

### 7. Guard Pattern

#### 📍 Location: `src/modules/auth/auth.guard.ts`

#### Intent

    🔹 Protects routes with authentication/authorization.

    🔹 Validates JWT tokens.

    🔹 Checks user roles and permissions.

#### Example

```typescript
// auth.guard.ts

@Injectable()
export class AuthGuard implements CanActivate {
  constructor(
    private jwtService: JwtService,
    private userService: UserService,
  ) {}

  async canActivate(context: ExecutionContext): Promise<boolean> {
    const request = context.switchToHttp().getRequest();
    const token = this.extractTokenFromHeader(request);

    if (!token) {
      throw new UnauthorizedException();
    }

    try {
      const payload = await this.jwtService.verifyAsync(token);
      request.user = await this.userService.findById(payload.sub);
    } catch {
      throw new UnauthorizedException();
    }

    return true;
  }
}

// Usage in controller
@Controller('/api/consultant/projects')
@UseGuards(AuthGuard)
export class ProjectController {
  // Protected routes
}
```

✅ Benefit: Centralized authentication logic, reusable across all protected routes.

### 8. Factory Pattern (Prisma & DTOs)

    🔹 Prisma Client → acts as factory for type-safe database queries.

    🔹 Prisma Migrate → generates migration scripts (factory for schema evolution).

    🔹 DTOs → act as factories for validated domain objects.

### 9. Benefits of This Architecture

    🔹 Maintainability → clear boundaries between controller, service, and repository layers.

    🔹 Testability → services tested with mock repositories.

    🔹 Flexibility → support REST API with potential for GraphQL without duplicating logic.

    🔹 Replaceability → swap DB (PostgreSQL → MySQL), or HTTP adapter (Fastify → Express).

    🔹 Scalability → easy to add new modules (new features, external integrations, etc.).

    🔹 Type Safety → TypeScript + Prisma provide end-to-end type safety.

### 10. Module Structure Example

Each feature module typically follows this structure:

```bash
    📂 project/
    ┣ 📄 project.module.ts          # Module definition
    ┣ 📄 project.controller.ts      # HTTP endpoints
    ┣ 📄 project.service.ts         # Business logic
    ┣ 📄 project.repository.ts      # Data access
    ┣ 📄 project.dto.ts             # Data Transfer Objects
    ┣ 📄 project.interface.ts       # TypeScript interfaces
    ┣ 📄 project.validation.ts      # Validation schemas (Zod)
    ┣ 📄 project.controller.spec.ts # Controller tests
    ┣ 📄 project.service.spec.ts    # Service tests
    ┗ 📄 project.repository.spec.ts # Repository tests
```

### 11. Summary

    🔹 The SEN backend applies Clean Architecture through a combination of patterns:

    🔹 Repositories handle persistence (Prisma Client).

    🔹 Services encapsulate business rules.

    🔹 DTOs define communication contracts.

    🔹 Dependency Injection ensures flexibility.

    🔹 Modules organize features.

    🔹 Guards protect routes.

    🔹 Together, these patterns make the system modular, robust, and future-proof.
