# 0003. Sustitución del backend URL en runtime con nginx

- Estado: Aceptada
- Fecha: 2026-06-01

## Contexto

El frontend es una SPA estática que necesita saber a qué backend pegarle. El
backend cambia por entorno (local, demo, producción). La pregunta es cuándo se
resuelve esa URL: en build time o en runtime.

Si se resuelve en build, cada entorno necesita su propia imagen Docker (una
build por URL de backend), lo que rompe el principio de "build once, deploy
anywhere".

## Decisión

Resolver el backend en **runtime**, no en build. El cliente hace todas sus
llamadas a `/api` (relativo); nginx proxea `/api` al backend real. La URL del
backend se inyecta al arrancar el contenedor, no al construir la imagen
(`Dockerfile`, commit `chore(docker): multi-stage build served by nginx`):

- Build multi-stage: `vite build` genera el `dist/`, luego `nginx:alpine` lo
  sirve (`Dockerfile:28-33`).
- `nginx:alpine` corre `envsubst` sobre los templates en
  `/etc/nginx/templates/*.template` al boot, sustituyendo `BACKEND_URL`
  (`Dockerfile:30-32`).
- `BACKEND_URL` tiene un default (`Dockerfile:35`) y se overridea con
  `-e BACKEND_URL=...` al `docker run`.

## Alternativas consideradas

- **URL absoluta inyectada en build** (`VITE_API_URL` horneada en el bundle).
  Es lo más simple, pero ata cada imagen a un backend: cambiar de entorno
  obliga a reconstruir. Una imagen por entorno es justo lo que se quiere evitar.
- **Config fetcheada en runtime** (un `config.json` que la SPA pide al arrancar).
  Funciona, pero suma un request extra y un archivo de config que mantener. El
  proxy de nginx con envsubst no toca el bundle ni agrega round-trips.

## Consecuencias

- **A favor:** una sola imagen sirve para todos los entornos; el backend se
  define al desplegar, no al construir. El cliente usa rutas relativas, así que
  no hay CORS entre el front y el back (mismo origen vía el proxy).
- **En contra:** depende del comportamiento de envsubst de `nginx:alpine` sobre
  los templates; quien mantenga el Dockerfile tiene que conocer ese mecanismo.
  El template de nginx (`nginx/default.conf.template`) es parte del contrato.
- En dev, Vite cumple el rol del proxy (`/api` proxeado al backend local), así
  que el modelo relativo es consistente entre dev y prod.
