# NestJS — Pipes, Guards, Interceptors, and Exception Filters

The four pieces that hook into a request's path before/after reaching the controller — each solves a different question, and they're often confused with each other. The real order in which they act on an incoming request:

```
Request → Guards → Interceptors (before) → Pipes → Controller Handler → Interceptors (after) → Response
                                                          ↓ (if something throws an exception at any point)
                                                   Exception Filters
```

## DTOs and Pipes — validating/transforming input data

A **DTO** (Data Transfer Object) defines the expected shape of a request's body/params. A **Pipe** is what actually validates (or transforms) that data against that shape before it reaches the handler.

```typescript
// create-user.dto.ts
export class CreateUserDto {
  @IsString()
  name: string;

  @IsEmail()
  email: string;

  @IsInt() @Min(18)
  age: number;
}

// users.controller.ts
@Post()
create(@Body() dto: CreateUserDto) {   // ValidationPipe runs BEFORE this executes
  return this.usersService.create(dto);
}
```

```typescript
// enabled globally in main.ts — automatically validates every DTO marked with class-validator decorators
app.useGlobalPipes(new ValidationPipe());
```

If the body doesn't satisfy the DTO (missing `email`, `age` isn't a number), `ValidationPipe` cuts it off right there — the controller doesn't even find out there was an invalid request.

## Guards — authentication and authorization

Decide **whether the request can continue** or not, before reaching the handler — the answer is a boolean.

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return Boolean(request.headers.authorization);   // true = continues, false = automatic 403
  }
}

@UseGuards(AuthGuard)
@Get('profile')
getProfile() { /* ... */ }
```

## Interceptors — behavior before AND after the handler

Unlike Pipes/Guards (which only act before), an Interceptor wraps the handler's entire execution — it can run logic before it runs, and transform what it returns afterward.

```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('Before the handler...');
    const start = Date.now();

    return next.handle().pipe(
      tap(() => console.log(`After the handler: ${Date.now() - start}ms`))
    );
  }
}
```

Typical uses: logging, adding metadata to the response, measuring times, transforming the response's shape (e.g. wrapping everything in `{ data: ... }`).

## Exception Filters — turning exceptions into HTTP responses

Catch any exception thrown in Guards, Interceptors, Pipes, or the Controller itself, and decide what to respond — without this, an unhandled exception ends up as a generic 500 with no control over the format.

```typescript
@Catch(HttpException)
export class HttpExceptionFilter implements ExceptionFilter {
  catch(exception: HttpException, host: ArgumentsHost) {
    const response = host.switchToHttp().getResponse();
    const status = exception.getStatus();

    response.status(status).json({
      statusCode: status,
      message: exception.message,
      timestamp: new Date().toISOString(),
    });
  }
}
```

## How not to mix them up

| Piece | Answers | When it runs |
|---|---|---|
| **Guard** | Can this request continue? | Before everything else |
| **Interceptor** | What do I do before AND after the handler? | Wraps the whole handler |
| **Pipe** | Does the input data have the right shape? | Right before the handler, on the parameters |
| **Exception Filter** | What do I respond if any of the above fails? | Only if an exception was thrown |

---
Related: [NestJS Architecture](architecture.md), [Authentication and Security](../../backend/authentication.md), [Endpoints for microservices (FastAPI)](../fastapi/microservice-endpoints.md#3-input-validation-with-pydantic) (Pydantic plays the same role as a DTO + Pipe, in Python).
