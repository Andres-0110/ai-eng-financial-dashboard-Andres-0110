# Regla: estados honestos de verificación

## Nombre de la regla

**Distinguir pruebas pasadas, fallidas y bloqueadas por el entorno**

## Alcance

Aplica a:

- `frontend/package.json`, `frontend/package-lock.json` y pruebas de `frontend/`.
- `backend/requirements.txt` y pruebas de `backend/`.
- Informes de revisión, cambios que afecten comportamiento y cualquier ejecución de lint, build o tests.

## Justificación

El frontend declara `npm test`, `npm run lint` y `npm run build` en [`frontend/package.json`](../../frontend/package.json#L6), y `vitest` está declarado como dependencia de desarrollo en [`frontend/package.json`](../../frontend/package.json#L41). El backend declara FastAPI y pytest en [`backend/requirements.txt`](../../backend/requirements.txt), pero [`verification.md`](../../verification.md#L103) registra que pytest no pudo recolectar por falta de `fastapi` y que `npm test` no pudo iniciar por falta de `vitest`. Un bloqueo del entorno no demuestra ni éxito ni fallo funcional.

## Guía específica para coding agents

1. Ejecuta los comandos de validación definidos por los manifiestos del proyecto, priorizando el conjunto mínimo que cubra el cambio.
2. Si faltan dependencias, restáuralas únicamente desde `frontend/package-lock.json` y `backend/requirements.txt` o los manifiestos correspondientes; no inventes versiones.
3. Clasifica cada resultado como `pasado`, `fallido` o `bloqueado por entorno`.
4. No reportes una suite como exitosa si no llegó a ejecutarse, y no conviertas una dependencia ausente en un defecto del producto.
5. Cuando una prueba falle después de poder ejecutarse, conserva el error y corrige la causa antes de declarar el cambio terminado.
6. En cambios que alteren periodos o cálculos financieros, añade pruebas enfocadas en el comportamiento nuevo, incluyendo el caso límite relevante.

## Criterio de cumplimiento

El informe final debe indicar qué comandos se ejecutaron, sus resultados y cualquier bloqueo reproducible, sin mezclar estados de infraestructura, entorno y funcionalidad.
