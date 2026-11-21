# Stack tecnológico

## Frontend

- Lenguaje principal: TypeScript.
- Framework de UI: React 19.
- Bundler y servidor de desarrollo: Vite.
- Estilos: Tailwind CSS mediante `@tailwindcss/vite`.
- Gráficos: Recharts.
- Iconos: `lucide-react`.
- Pruebas: Vitest y `@vitest/coverage-v8`.
- Lint y compilación: ESLint y TypeScript.
- Dependencias y scripts declarados en [`frontend/package.json`](../frontend/package.json).

Scripts disponibles:

```text
npm run dev
npm run build
npm run lint
npm test
npm run test:coverage
```

La configuración de Vite en [`frontend/vite.config.ts`](../frontend/vite.config.ts) habilita el host `0.0.0.0`, el proxy `/api` hacia `http://backend:8000` y el alias `@`.

## Backend

- Lenguaje principal: Python.
- Framework HTTP: FastAPI.
- Validación y modelos: Pydantic, usado por FastAPI en [`backend/app/routes.py`](../backend/app/routes.py).
- Servidor ASGI: Uvicorn.
- Depuración: debugpy.
- Pruebas: pytest, pytest-cov y httpx.
- Dependencias declaradas en [`backend/requirements.txt`](../backend/requirements.txt).

El objeto ASGI es `app` en [`backend/app/main.py`](../backend/app/main.py), y el arranque documentado del contenedor usa `app.main:app`.

## Contenedores y ejecución

- [`frontend/Dockerfile`](../frontend/Dockerfile) usa `node:24-alpine`, instala el lockfile/package manifest con `npm install` y ejecuta Vite en el puerto `5173`.
- [`backend/Dockerfile`](../backend/Dockerfile) usa `python:3.13-slim`, instala `requirements.txt` y ejecuta Uvicorn mediante debugpy en el puerto `8000`, con el puerto `5678` para depuración.
- [`docker-compose.yml`](../docker-compose.yml) orquesta ambos servicios, publica `5173`, `8000` y `5678`, y monta los directorios locales en los contenedores.
- El comando recomendado por la documentación es `docker compose up --build`; para detener servicios se usa `docker compose down`, según [`README.md`](../README.md) y [`README.es.md`](../README.es.md).
