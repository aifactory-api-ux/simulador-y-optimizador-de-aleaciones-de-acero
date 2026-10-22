Hola, soy Developer. Implementaré las funcionalidades del proyecto según el plan detallado a continuación.

# DEVELOPMENT PLAN: Simulador y Optimizador de Aleaciones de Acero

## 1. ARCHITECTURE OVERVIEW

El proyecto consiste en una aplicación web de pila completa para el cálculo del Equivalente de Carbono (CEV) en tiempo real y la optimización de costes de aleaciones de acero. La arquitectura se divide en dos servicios principales: un backend y un frontend, ambos contenerizados con Docker y orquestados con Docker Compose.

**Backend:**
Desarrollado con Python 3.11 y FastAPI, utiliza Pydantic para la validación de datos, SQLAlchemy como ORM y SQLite como base de datos persistente. Pandas se emplea para el procesamiento de datos de precios. El backend expone una API RESTful para:
*   Cálculo de CEV y clasificación de soldabilidad.
*   Optimización de mezclas de materiales para minimizar costes bajo restricciones de CEV y composición.
*   Gestión de materias primas (CRUD, historial de precios).
*   Ingesta de precios desde archivos CSV.
*   Registro y consulta de aleaciones históricas.
*   Analíticas agregadas y tendencias.
El backend incluye un endpoint `/health` para monitoreo y un mecanismo de inicialización de base de datos con datos semilla al inicio.

**Frontend:**
Desarrollado con React 18 y TypeScript, utiliza Vite como herramienta de construcción, Axios para las llamadas a la API y React Router DOM para la navegación. La interfaz de usuario sigue estrictamente el contrato de diseño UI/UX, implementando componentes base reutilizables y páginas específicas para:
*   Interfaz de entrada de composición (cálculo local de CEV en tiempo real).
*   Sugerencias de optimización de costes.
*   Vistas analíticas.
El frontend realiza el cálculo de CEV y la clasificación de soldabilidad de forma local y síncrona para proporcionar retroalimentación instantánea, independientemente de la disponibilidad del backend o de los precios.

**Base de Datos:**
SQLite se utiliza para la persistencia de datos de materias primas, historial de precios, aleaciones y sus composiciones. La base de datos se inicializa automáticamente al inicio del backend y se gestionan backups diarios.

**Contenerización y Orquestación:**
Ambos servicios se empaquetan en imágenes Docker y se orquestan mediante Docker Compose. Esto facilita el despliegue local y en entornos de producción, asegurando la portabilidad y la escalabilidad. Se incluyen scripts para la gestión del entorno y la automatización de tareas.

**Estructura de Carpetas:**
La estructura de carpetas sigue el estándar definido, con directorios `backend/`, `frontend/`, `docs/` y archivos de configuración en la raíz del proyecto.

```
project-root/
├── .env.example
├── .gitignore
├── README.md
├── docker-compose.yml
├── .github/
│   └── workflows/
│       └── ci-cd.yml
├── backend/
│   ├── Dockerfile
│   ├── requirements.txt
│   ├── app/
│   │   ├── __init__.py
│   │   ├── main.py
│   │   ├── config.py
│   │   ├── database.py
│   │   ├── models.py
│   │   ├── schemas.py
│   │   ├── cev.py
│   │   ├── optimizer.py
│   │   ├── pricing.py
│   │   ├── persistence.py
│   │   ├── analytics.py
│   │   └── routers/
│   │       ├── __init__.py
│   │       ├── calculos.py
│   │       ├── optimizacion.py
│   │       ├── materiales.py
│   │       ├── aleaciones.py
│   │       └── analiticas.py
│   ├── data/
│   │   └── precios_mercado.csv
│   └── scripts/
│       └── backup_sqlite.py
├── frontend/
│   ├── Dockerfile
│   ├── package.json
│   ├── package-lock.json
│   ├── tsconfig.json
│   ├── tsconfig.node.json
│   ├── vite.config.ts
│   ├── index.html
│   └── src/
│       ├── main.tsx
│       ├── App.tsx
│       ├── styles/
│       │   ├── tokens.ts
│       │   └── global.css
│       ├── api/
│       │   ├── client.ts
│       │   ├── calculos.ts
│       │   ├── materiales.ts
│       │   ├── aleaciones.ts
│       │   └── analiticas.ts
│       ├── types/
│       │   └── api.ts
│       ├── hooks/
│       │   ├── useCEVCalculator.ts
│       │   ├── useOptimization.ts
│       │   └── useAnalytics.ts
│       ├── components/
│       │   └── ui/
│       │       ├── NavegacionPrincipal.tsx
│       │       ├── BotonCTA.tsx
│       │       ├── TarjetaContenido.tsx
│       │       ├── CampoEntradaNumerico.tsx
│       │       ├── IndicadorEstado.tsx
│       │       ├── TablaDatos.tsx
│       │       └── Grafico.tsx
│       └── pages/
│           ├── DesignSystemOverview.tsx
│           ├── ComponentsLibrary.tsx
│           ├── InterfazEntradaComposicion.tsx
│           ├── SugerenciasOptimizacionCostes.tsx
│           └── VistasAnaliticas.tsx
└── docs/
    └── ACTUALIZACION_PRECIOS.md
```

