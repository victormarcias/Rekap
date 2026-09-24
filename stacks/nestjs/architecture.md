# NestJS — Architecture

A Node framework built on top of Express (or Fastify, interchangeable) — unlike Express, which gives total control but no structure, NestJS imposes an opinionated architecture inspired by Angular (modules, decorators, Dependency Injection).

## NestJS vs Express

- **Native TypeScript**: Express is plain JS with optional types bolted on top; NestJS is designed in TypeScript from the framework's very design.
- **Imposed architecture**: Express doesn't tell you how to organize your code — every project ends up with its own convention. NestJS forces a structure (Modules → Controllers → Services) from the very first file.
- **Built-in Dependency Injection**: Express has no native DI — you have to build it by hand or with a separate library. In NestJS it's part of the framework's core.
- **Testing out of the box**: NestJS ships its own testing module (`@nestjs/testing`) built in, designed to instantiate isolated modules/services in unit tests.

Neither is "better" in the abstract: Express gives more freedom and less friction for something small; NestJS pays the cost of a steeper learning curve in exchange for a consistent architecture that scales better across large teams.

## Modules, Controllers, Services

```typescript
// users.module.ts — groups everything related to "users" into one unit
@Module({
  controllers: [UsersController],
  providers: [UsersService],   // services available for DI within this module
})
export class UsersModule {}

// users.controller.ts — receives the HTTP request, delegates the logic to the service
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}   // DI: received, not instantiated by hand

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);   // the controller doesn't know HOW it's resolved, just delegates
  }
}

// users.service.ts — the real business logic
@Injectable()
export class UsersService {
  findOne(id: string) {
    // the real query, the business rule, etc. go here
    return { id, name: 'Ana' };
  }
}
```

- **Module**: groups a section of the backend (users, orders, auth) — declares which controllers and providers belong there.
- **Controller**: receives the request, extracts the data (`@Param`, `@Body`, `@Query`), delegates to the service — contains no business logic.
- **Service**: runs the real business logic — the controller uses it, never the other way around.

## Dependency Injection

A class doesn't instantiate its own dependencies (`new UsersService()`) — it **receives** them from outside, resolved automatically by the framework from the constructor.

```typescript
@Injectable()   // marks the class as "injectable" — NestJS's container can resolve it
export class UsersService {
  constructor(private readonly db: DatabaseService) {}   // NestJS resolves and injects DatabaseService on its own
}
```

**What problem it solves**: without DI, every class that needs `DatabaseService` would have to create its own instance (`new DatabaseService()`) or receive it by hand at every point in the call chain. With DI, any class simply **declares** it in its constructor, and the framework takes care of building it and passing it in — easier to test (in a test, you can inject a mock/fake instead of the real implementation) and to replace without touching the code that uses it.

---
Related: [Pipes, Guards, Interceptors, and Exception Filters](pipes-guards-interceptors-filters.md), [Controller / Service / Repository](../../backend/controller-service-repository.md) (same pattern, framework-agnostic), [Endpoints for microservices (FastAPI)](../fastapi/microservice-endpoints.md#4-dependency-injection-with-depends) (same DI concept, in Python).
