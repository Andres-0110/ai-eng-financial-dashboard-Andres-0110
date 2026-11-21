# Overview del producto

## Qué hace hoy

El producto es un dashboard financiero que muestra una vista ejecutiva de movimientos financieros. En la pantalla principal, [`frontend/src/App.tsx`](../frontend/src/App.tsx) solicita `GET /api/metrics`, calcula en el cliente ingresos, egresos, beneficio y porcentaje de beneficio mediante [`frontend/src/lib/financial-utils.ts`](../frontend/src/lib/financial-utils.ts), agrupa los movimientos por mes y los entrega a tarjetas KPI y gráficos de [`frontend/src/components/dashboard/`](../frontend/src/components/dashboard/).

Los movimientos tienen fecha, importe, tipo (`income`/`outcome`), categoría y segmento (`B2B`/`B2C`), según el modelo `FinancialMovement` de [`backend/app/routes.py`](../backend/app/routes.py). El backend genera 360 movimientos simulados por solicitud usando `seed=42`; no hay una base de datos ni persistencia observada en esta implementación.

## Conexión frontend-backend

1. El navegador carga el frontend en el servidor Vite.
2. [`frontend/src/App.tsx`](../frontend/src/App.tsx) usa `VITE_API_BASE_URL` si está definida; de lo contrario solicita la ruta relativa `/api/metrics`.
3. [`frontend/vite.config.ts`](../frontend/vite.config.ts) reenvía `/api` a `http://backend:8000` y define el alias `@` para `frontend/src`.
4. [`docker-compose.yml`](../docker-compose.yml) ejecuta el frontend en el puerto `5173` y el backend en el puerto `8000`.
5. El backend registra el router de [`backend/app/routes.py`](../backend/app/routes.py) desde [`backend/app/main.py`](../backend/app/main.py).

## Endpoints implementados

El router de [`backend/app/routes.py`](../backend/app/routes.py) implementa:

- `GET /health`: devuelve el estado básico `{"status": "ok"}`.
- `GET /api/metrics`: devuelve movimientos ordenados y permite filtros de fecha, categoría, tipo de operación y segmento.
- `GET /api/metrics/facets`: devuelve tipos, segmentos, categorías y rango de fechas disponibles.
- `GET /api/metrics/summary`: agrupa ingresos, egresos y neto por día, semana o mes.
- `GET /api/metrics/categories/top`: devuelve categorías principales por importe y tipo de operación.
- `GET /api/metrics/comparison`: compara el neto del periodo actual con el periodo anterior.
- `GET /api/metrics/alerts`: identifica periodos con egresos por encima de un umbral del promedio histórico.
- `GET /api/metrics/b2b` y `GET /api/metrics/b2c`: devuelven movimientos filtrados por segmento.

La interfaz actual solo consume `GET /api/metrics`, aunque el backend expone los demás endpoints. La documentación interactiva de FastAPI queda disponible en `/docs` cuando el backend está en marcha, como se describe en [`verification.md`](../verification.md).