## 2. ACCEPTANCE CRITERIA
1.  El CEV se calcula correctamente según la fórmula y se recalcula automáticamente al modificar cualquier elemento, en menos de 1 segundo, sin recargar la página.
2.  El sistema debe advertir o bloquear la entrada si la suma de porcentajes de la aleación supera el 100%.
3.  La pantalla muestra el CEV vigente y un indicador visual de aptitud (verde / ámbar / rojo) legible a distancia de operación. La pantalla es usable en dispositivo táctil, con controles de tamaño adecuado.
4.  El sistema clasifica la soldabilidad en al menos tres niveles (apta, con precauciones, no apta) según umbrales de CEV configurables. Cuando el CEV supera el umbral de riesgo, se muestra una alerta explícita.
5.  El sistema devuelve una mezcla propuesta con las cantidades por material de mínimo coste que cumple todas las restricciones de composición y el CEV objetivo dentro de la tolerancia definida.
6.  El ingeniero puede crear, editar y desactivar materias primas con su precio y fecha de vigencia. Cada cálculo de optimización usa el precio vigente en la fecha del cálculo.
7.  Un usuario sin sesión válida no puede acceder a ninguna pantalla funcional del simulador. El rol operario solo accede a la vista táctil de cálculo y consulta, y recibe un error de autorización en el resto de endpoints.
8.  El formulario admite al menos los elementos C, Mn, Si, P, S, Cr, Ni, Mo, Cu y V, con su unidad en % en peso. El CEV se recalcula automáticamente al modificar cualquier elemento, en menos de 1 segundo, sin recargar la página.
9.  La comparativa muestra coste de la receta actual, coste de la mezcla optimizada, ahorro en € y ahorro en %. La comparativa indica si la mezcla optimizada empeora la receta actual en coste.
10. Cada cálculo y optimización se persiste con fecha, usuario, composición de entrada, parámetros aplicados y resultado.
11. El histórico permite filtrar por rango de fechas, usuario y rango de CEV.
12. El reporte se puede descargar en al menos un formato estructurado (PDF o CSV) desde la vista de resultado y desde el histórico. El reporte incluye fecha, usuario, composición, CEV, clasificación de soldabilidad y mezcla optimizada.
13. El sistema continúa calculando el CEV de forma independiente si la fuente de precios no está disponible. La funcionalidad de optimización de costes debería indicar la indisponibilidad de precios o no estar disponible.
14. El sistema debe procesar precios de mercado de chatarra y ferroaleaciones desde CSV o base de datos. Cada cálculo de optimización usa el precio vigente en la fecha del cálculo.
15. El CEV calculado por la aplicación coincide exactamente con el cálculo manual usando la fórmula CEV = C + Mn/6 + (Cr+Mo+V)/5 + (Ni+Cu)/15, mostrando 3 decimales.
16. El sistema debe rechazar valores fuera de rango (ej. negativos o no numéricos) y/o mostrar un mensaje de error apropiado. Para valores excesivamente altos, el sistema debe validar la entrada o advertir.
17. El sistema procesa precios de mercado de chatarra y ferroaleaciones desde CSV o base de datos. La optimización de costes utiliza el precio actualizado desde la base de datos SQL interna.
18. La API REST debe estar protegida. Un usuario sin sesión válida no puede acceder a ninguna pantalla funcional del simulador. El rol operario recibe un error de autorización en el resto de endpoints.
19. El sistema debe indicar claramente que no se pudo encontrar una mezcla optimizada que cumpla todas las restricciones, o proponer la mejor opción posible con una advertencia.
20. El sistema clasifica la soldabilidad correctamente en los niveles (apta, con precauciones, no apta) según los umbrales de CEV configurables, incluso en los límites exactos. Se muestra una alerta explícita.
21. El reporte incluye fecha, usuario, composición, CEV, clasificación de soldabilidad. La sección de mezcla optimizada con coste se maneja adecuadamente cuando no aplica.
22. El valor del CEV se recalcula automáticamente en la interfaz en menos de 1 segundo tras cada modificación, sin recargar la página.
23. El sistema mantiene una disponibilidad de al menos 99% y los tiempos de respuesta para los cálculos de CEV y optimización se mantienen por debajo de 3 segundos para el 95% de las solicitudes.
24. El Operario accede exitosamente a la vista táctil de cálculo y consulta. Para las demás pantallas (Optimación, Gestión de Materias Primas, Histórico), el sistema muestra un error de autorización o redirige a una página.
25. El sistema debe manejar la corrupción del archivo CSV de precios de manera robusta, registrando un error y, idealmente, utilizando la última configuración válida o mostrando un mensaje claro al usuario.
26. El cálculo del CEV se realiza correctamente y se muestra el resultado. Al intentar la optimización de costes, el sistema informa que la funcionalidad no está disponible debido a la falta de datos de precios.
27. El cálculo de la mezcla optimizada se completa y los resultados se muestran en menos de 3 segundos.
28. El cálculo de optimización de costes utiliza exclusivamente el precio de la materia prima que está vigente en la fecha del cálculo.
29. La API debe rechazar la solicitud con un código de estado 401 (No autorizado) o 403 (Prohibido) para todos los endpoints protegidos.
30. La lista de materias primas se carga y es completamente navegable en menos de 3 segundos.
31. Cada cálculo y optimización se persiste con fecha, usuario, composición de entrada, parámetros aplicados y resultado, y es correctamente visible en el histórico.
32. La pantalla es usable en dispositivo táctil, con controles de tamaño adecuado y sin necesidad de scroll horizontal. Muestra el CEV vigente y un indicador visual de aptitud (verde / ámbar / rojo) legible a distancia.
33. Los contenedores Docker se inician sin errores, la aplicación es accesible y todas las funcionalidades (cálculo de CEV, optimización) operan correctamente.
34. La base de datos de precios se restaura exitosamente desde el backup y los datos de materias primas son consistentes con el estado del backup.
35. El tiempo de respuesta promedio de las APIs para el cálculo de CEV y optimización es inferior a 100 ms, y el 95% de las solicitudes se completan en menos de 300 ms.
36. El sistema debe sanitizar las entradas, prevenir la ejecución de sentencias SQL maliciosas, no mostrar errores internos de la base de datos ni permitir la manipulación de la estructura de la base de datos.
37. El reporte se genera y está disponible para descarga en menos de 5 segundos.
38. El sistema sanitiza las entradas de usuario, impidiendo la ejecución de scripts maliciosos y mostrando el contenido de forma segura.
39. La API debe devolver un código de estado HTTP 400 (Bad Request) y un mensaje de error claro y descriptivo que indique la causa del fallo en cada caso.
40. Las operaciones de actualización y creación de materias primas se completan en menos de 1 segundo, y la integridad de los datos en la base de datos SQLite se mantiene sin conflictos ni corrupción.

