# Regla: documentación y variables de entorno coherentes

## Nombre de la regla

**Documentación y variables de entorno coherentes**

## Alcance

Aplica a:

- `README.md` y `README.es.md`.
- Archivos `.env*`, `frontend/`, configuración de Vite y cualquier código que lea `import.meta.env`.
- Tareas que agreguen, renombren o documenten variables de entorno, proxies o URLs de servicios.

## Justificación

Ambos README indican copiar `frontend/.env.example`, pero ese archivo no existe en el repositorio. El frontend sí consume `VITE_API_BASE_URL` en [`frontend/src/App.tsx`](../../frontend/src/App.tsx#L13) y usa la ruta `/api/metrics` cuando la variable está vacía. Esta discrepancia deja una instrucción documentada que no se puede seguir literalmente para configurar un backend alternativo.

## Guía específica para coding agents

1. Antes de documentar una variable o ruta, verifica que el archivo exista y que el nombre coincida con el código que la consume.
2. Si se documenta una variable de entorno, crea o conserva un archivo de ejemplo versionado en la ruta indicada, sin incluir secretos.
3. Si no corresponde crear un archivo de ejemplo, documenta cómo crear la configuración manualmente, el valor esperado y el comportamiento por defecto.
4. Mantén sincronizadas las instrucciones de `README.md` y `README.es.md` cuando describan el mismo flujo.
5. Después de cambiar la configuración, comprueba las rutas con el árbol real del repositorio y ejecuta el build o la validación específica disponible.

## Criterio de cumplimiento

No debe quedar ninguna instrucción que apunte a un archivo inexistente o a una variable cuyo nombre no coincida con la implementación.
