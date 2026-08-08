# Proveedores de Identidad Gestionados (Cognito, Auth0, Firebase Auth)

En vez de construir el flujo de auth propio (registro, login, hashing, JWT — ver [Autenticación y Seguridad](autenticacion.md)), delegarlo a un servicio externo especializado que ya lo resolvió, testeó y hardeneó.

## Qué resuelven

- Registro/login (incluyendo social login — Google, GitHub, etc. — sin implementar OAuth contra cada proveedor a mano).
- MFA (multi-factor authentication).
- Recuperación de contraseña, verificación de email.
- Emisión y validación de tokens (JWT, sesiones).
- Compliance/certificaciones que un equipo chico difícilmente puede armar por su cuenta (SOC 2, etc.).

```python
# conceptual — el backend ya no hashea passwords ni emite JWT propios,
# solo valida el token que emitió el proveedor externo
def get_current_user(token: str):
    claims = cognito_client.verify_token(token)  # la verificación la hace el SDK del proveedor
    return claims["sub"]
```

## El trade-off

**A favor**: menos código propio para mantener y menos superficie de ataque — cada línea de auth que no escribís es una línea que no podés tener un bug de seguridad en. Los proveedores grandes invierten en seguridad más de lo que la mayoría de los equipos puede.

**En contra**: *vendor lock-in* — migrar de un proveedor a otro más adelante implica migrar usuarios, tokens, y a veces re-verificar identidades. Menos control fino sobre el flujo exacto (personalizar el UX de login puede ser limitado). Costo que crece con la cantidad de usuarios activos.

## Ejemplos

- **AWS Cognito**: integra nativo con el resto de servicios AWS (API Gateway, Lambda).
- **Auth0** (ahora parte de Okta): más flexible/agnóstico de cloud, fuerte en customización de flujos.
- **Firebase Auth**: la opción más simple para apps que ya usan el resto del ecosistema Firebase.

## Los conceptos siguen aplicando

Usar un proveedor gestionado no vuelve irrelevante lo de [Autenticación y Seguridad](autenticacion.md) — el backend igual necesita entender qué es un JWT, cómo se valida una firma, y la diferencia entre [Autenticación vs Autorización](autenticacion.md#7-autenticación-vs-autorización) (el proveedor resuelve la autenticación — quién sos —, pero los permisos específicos de tu dominio — qué podés hacer — siguen siendo responsabilidad de tu backend).

---
Relacionado: [Autenticación y Seguridad](autenticacion.md), [API Gateway](api-gateway.md) (la autenticación suele validarse ahí, centralizada).