## TEAM SCOPE (MANDATORY — PARSED BY THE PIPELINE)
**Role:** role-tl (technical_lead)
**Role:** role-be (backend_developer)
**Role:** role-fe (frontend_developer)
**Role:** role-devops (devops_support)

## 3. EXECUTABLE ITEMS

### ITEM 1: Foundation — shared types, interfaces, DB schemas, config
**Goal:** Create all foundational backend modules including configuration, database setup, SQLAlchemy models, Pydantic schemas, and the core CEV calculation logic. Also, define shared TypeScript API types and design tokens for the frontend. This item establishes the core data contracts and shared utilities.
**Files to create:**
- backend/app/__init__.py (create) - Inicialización del paquete.
- backend/app/config.py (create) - Configuración de la aplicación y variables de entorno.
- backend/app/database.py (create) - Configuración de la base de datos SQLite y sesión.
- backend/app/models.py (create) - Definición de modelos SQLAlchemy para `MATERIA_PRIMA`, `ALEACION`, `COMPOSICION_ALEACION`, `HISTORIAL_PRECIOS`.
- backend/app/schemas.py (create) - Definición de modelos Pydantic para request/response de la API.
- backend/app/cev.py (create) - Lógica para `calcular_cev` y `clasificar_soldabilidad`.
- frontend/src/types/api.ts (create) - Definición de interfaces TypeScript para los contratos de la API.
- frontend/src/styles/tokens.ts (create) - Definición de tokens de diseño (colores, tipografía, layout) según el contrato UI/UX.
**Dependencies:** None
**Validation:**
1.  Verificar que `backend/app/config.py` carga las variables de entorno correctamente.
2.  Ejecutar un script Python para inicializar la base de datos y verificar que las tablas `MATERIA_PRIMA`, `ALEACION`, `COMPOSICION_ALEACION`, `HISTORIAL_PRECIOS` se crean correctamente.
3.  Verificar que `backend/app/cev.py` calcula el CEV y clasifica la soldabilidad con valores de ejemplo.
4.  Compilar el frontend (`npm install` y `npm run build` en `frontend/`) para asegurar que `frontend/src/types/api.ts` y `frontend/src/styles/tokens.ts` no tienen errores de sintaxis.
**Role:** role-tl (technical_lead)

