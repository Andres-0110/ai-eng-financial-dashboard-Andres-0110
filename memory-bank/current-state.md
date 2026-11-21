# Estado actual del proyecto

## Implementado

- El frontend carga movimientos desde `GET /api/metrics` y presenta KPIs y gráficos; la lógica está en [`frontend/src/App.tsx`](../frontend/src/App.tsx) y [`frontend/src/lib/financial-utils.ts`](../frontend/src/lib/financial-utils.ts).
- El backend define modelos Pydantic, genera datos simulados deterministas con `seed=42` y ofrece los endpoints descritos en [`backend/app/routes.py`](../backend/app/routes.py).
- El endpoint de salud, filtros de movimientos, endpoints B2B/B2C, facetas, resumen, categorías principales, comparación y alertas están implementados en el router.
- Las pruebas de backend en [`backend/tests/test_routes.py`](../backend/tests/test_routes.py) cubren salud, generación y orden de datos, filtros de fecha/categoría/tipo, B2B/B2C, facetas, resumen, categorías principales, comparación y alertas.
- Las pruebas de frontend en [`frontend/src/lib/financial-utils.test.ts`](../frontend/src/lib/financial-utils.test.ts) cubren KPIs, beneficio sin ingresos, agrupación cronológica entre años y formateo de moneda y porcentaje.
- `docker compose config` fue ejecutado correctamente durante la revisión documentada en [`verification.md`](../verification.md).

## Verificado por inspección o configuración

- La conexión del proxy Vite con `http://backend:8000` está declarada en [`frontend/vite.config.ts`](../frontend/vite.config.ts).
- Los servicios, puertos y volúmenes de desarrollo están declarados en [`docker-compose.yml`](../docker-compose.yml).
- FastAPI registra CORS abierto (`allow_origins=["*"]`) en [`backend/app/main.py`](../backend/app/main.py).
- La API expone documentación interactiva en `/docs` por el comportamiento estándar de FastAPI, y las URLs locales están documentadas en [`README.es.md`](../README.es.md).

## Gaps y fricciones conocidos

- La documentación indica copiar `frontend/.env.example` en [`README.md`](../README.md) y [`README.es.md`](../README.es.md), pero ese archivo no existe en el árbol revisado. El código sí lee `VITE_API_BASE_URL`.
- El encabezado del dashboard pasa el periodo fijo `2024 - Full Year` en [`frontend/src/App.tsx`](../frontend/src/App.tsx), mientras el backend calcula fechas usando el año actual y el anterior en [`backend/app/routes.py`](../backend/app/routes.py). El periodo visible puede no coincidir con los datos.
- No se observa una base de datos ni persistencia; los movimientos se generan en memoria por solicitud.
- La configuración observada es de desarrollo: monta código local, publica debugpy y usa `--reload` en [`docker-compose.yml`](../docker-compose.yml) y [`backend/Dockerfile`](../backend/Dockerfile). No se verificó una configuración de producción separada.
- `depends_on` solo ordena el inicio del backend y frontend; Compose no declara `healthcheck`, por lo que la disponibilidad de la API no queda garantizada por la configuración.
- La política CORS permite cualquier origen y no se ha evaluado para producción.
- Durante la revisión, `pytest -q` no pudo recolectar porque faltaba `fastapi` en el entorno Python activo y `npm test` no pudo iniciar porque `vitest` no estaba instalado. Esto es un bloqueo de entorno documentado, no evidencia de un fallo funcional.
- No se verificó la construcción ni el arranque real de los contenedores.

## Prioridades respaldadas por el estado actual

Estas prioridades se derivan directamente de los gaps documentados, no constituyen un roadmap adicional:

1. Resolver la discrepancia entre la documentación y la ausencia de `frontend/.env.example`, ya sea creando el ejemplo o corrigiendo las instrucciones.
2. Hacer que el periodo del encabezado se derive de los datos o del filtro efectivo y cubrir el cambio de año con una prueba.
3. Separar y validar una configuración de producción si el repositorio necesita soportarla: sin debugpy, sin `--reload`, sin montajes de código y con CORS restringido.
4. Añadir comprobación de salud en Compose y ejecutar un smoke test real de build, arranque, `/health` y `/api/metrics`.
5. Restaurar dependencias desde los manifiestos existentes y repetir las suites frontend y backend, registrando cada resultado como pasado, fallido o bloqueado por entorno.
