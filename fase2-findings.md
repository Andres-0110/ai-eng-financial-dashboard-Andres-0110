# Fase 2: hallazgos y reglas propuestas

Este documento consolida los hallazgos de [`verification.md`](./verification.md), agrupados por categoría. Cada hallazgo conserva su estado de verificación y una evidencia concreta del repositorio. Las reglas son propuestas para una fase posterior; en esta fase no se crearon `.agents/rules` ni `memory-bank`.

## Resumen

| Categoría | Hallazgos | Prioridad dominante |
| --- | ---: | --- |
| Documentación y configuración | 1 | Media |
| Coherencia entre datos y UI | 1 | Alta |
| Despliegue y seguridad | 1 | Alta |
| Verificación y testing | 2 | Media |

## 1. Documentación y configuración

### F1 — Referencia a un archivo de entorno inexistente

- **Estado:** Confirmado.
- **Prioridad:** Media.
- **Descripción:** Ambos README indican copiar `frontend/.env.example`, pero el archivo no existe en el árbol revisado. El frontend sí lee `VITE_API_BASE_URL` y, cuando está vacío, utiliza la ruta relativa que depende del proxy de Vite.
- **Evidencia:**
  - [`README.md:45-46`](./README.md#L45) documenta el proxy y ordena copiar `frontend/.env.example`.
  - [`README.es.md:45-46`](./README.es.md#L45) repite la misma instrucción en español.
  - [`frontend/src/App.tsx:13-16`](./frontend/src/App.tsx#L13) usa `import.meta.env.VITE_API_BASE_URL ?? ""` y solicita `/api/metrics`.
  - La revisión del árbol del repositorio no encontró `frontend/.env.example`.
- **Impacto:** Una persona que necesite configurar un backend alternativo no puede seguir literalmente la documentación; la configuración documentada y la disponible quedan desalineadas.
- **Regla propuesta R1:** Toda variable de entorno mencionada en la documentación debe tener un archivo de ejemplo versionado en la ruta indicada, o la documentación debe explicar explícitamente cómo crearla y qué valores admite. Las rutas y nombres deben validarse contra el árbol real antes de cerrar una revisión.

## 2. Coherencia entre datos y UI

### F2 — Periodo mostrado fijo y potencialmente incorrecto

- **Estado:** Confirmado.
- **Prioridad:** Alta.
- **Descripción:** La interfaz muestra siempre `2024 - Full Year`, mientras el backend calcula el año de cada movimiento a partir de la fecha actual y del mes solicitado. El periodo visible puede no describir los datos que se están presentando.
- **Evidencia:**
  - [`frontend/src/App.tsx:49`](./frontend/src/App.tsx#L49) pasa el periodo fijo `2024 - Full Year`.
  - [`frontend/src/components/dashboard/dashboard-header.tsx:7`](./frontend/src/components/dashboard/dashboard-header.tsx#L7) mantiene además `2024 — Full Year` como valor por defecto.
  - [`backend/app/routes.py:65-75`](./backend/app/routes.py#L65) calcula el año usando `today.year` y `today.year - 1`, y asigna esa fecha a cada movimiento.
  - [`verification.md:99`](./verification.md#L99) registra la discrepancia observada.
- **Impacto:** El usuario puede interpretar métricas de un periodo distinto al que indica el encabezado, especialmente cuando cambia el año calendario.
- **Regla propuesta R2:** Los textos de periodo y filtros visibles deben derivarse de los mismos límites de fecha que generan o filtran los datos. No se deben dejar años o periodos financieros hardcodeados en componentes de presentación; deben existir pruebas que detecten el cambio de año.

## 3. Despliegue y seguridad

### F3 — Configuración de desarrollo expuesta como única configuración operativa

- **Estado:** Parcialmente confirmado; falta validar si existe una configuración de producción fuera de los archivos revisados.
- **Prioridad:** Alta.
- **Descripción:** La configuración observada monta el código local, publica el puerto de debugpy y arranca Uvicorn con recarga automática. Además, FastAPI permite cualquier origen mediante CORS. Estos valores son apropiados para desarrollo, pero no deben asumirse como configuración de producción.
- **Evidencia:**
  - [`docker-compose.yml:8-12`](./docker-compose.yml#L8) monta `./frontend:/app`, conserva `node_modules` en volumen y solo declara `depends_on`.
  - [`docker-compose.yml:18-22`](./docker-compose.yml#L18) publica HTTP en `8000` y debugpy en `5678`, y monta `./backend:/app`.
  - [`backend/Dockerfile:10-12`](./backend/Dockerfile#L10) expone `5678` y ejecuta debugpy con `--reload`.
  - [`backend/app/main.py:7-13`](./backend/app/main.py#L7) configura `allow_origins=["*"]`, todos los métodos y todas las cabeceras.
  - [`verification.md:101`](./verification.md#L101) deja sin verificar la existencia de una configuración de producción.
- **Impacto:** Si esta configuración se reutiliza fuera de desarrollo, aumenta la superficie expuesta (debug remoto, recarga y CORS sin restricción) y puede provocar un despliegue no reproducible.
- **Regla propuesta R3:** Toda configuración de desarrollo debe identificarse como tal y separarse de la de producción. Una configuración de producción debe desactivar `--reload` y debugpy, evitar montajes de código fuente, restringir CORS a orígenes permitidos y declarar comprobaciones de salud cuando el arranque dependa de otro servicio.

## 4. Verificación y testing

### F4 — La verificación de contenedores no cubre build ni arranque real

- **Estado:** No verificado en runtime.
- **Prioridad:** Media.
- **Descripción:** Se validó la sintaxis declarativa de Compose, pero no se verificó la construcción ni el arranque real de los contenedores. `depends_on` solo ordena el inicio y no espera a que la API esté lista; aunque existe `/health`, Compose no declara un `healthcheck`.
- **Evidencia:**
  - [`verification.md:42`](./verification.md#L42) documenta la ausencia de `healthcheck` y la semántica de `depends_on`.
  - [`docker-compose.yml:11-12`](./docker-compose.yml#L11) solo declara `depends_on: - backend`.
  - [`verification.md:91-93`](./verification.md#L91) distingue entre `docker compose config` exitoso y build/arranque no verificados.
- **Impacto:** Un Compose sintácticamente válido puede fallar al construir, arrancar o atender tráfico; el frontend puede iniciarse antes de que el backend esté listo.
- **Regla propuesta R4:** Una revisión de infraestructura debe ejecutar, cuando sea posible, el build y un smoke test de arranque, incluyendo `/health`. Si un servicio depende de disponibilidad y no solo de orden, debe declarar `healthcheck` y una condición de dependencia compatible.

### F5 — Suites declaradas pero no ejecutables en el entorno de revisión

- **Estado:** Confirmado como bloqueo del entorno, no como fallo funcional del código.
- **Prioridad:** Media.
- **Descripción:** Hay scripts y pruebas declaradas, pero la revisión no pudo ejecutarlas porque faltaban dependencias del entorno: `fastapi` para pytest y `vitest` para `npm test`. No se instalaron dependencias durante la revisión.
- **Evidencia:**
  - [`frontend/package.json:6-13`](./frontend/package.json#L6) declara `build`, `lint`, `test` y `test:coverage`.
  - [`frontend/package.json:31-42`](./frontend/package.json#L31) declara `@vitest/coverage-v8` y `vitest` como dependencias de desarrollo.
  - [`verification.md:103-107`](./verification.md#L103) registra las pruebas existentes y los errores de recolección/ejecución.
  - [`backend/requirements.txt:1-5`](./backend/requirements.txt#L1) declara FastAPI y pytest para el backend.
- **Impacto:** No hay evidencia de ejecución reciente de las pruebas; los resultados ausentes no deben interpretarse como pruebas exitosas ni como fallos del producto.
- **Regla propuesta R5:** Antes de concluir una verificación, instalar dependencias únicamente desde los manifiestos del proyecto si faltan en el entorno y ejecutar los comandos documentados. Registrar por separado “prueba pasada”, “prueba fallida” y “prueba bloqueada por entorno”, sin convertir un bloqueo en un resultado funcional.

## Reglas transversales propuestas

1. **Evidencia reproducible:** Cada hallazgo debe incluir ruta y línea, o explicar explícitamente cuando la evidencia sea la ausencia de un archivo o una ejecución no realizada.
2. **Separación de estados:** `Confirmado`, `Parcialmente confirmado` y `No verificado` no deben mezclarse; cada estado implica una acción de seguimiento diferente.
3. **Reglas fuera del código hasta su aprobación:** Estas propuestas documentan invariantes observadas y no crean todavía archivos de reglas ni memoria del proyecto.

## Acciones sugeridas para la siguiente fase

1. Decidir si se crea `frontend/.env.example` o si se corrige la documentación para describir la configuración manual.
2. Hacer que el periodo del encabezado provenga de los datos o del filtro activo y añadir una prueba de cambio de año.
3. Definir explícitamente una configuración de producción y restringir CORS/debug en ella.
4. Añadir `healthcheck` a Compose y ejecutar un smoke test real de los servicios.
5. Restaurar dependencias desde `frontend/package-lock.json` y `backend/requirements.txt`, y repetir las suites declaradas.