### ITEM 2: Backend - Health, Raw Materials API & Price Ingestion
**Goal:** Implement the `/health` endpoint, the CRUD operations for raw materials, the price history endpoint, and the CSV price ingestion functionality. This includes the initial CSV load on application startup.
**Files to create:**
- backend/Dockerfile (create) - Dockerfile para el servicio backend, multi-stage build, expone puerto 21000.
- backend/requirements.txt (create) - Dependencias de Python para el backend.
- backend/app/main.py (create) - Punto de entrada de la aplicación FastAPI, incluye el endpoint `/health` y el `lifespan` para inicialización de DB y carga CSV.
- backend/app/pricing.py (create) - Lógica para `cargar_precios_desde_csv`, `obtener_precios_actuales`, `actualizar_precio`, `importar_csv`.
- backend/app/persistence.py (create) - Funciones de persistencia para materias primas y su historial de precios.
- backend/app/routers/__init__.py (create) - Inicialización del paquete de routers.
- backend/app/routers/materiales.py (create) - Router para endpoints de materias primas (`GET /api/v1/materiales-primas`, `POST /api/v1/materiales-primas`, `PUT /api/v1/materiales-primas/{id}`, `DELETE /api/v1/materiales-primas/{id}`, `GET /api/v1/materiales-primas/{id}/historial-precios`) y precios (`POST /api/v1/precios/importar-csv`).
- backend/data/precios_mercado.csv (create) - Archivo CSV de ejemplo para precios de mercado.
**Dependencies:** Item 1
**Validation:**
1.  Iniciar el backend y verificar que el endpoint `GET /health` devuelve `{"estado": "ok", "servicio": "simulador-cev", "base_datos": "ok", "fuente_precios": "sqlite"}`.
2.  Verificar que la carga inicial de `precios_mercado.csv` se realiza al inicio del backend y que las materias primas se persisten en la DB.
3.  Probar `POST /api/v1/materiales-primas` para crear una nueva materia prima.
4.  Probar `GET /api/v1/materiales-primas` para listar materias primas.
5.  Probar `PUT /api/v1/materiales-primas/{id}` para actualizar un precio y verificar que se crea un registro en el historial.
6.  Probar `GET /api/v1/materiales-primas/{id}/historial-precios` para obtener el historial de precios.
7.  Probar `POST /api/v1/precios/importar-csv` con un CSV válido y uno malformado, verificando las respuestas `ImportarCSVResponse` y el manejo de errores.
**Role:** role-be (backend_developer)

