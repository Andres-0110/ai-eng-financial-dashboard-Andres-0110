# Regla: verificar build y disponibilidad de contenedores

## Nombre de la regla

**Validación de build, arranque y salud de servicios**

## Alcance

Aplica a:

- `docker-compose.yml`.
- Dockerfiles y scripts de arranque.
- Endpoints de salud, especialmente `GET /health`.
- Revisiones, cambios o incidencias relacionadas con Docker, Compose, puertos, dependencias o disponibilidad del backend.

## Justificación

La revisión confirmó que `docker compose config` valida la configuración declarativa, pero no verificó el build ni el arranque real, según [`verification.md`](../../verification.md#L91). Además, Compose solo declara `depends_on` para el backend en [`docker-compose.yml`](../../docker-compose.yml#L11), y no un `healthcheck`; el orden de inicio no garantiza que la API ya acepte solicitudes.

## Guía específica para coding agents

1. Después de cambiar Dockerfiles o Compose, ejecuta la validación sintáctica de Compose.
2. Cuando el entorno lo permita, construye las imágenes y arranca los servicios reales; no trates la validación sintáctica como evidencia de arranque correcto.
3. Comprueba el endpoint `GET /health` y una ruta funcional mínima, como `GET /api/metrics`, desde el contexto donde las consumirá el frontend.
4. Si el frontend depende de que el backend esté listo, añade un `healthcheck` al backend y configura la dependencia basada en salud.
5. Registra por separado errores de build, errores de arranque, fallos de salud y problemas de conectividad.
6. Si no puedes ejecutar contenedores, marca la verificación como no realizada y describe exactamente qué comando quedó pendiente.

## Criterio de cumplimiento

Una revisión de infraestructura debe aportar evidencia de configuración válida y, cuando sea posible, de imágenes construidas, servicios arrancados y endpoint de salud operativo.
