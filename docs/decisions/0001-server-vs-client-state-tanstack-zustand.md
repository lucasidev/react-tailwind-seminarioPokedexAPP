# 0001. Estado server vs client: TanStack Query + Zustand en vez de Redux

- Estado: Aceptada
- Fecha: 2026-06-01

## Contexto

El cliente original manejaba todo el estado con Redux: tanto los datos que
vienen del backend (el usuario, los pokemones) como el estado puramente de
cliente (el token de sesión). Tratar ambos con la misma herramienta lleva a
escribir a mano lo que un cache de servidor ya resuelve: loading states,
refetch, invalidación, deduplicación de requests.

## Decisión

Separar el estado por naturaleza (commit `feat: rewrite client in typescript
with tanstack query and zustand`):

- **TanStack Query** maneja todo el **server state**: el usuario (`useMe`) y
  los pokemones del proxy se cachean y se refetchean ante mutaciones (capturar,
  liberar, equipo) invalidando la query `['me']`.
- **Zustand** maneja solo el **client state**: el token JWT. Es un store mínimo,
  sin el boilerplate de Redux.

## Alternativas consideradas

- **Redux / Redux Toolkit para todo** (lo que había). Funciona, pero obliga a
  modelar el server state como si fuera client state: thunks para fetch,
  reducers para loading/error, lógica manual de invalidación. TanStack Query da
  todo eso de fábrica para datos remotos.
- **React Context para el token.** Alcanza para algo tan chico, pero re-renderiza
  todo el árbol consumidor en cada cambio y no persiste solo. Zustand con
  persistencia a localStorage es más directo.

## Consecuencias

- **A favor:** mucho menos código de estado. El cache, el refetch y la
  invalidación los maneja TanStack Query; el token vive en un store de pocas
  líneas. La separación server/client es explícita y fácil de razonar.
- **En contra:** son dos librerías de estado en vez de una. El equipo tiene que
  saber cuál usar para qué (regla simple: datos del backend a Query, lo demás a
  Zustand).
- Las mutaciones deben acordarse de invalidar la query correcta (`['me']`), o la
  UI muestra datos viejos. Es el contrato de TanStack Query.