### ITEM 3: Backend - CEV Calculation & Alloy Records API
**Goal:** Implement the API endpoint for CEV calculation and the CRUD operations for alloy records. This includes persisting the calculated CEV and soldability, and handling alloy compositions.
**Files to create:**
- backend/app/routers/calculos.py (create) - Router para el endpoint de cálculo de CEV (`POST /api/v1/calculos/cev`).
- backend/app/routers/aleaciones.py (create) - Router para endpoints de aleaciones (`GET /api/v1/aleaciones`, `POST /api/v1/aleaciones`, `GET /api/v1/aleaciones/{id}`).
**Dependencies:** Item 1, Item 2
**Validation:**
1.  Probar `POST /api/v1/calculos/cev` con diferentes composiciones elementales y verificar que el CEV, la soldabilidad, `suma_porcentajes`, `supera_cien`, `es_valida` y `es_segura` se calculan correctamente.
2.  Probar `POST /api/v1/aleaciones` para guardar una nueva aleación con su composición y verificar que se persiste en la base de datos.
3.  Probar `GET /api/v1/aleaciones` para listar aleaciones.
4.  Probar `GET /api/v1/aleaciones/{id}` para obtener una aleación específica.
5.  Verificar el manejo de errores para entradas inválidas (ej. suma de porcentajes > 100%, materia prima inexistente).
**Role:** role-be (backend_developer)

### ITEM 4: Backend - Cost Optimization API
**Goal:** Implement the API endpoint for cost optimization, which calculates the cheapest feasible mixture of materials given elemental targets and CEV constraints.
**Files to create:**
- backend/app/optimizer.py (create) - Lógica para `optimizar_mezcla`, incluyendo la carga de precios vigentes y la aplicación de restricciones.
- backend/app/routers/optimizacion.py (create) - Router para el endpoint de optimización (`POST /api/v1/optimize`).
**Dependencies:** Item 1, Item 2, Item 3
**Validation:**
1.  Probar `POST /api/v1/optimize` con una `OptimizacionRequest` válida y verificar que devuelve una `OptimizacionResponse` con una mezcla optimizada, ahorro estimado y cumplimiento de restricciones.
2.  Probar el escenario donde no se encuentra una mezcla optimizada y verificar que se devuelven advertencias apropiadas.
3.  Simular la indisponibilidad de precios (ej. eliminando todas las materias primas) y verificar que el endpoint devuelve HTTP `503` con el código `FUENTE_PRECIOS_NO_DISPONIBLE`.
4.  Verificar que el optimizador rechaza solicitudes con porcentajes de materiales que exceden el 100%.
**Role:** role-be (backend_developer)

### ITEM 5: Backend - Analytics API
**Goal:** Implement the API endpoints for analytical summaries and historical data, allowing filtering by date range and alloy name.
**Files to create:**
- backend/app/analytics.py (create) - Lógica para `obtener_analiticas`, incluyendo resumen, tendencias y datos históricos.
- backend/app/routers/analiticas.py (create) - Router para los endpoints de analíticas (`GET /api/v1/analiticas/resumen`, `GET /api/v1/analiticas/historico`).
**Dependencies:** Item 1, Item 3
**Validation:**
1.  Probar `GET /api/v1/analiticas/resumen` sin filtros y con filtros de fecha y `colada`, verificando que devuelve `AnaliticaResponse` con datos agregados correctos.
2.  Probar `GET /api/v1/analiticas/historico` con y sin filtros, verificando que devuelve la lista de `HistoricoAnaliticoItem` correctamente.
3.  Asegurar que los cálculos de `cev_medio`, `coste_medio`, `porcentaje_coladas_seguras` y `ahorro_medio_porcentaje` son precisos.
**Role:** role-be (backend_developer)

