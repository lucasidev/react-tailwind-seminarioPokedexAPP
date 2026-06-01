# Architecture Decision Records

Registro de las decisiones de arquitectura del frontend: por qué el manejo de
estado, la autenticación en el cliente y el empaquetado son como son. Cada ADR
captura una decisión con sus alternativas reales y el tradeoff asumido.

## Criterio: cuándo una decisión merece un ADR

Una decisión se documenta como ADR si cumple al menos dos de estas tres:

1. **Había alternativas reales.** Se eligió entre opciones con tradeoffs.
2. **Alguien querría romperla en 3 meses sin entender por qué.**
3. **El "por qué" se olvida fácil.** No es deducible del código solo.

## Índice

| ADR | Decisión |
|---|---|
| [0001](0001-server-vs-client-state-tanstack-zustand.md) | Estado server vs client: TanStack Query + Zustand en vez de Redux |
| [0002](0002-jwt-in-localstorage-with-axios-interceptor.md) | Token JWT en localStorage con interceptor axios |
| [0003](0003-runtime-nginx-backend-url-substitution.md) | Sustitución del backend URL en runtime con nginx |

## Formato

Cada ADR sigue la misma estructura: Contexto, Decisión, Alternativas
consideradas, Consecuencias. Estado y fecha en el encabezado.
