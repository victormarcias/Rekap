# NestJS — Arquitectura

Framework de Node construido sobre Express (o Fastify, intercambiable) — a diferencia de Express, que da control total pero ninguna estructura, NestJS impone una arquitectura opinionada inspirada en Angular (módulos, decorators, Dependency Injection).

## NestJS vs Express

- **TypeScript nativo**: Express es JS puro con tipos opcionales pegados encima; NestJS está pensado en TypeScript desde el diseño mismo del framework.
- **Arquitectura impuesta**: Express no te dice cómo organizar el código — cada proyecto termina con su propia convención. NestJS fuerza una estructura (Módulos → Controllers → Services) desde el primer archivo.
- **Dependency Injection incluido**: Express no tiene DI nativo — hay que armarlo a mano o con una librería aparte. En NestJS es parte del núcleo del framework.
- **Testing de fábrica**: NestJS trae su propio módulo de testing (`@nestjs/testing`) integrado, pensado para instanciar módulos/servicios aislados en tests unitarios.

Ninguna de las dos es "mejor" en abstracto: Express da más libertad y menos fricción para algo chico; NestJS paga el costo de una curva de entrada más alta a cambio de una arquitectura consistente que escala mejor en equipos grandes.

## Módulos, Controllers, Services

```typescript
// users.module.ts — agrupa todo lo relacionado a "users" en una unidad
@Module({
  controllers: [UsersController],
  providers: [UsersService],   // servicios disponibles para DI dentro de este módulo
})
export class UsersModule {}

// users.controller.ts — recibe la request HTTP, delega la lógica al service
@Controller('users')
export class UsersController {
  constructor(private readonly usersService: UsersService) {}   // DI: recibido, no instanciado a mano

  @Get(':id')
  findOne(@Param('id') id: string) {
    return this.usersService.findOne(id);   // el controller no sabe CÓMO se resuelve, solo delega
  }
}

// users.service.ts — la lógica de negocio real
@Injectable()
export class UsersService {
  findOne(id: string) {
    // acá va la query real, la regla de negocio, etc.
    return { id, name: 'Ana' };
  }
}
```

- **Módulo**: agrupa una sección del backend (users, orders, auth) — declara qué controllers y providers pertenecen ahí.
- **Controller**: recibe la request, extrae los datos (`@Param`, `@Body`, `@Query`), delega al service — no contiene lógica de negocio.
- **Service**: ejecuta la lógica de negocio real — el controller lo usa, nunca al revés.

## Dependency Injection

Una clase no instancia sus propias dependencias (`new UsersService()`) — las **recibe** desde afuera, resuelto automáticamente por el framework a partir del constructor.

```typescript
@Injectable()   // marca la clase como "inyectable" — el contenedor de NestJS puede resolverla
export class UsersService {
  constructor(private readonly db: DatabaseService) {}   // NestJS resuelve e inyecta DatabaseService solo
}
```

**Qué problema resuelve**: sin DI, cada clase que necesita `DatabaseService` tendría que crear su propia instancia (`new DatabaseService()`) o recibirla a mano en cada punto de la cadena de llamadas. Con DI, cualquier clase simplemente la **declara** en su constructor, y el framework se encarga de construirla y pasarla — más fácil de testear (en un test, se puede inyectar un mock/fake en vez de la implementación real) y de reemplazar sin tocar el código que la usa.

---
Relacionado: [Pipes, Guards, Interceptors y Exception Filters](pipes-guards-interceptors-filters.es.md), [Controller / Service / Repository](../../backend/controller-service-repository.es.md) (mismo patrón, agnóstico de framework), [Endpoints para microservicios (FastAPI)](../fastapi/microservice-endpoints.es.md#4-dependency-injection-con-depends) (mismo concepto de DI, en Python).
