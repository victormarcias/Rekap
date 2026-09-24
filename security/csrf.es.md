# CSRF (Cross-Site Request Forgery)

Engañar al navegador de una víctima **ya autenticada** para que mande una request no deseada a un sitio donde tiene sesión activa. El navegador adjunta las cookies de ese dominio automáticamente en cualquier request, venga de donde venga — así que el servidor, si no hace nada más, no puede distinguir a simple vista una request legítima de una forjada.

```html
<!-- Página maliciosa que la víctima visita mientras tiene sesión activa en banco.com -->
<form action="https://banco.com/transferir" method="POST" id="ataque">
  <input type="hidden" name="monto" value="10000">
  <input type="hidden" name="destino" value="cuenta-del-atacante">
</form>
<script>document.getElementById("ataque").submit()</script>
<!-- El navegador manda la cookie de sesión de banco.com automáticamente con este POST -->
```

## Por qué un GET no debería tener efectos secundarios

Si `GET /transferir?monto=10000&destino=...` cambiara estado, el ataque ni siquiera necesitaría un formulario — alcanzaría con un `<img src="https://banco.com/transferir?...">` en cualquier página. Es una de las razones de fondo detrás de los [Safe Methods de HTTP](../backend/http-methods.es.md#safe-methods--sin-efectos-secundarios).

## Cómo se previene

- **`SameSite` en la cookie de sesión**: `Strict`/`Lax` le dicen al navegador que no mande esa cookie en requests que vienen de otro sitio — corta el ataque en el origen, sin tocar el backend (ver [Cookies](../frontend-react/almacenamiento-cliente.md#cookies)).
- **CSRF token**: un valor único por sesión (o por formulario) que el servidor exige en cada request que modifica estado, y que un atacante externo no tiene forma de conocer ni replicar.

```python
# El servidor genera un token único al crear la sesión y lo exige
# en cualquier request que modifique estado
@app.post("/transferir")
def transferir(monto: float, csrf_token: str = Form(...)):
    if csrf_token != session["csrf_token"]:
        raise HTTPException(403, "CSRF token inválido")
    ...
```

`SameSite` y CSRF token no son excluyentes — `SameSite=Lax` (el default en los navegadores modernos) ya cubre la mayoría de los casos, pero un CSRF token sigue siendo la defensa explícita para APIs que necesitan aceptar cookies cross-site a propósito.

---
Relacionado: [XSS](xss.es.md), [Autenticación y Seguridad](../backend/authentication.es.md#8-dónde-guardar-el-token-en-el-cliente), [HTTP Methods](../backend/http-methods.es.md#safe-methods--sin-efectos-secundarios), [Almacenamiento en el cliente](../frontend-react/almacenamiento-cliente.md#cookies).
