# 0002. Token JWT en localStorage con interceptor axios

- Estado: Aceptada
- Fecha: 2026-06-01

## Contexto

El backend emite un JWT stateless en el login. El cliente necesita guardarlo,
adjuntarlo a cada request protegida, y reaccionar cuando expira o es inválido.
Dónde se guarda el token tiene implicancias de seguridad reales.

## Decisión

Guardar el token en **localStorage** (vía el store de Zustand persistido) y
centralizar su uso en un interceptor de axios (`src/lib/api.ts`):

- El interceptor de request adjunta `Authorization: Bearer <token>` a cada
  llamada (`api.ts:11-14`).
- El interceptor de response detecta un `401`, limpia el token y la UI vuelve
  al login (`api.ts:21-24`). Así un token expirado o revocado no deja al
  usuario en un limbo.

## Alternativas consideradas

- **Cookie httpOnly.** Es la opción más resistente a XSS: JavaScript no puede
  leer la cookie, así que un script inyectado no roba el token. El costo es que
  requiere coordinación servidor-cliente (set-cookie, SameSite, CSRF) y un
  backend que la emita y la valide por cookie. Para el alcance de este cliente
  (SPA que habla con una API por Bearer), localStorage es el camino directo.
- **Token solo en memoria.** Inmune a persistencia maliciosa, pero se pierde al
  refrescar la página: el usuario tendría que re-loguearse en cada reload. Mala
  UX para una demo.

## Consecuencias

- **A favor:** simple y suficiente para una SPA con auth por Bearer. El token
  sobrevive a reloads; el manejo de expiración está centralizado en un solo
  lugar (el interceptor), no disperso por los componentes.
- **En contra:** localStorage es legible por JavaScript, así que es vulnerable a
  robo de token vía XSS. Es el tradeoff conocido de esta decisión. Se mitiga
  parcialmente del lado del contenido (sin inyección de HTML no sanitizado), y
  el JWT es de vida corta. Para un contexto con requisitos de seguridad más
  altos, la cookie httpOnly sería la elección.
