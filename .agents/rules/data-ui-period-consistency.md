# Regla: periodo visible derivado de los datos

## Nombre de la regla

**Periodo visible derivado de los datos o del filtro activo**

## Alcance

Aplica a:

- `frontend/src/App.tsx`.
- `frontend/src/components/dashboard/`.
- `frontend/src/lib/` y tipos de movimientos, filtros o métricas.
- `backend/app/routes.py` y cualquier endpoint que determine límites temporales.
- Cambios que afecten encabezados, etiquetas, filtros, agrupaciones o periodos financieros.

## Justificación

La aplicación pasa el texto fijo `2024 - Full Year` al encabezado en [`frontend/src/App.tsx`](../../frontend/src/App.tsx#L49), y el componente usa además `2024 — Full Year` como valor por defecto en [`frontend/src/components/dashboard/dashboard-header.tsx`](../../frontend/src/components/dashboard/dashboard-header.tsx#L7). En cambio, el backend calcula el año de los movimientos desde la fecha actual en [`backend/app/routes.py`](../../backend/app/routes.py#L65), por lo que el texto puede dejar de describir los datos mostrados al cambiar el año.

## Guía específica para coding agents

1. No introduzcas años, rangos o etiquetas de periodo hardcodeados en componentes de presentación si los datos se calculan dinámicamente.
2. Deriva la etiqueta visible de los mismos límites de fecha, filtros o respuesta de API que determinan las métricas.
3. Si el periodo es un valor por defecto, documenta por qué es seguro y evita que contradiga el periodo efectivo.
4. Cuando cambies la generación o filtrado de fechas, revisa también encabezados, tarjetas, gráficos y textos accesibles.
5. Añade o actualiza una prueba que cubra el cambio de año o un periodo distinto del valor histórico fijo.

## Criterio de cumplimiento

Para cualquier conjunto de datos renderizado, el periodo visible debe coincidir con sus límites efectivos y las pruebas deben detectar una etiqueta obsoleta al cambiar el año.