### ITEM 6: Frontend - Design System & Base Components
**Goal:** Implement the core frontend application structure, including routing, global styles, API client, and all mandatory base UI components. Also, implement the `DesignSystemOverview` and `ComponentsLibrary` pages as per the UI/UX contract.
**Files to create:**
- frontend/Dockerfile (create) - Dockerfile para el servicio frontend, multi-stage build, expone puerto 21300.
- frontend/package.json (create) - Dependencias de Node.js y scripts para el frontend.
- frontend/package-lock.json (create) - Bloqueo de dependencias.
- frontend/tsconfig.json (create) - Configuración de TypeScript para el frontend.
- frontend/tsconfig.node.json (create) - Configuración de TypeScript para Node.js (Vite).
- frontend/vite.config.ts (create) - Configuración de Vite, incluyendo el uso de `VITE_API_BASE_URL`.
- frontend/index.html (create) - Archivo HTML de entrada de la aplicación.
- frontend/src/main.tsx (create) - Punto de entrada de la aplicación React.
- frontend/src/App.tsx (create) - Componente principal de la aplicación, configura el enrutamiento.
- frontend/src/styles/global.css (create) - Estilos globales y variables CSS.
- frontend/src/api/client.ts (create) - Cliente Axios para las llamadas a la API.
- frontend/src/components/ui/NavegacionPrincipal.tsx (create) - Componente `NavegacionPrincipal`.
- frontend/src/components/ui/BotonCTA.tsx (create) - Componente `BotonCTA`.
- frontend/src/components/ui/TarjetaContenido.tsx (create) - Componente `TarjetaContenido`.
- frontend/src/components/ui/CampoEntradaNumerico.tsx (create) - Componente `CampoEntradaNumerico`.
- frontend/src/components/ui/IndicadorEstado.tsx (create) - Componente `IndicadorEstado`.
- frontend/src/components/ui/TablaDatos.tsx (create) - Componente `TablaDatos`.
- frontend/src/components/ui/Grafico.tsx (create) - Componente `Grafico`.
- frontend/src/pages/DesignSystemOverview.tsx (create) - Implementación de la página `Design System Overview`.
- frontend/src/pages/ComponentsLibrary.tsx (create) - Implementación de la página `Components Library`.
**Dependencies:** Item 1
**Validation:**
1.  Construir la imagen Docker del frontend y verificar que se inicia correctamente en el puerto 21300.
2.  Acceder a `http://localhost:21300` y verificar que la aplicación React se carga sin errores.
3.  Navegar a las rutas de `DesignSystemOverview` y `ComponentsLibrary` y verificar que se renderizan fielmente al contrato UI/UX.
4.  Verificar que todos los componentes base (`NavegacionPrincipal`, `BotonCTA`, etc.) se renderizan correctamente en `ComponentsLibrary`.
5.  Asegurar que `frontend/src/styles/tokens.ts` se utiliza para los colores y dimensiones.
**Role:** role-fe (frontend_developer)

### ITEM 7: Frontend - CEV Input Interface
**Goal:** Implement the `InterfazEntradaComposicion` page, including the elemental input panel, real-time CEV and soldability display, and integration with the `useCEVCalculator` hook.
**Files to create:**
- frontend/src/hooks/useCEVCalculator.ts (create) - Hook `useCEVCalculator` para la lógica de cálculo local de CEV.
- frontend/src/api/calculos.ts (create) - Módulo API para llamadas relacionadas con cálculos de CEV.
- frontend/src/pages/InterfazEntradaComposicion.tsx (create) - Implementación de la página `Interfaz de Entrada de Composición` según el contrato UI/UX.
**Dependencies:** Item 1, Item 3, Item 6
**Validation:**
1.  Navegar a la página `Interfaz de Entrada de Composición`.
2.  Introducir valores en los campos de elementos y verificar que el CEV y la soldabilidad se actualizan en tiempo real (<1s) sin recargar la página.
3.  Verificar que el indicador visual de aptitud (`IndicadorEstado`) cambia de color según la soldabilidad.
4.  Introducir una suma de porcentajes superior al 100% y verificar que se muestra una advertencia y la acción de optimización se deshabilita.
5.  Verificar que la página es usable en dispositivos táctiles, con controles de tamaño adecuado.
**Role:** role-fe (frontend_developer)

