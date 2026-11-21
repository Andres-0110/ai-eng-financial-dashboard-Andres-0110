# Regla: separar desarrollo y producción

## Nombre de la regla

**Separación explícita de configuración de desarrollo y producción**

## Alcance

Aplica a:

- `docker-compose.yml`.
- `backend/Dockerfile` y comandos de Uvicorn/debugpy.
- `backend/app/main.py` y configuración de CORS.
- Cualquier Dockerfile, Compose override, manifiesto de despliegue o configuración de runtime.

## Justificación

La configuración actual monta el código local y publica debugpy en [`docker-compose.yml`](../../docker-compose.yml#L8), [`docker-compose.yml`](../../docker-compose.yml#L18) y [`docker-compose.yml`](../../docker-compose.yml#L21). El backend arranca debugpy y Uvicorn con `--reload` en [`backend/Dockerfile`](../../backend/Dockerfile#L10), mientras FastAPI permite todos los orígenes, métodos y cabeceras en [`backend/app/main.py`](../../backend/app/main.py#L7). Estos valores son de desarrollo y no deben reutilizarse como configuración de producción.

## Guía específica para coding agents

1. Identifica explícitamente qué archivos o perfiles son solo de desarrollo.
2. Para producción, no uses `--reload`, debugpy, puertos de depuración publicados ni montajes de código fuente del host.
3. Restringe `allow_origins`, métodos y cabeceras a los valores necesarios para los clientes autorizados; no uses comodines como configuración de producción.
4. Mantén la configuración de producción reproducible mediante imágenes construidas y dependencias declaradas.
5. Si un servicio depende de la disponibilidad de otro, define un `healthcheck` y una condición de dependencia apropiada, no solo el orden de `depends_on`.
6. No afirmes que existe una configuración de producción sin localizarla y validarla en el repositorio.

## Criterio de cumplimiento

Debe ser imposible confundir el perfil de desarrollo con el de producción, y el perfil de producción no debe exponer debug, recarga automática, montajes de fuente ni CORS abierto.
