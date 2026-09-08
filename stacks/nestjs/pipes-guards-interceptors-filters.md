# NestJS — Pipes, Guards, Interceptors y Exception Filters

Las cuatro piezas que se enganchan en el camino de una request antes/después de llegar al controller — cada una resuelve una pregunta distinta, y se confunden seguido entre sí. Orden real en el que actúan sobre una request entrante:

```
Request → Guards → Interceptors (antes) → Pipes → Controller Handler → Interceptors (después) → Response
                                                          ↓ (si algo tira una excepción en cualquier punto)
                                                   Exception Filters
```

## DTOs y Pipes — validar/transformar los datos de entrada

Un **DTO** (Data Transfer Object) define la forma esperada del body/params de una request. Un **Pipe** es lo que efectivamente valida (o transforma) esos datos contra esa forma antes de que lleguen al handler.

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
create(@Body() dto: CreateUserDto) {   // ValidationPipe corre ANTES de que esto se ejecute
  return this.usersService.create(dto);
}
```

```typescript
// habilitado globalmente en main.ts — valida automáticamente todo DTO marcado con decorators de class-validator
app.useGlobalPipes(new ValidationPipe());
```

Si el body no cumple el DTO (falta `email`, `age` no es un número), el `ValidationPipe` corta ahí — el controller ni se entera de que hubo una request inválida.

## Guards — autenticación y autorización

Deciden **si la request puede continuar** o no, antes de llegar al handler — la respuesta es un booleano.

```typescript
@Injectable()
export class AuthGuard implements CanActivate {
  canActivate(context: ExecutionContext): boolean {
    const request = context.switchToHttp().getRequest();
    return Boolean(request.headers.authorization);   // true = sigue, false = 403 automático
  }
}

@UseGuards(AuthGuard)
@Get('perfil')
getPerfil() { /* ... */ }
```

## Interceptors — comportamiento antes Y después del handler

A diferencia de Pipes/Guards (que solo actúan antes), un Interceptor envuelve la ejecución completa del handler — puede correr lógica antes de que se ejecute, y transformar lo que devuelve después.

```typescript
@Injectable()
export class LoggingInterceptor implements NestInterceptor {
  intercept(context: ExecutionContext, next: CallHandler) {
    console.log('Antes del handler...');
    const inicio = Date.now();

    return next.handle().pipe(
      tap(() => console.log(`Después del handler: ${Date.now() - inicio}ms`))
    );
  }
}
```

Usos típicos: logging, agregar metadata a la respuesta, medir tiempos, transformar la forma de la respuesta (ej. envolver todo en `{ data: ... }`).

## Exception Filters — transformar excepciones en respuestas HTTP

Atrapan cualquier excepción lanzada en Guards, Interceptors, Pipes o el Controller mismo, y deciden qué responder — sin esto, una excepción sin manejar termina en un 500 genérico sin control sobre el formato.

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

## Cómo no confundirlos

| Pieza | Responde a | Cuándo corre |
|---|---|---|
| **Guard** | ¿Puede esta request continuar? | Antes de todo lo demás |
| **Interceptor** | ¿Qué hago antes Y después del handler? | Envuelve al handler completo |
| **Pipe** | ¿Los datos de entrada tienen la forma correcta? | Justo antes del handler, sobre los parámetros |
| **Exception Filter** | ¿Qué respondo si algo de lo anterior falla? | Solo si se lanzó una excepción |

---
Relacionado: [Arquitectura de NestJS](arquitectura.md), [Autenticación y Seguridad](../../backend/autenticacion.md), [Endpoints para microservicios (FastAPI)](../fastapi/endpoints-microservicios.md#3-validación-de-entrada-con-pydantic) (Pydantic cumple el mismo rol que un DTO + Pipe, en Python).