### ITEM 8: Frontend - Cost Optimization Suggestions
**Goal:** Implement the `SugerenciasOptimizacionCostes` page, displaying current and optimized mixture comparisons, cost savings, and integrating with the `useOptimization` hook.
**Files to create:**
- frontend/src/hooks/useOptimization.ts (create) - Hook `useOptimization` para la lógica de optimización y aplicación de mezclas.
- frontend/src/api/optimizacion.ts (create) - Módulo API para llamadas relacionadas con la optimización.
- frontend/src/api/aleaciones.ts (create) - Módulo API para llamadas relacionadas con aleaciones (para la función `aplicar` de `useOptimization`).
- frontend/src/pages/SugerenciasOptimizacionCostes.tsx (create) - Implementación de la página `Sugerencias de Optimización de Costes` según el contrato UI/UX.
**Dependencies:** Item 1, Item 4, Item 6, Item 7
**Validation:**
1.  Navegar a la página `Sugerencias de Optimización de Costes`.
2.  Simular una solicitud de optimización y verificar que la página muestra la comparación entre la mezcla actual y la optimizada, incluyendo CEV, soldabilidad, costes y ahorro.
3.  Verificar que el botón "Aplicar mezcla" llama a la función `aplicar` del hook `useOptimization` y que el estado `aplicado` se actualiza.
4.  Probar el manejo de errores y advertencias cuando la optimización falla o no encuentra una mezcla adecuada.
**Role:** role-fe (frontend_developer)

### ITEM 9: Frontend - Analytical Views
**Goal:** Implement the `VistasAnaliticas` page, including date and heat filters, aggregate KPI cards, CEV and cost trend charts, and a historical data table, integrating with the `useAnalytics` hook.
**Files to create:**
- frontend/src/hooks/useAnalytics.ts (create) - Hook `useAnalytics` para la lógica de filtrado y obtención de datos analíticos.
- frontend/src/api/analiticas.ts (create) - Módulo API para llamadas relacionadas con analíticas.
- frontend/src/pages/VistasAnaliticas.tsx (create) - Implementación de la página `Vistas Analíticas` según el contrato UI/UX.
**Dependencies:** Item 1, Item 5, Item 6
**Validation:**
1.  Navegar a la página `Vistas Analíticas`.
2.  Aplicar filtros de fecha y `colada` y verificar que los datos en las tarjetas KPI, gráficos y tabla histórica se actualizan correctamente.
3.  Verificar que los gráficos (`Grafico`) y la tabla (`TablaDatos`) muestran los datos de tendencias e histórico de forma legible.
4.  Asegurar que la interfaz de usuario es responsiva y usable.
**Role:** role-fe (frontend_developer)

### ITEM 10: Infrastructure & Deployment
**Goal:** Complete the Docker orchestration, including `docker-compose.yml`, `run.sh`, `.env.example`, `.gitignore`, `.dockerignore`, `README.md`, el script de backup de SQLite y la documentación de actualización de precios.
**Files to create:**
- docker-compose.yml (create) - Configuración de Docker Compose para ambos servicios (backend y frontend) con healthchecks y `depends_on: service_healthy`.
- .env.example (create) - Archivo de ejemplo con todas las variables de entorno requeridas y sus descripciones.
- .gitignore (create) - Archivo para ignorar archivos y directorios no deseados en el control de versiones.
- .dockerignore (create) - Archivo para ignorar archivos y directorios al construir las imágenes Docker.
- run.sh (create) - Script para construir, iniciar y verificar el estado de la aplicación Docker Compose.
- README.md (create) - Documentación del proyecto, incluyendo prerrequisitos, cómo clonar, ejecutar, probar y endpoints.
- backend/scripts/backup_sqlite.py (create) - Script para realizar backups diarios de la base de datos SQLite.
- docs/ACTUALIZACION_PRECIOS.md (create) - Documentación sobre cómo actualizar los precios de las materias primas.
- .github/workflows/ci-cd.yml (create) - Workflow de GitHub Actions para CI/CD.
**Dependencies:** All previous items
**Validation:**
1.  Ejecutar `./run.sh` y verificar que todos los servicios se construyen, inician y reportan un estado saludable.
2.  Acceder a `http://localhost:21300` y verificar que la aplicación web es completamente funcional.
3.  Verificar que el script `backend/scripts/backup_sqlite.py` se ejecuta correctamente y crea un archivo de backup de la base de datos.
4.  Asegurar que `.env.example` contiene todas las variables de entorno necesarias y que `README.md` proporciona instrucciones claras.
5.  Verificar que el workflow de CI/CD está configurado correctamente en `.github/workflows/ci-cd.yml`.
**Role:** role-devops (devops_support)