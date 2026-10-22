# Delivery Report

**Outcome:** `partial`

## Completion metrics

- Implementation files: 1
- Expected implementation files: 60
- Blocking items: 10/24 done
- Failed items: 14
- Requirements: 0/84 met
- Fidelity: 0.0%
- Abort reason: none

## Outstanding findings

- Fidelity requirement not verifiable (no repair item created): TC001: Verificar el cálculo correcto del CEV y su actualización en tiempo real. → El CEV se calcula correctamente según la fórmula y se recalcula automáticamente al modificar cualquier elemento, en menos de 1 segundo, sin recargar la página.
- Fidelity requirement not verifiable (no repair item created): TC002: Validar la advertencia/bloqueo al introducir una composición con suma de porcentajes superior al 100%. → El sistema debe advertir o bloquear la entrada si la suma de porcentajes de la aleación supera el 100%.
- Fidelity requirement not verifiable (no repair item created): TC003: Comprobar la pantalla simplificada para operarios, CEV y el indicador visual de aptitud. → La pantalla muestra el CEV vigente y un indicador visual de aptitud (verde / ámbar / rojo) legible a distancia de operación. La pantalla es usable en dispositivo táctil, con controles de tamaño adecuad…
- Fidelity requirement not verifiable (no repair item created): TC004: Verificar la clasificación de soldabilidad y las alertas/recomendaciones según el CEV. → El sistema clasifica la soldabilidad en al menos tres niveles (apta, con precauciones, no apta) según umbrales de CEV configurables. Cuando el CEV supera el umbral de riesgo, se muestra una alerta explíc…
- Fidelity requirement not verifiable (no repair item created): TC005: Validar que la optimización de costes propone una mezcla de mínimo coste que cumple el CEV objetivo. → El sistema devuelve una mezcla propuesta con las cantidades por material de mínimo coste que cumple todas las restricciones de composición y el CEV objetivo dentro de la tolerancia definida…
- Fidelity requirement not verifiable (no repair item created): TC006: Verificar la capacidad de crear, editar y desactivar materias primas y que la optimización usa precios vigentes. → El ingeniero puede crear, editar y desactivar materias primas con su precio y fecha de vigencia. Cada cálculo de optimización usa el precio vigente en la fecha del cálculo. El s…
- Fidelity requirement not verifiable (no repair item created): TC007: Verificar que el rol Operario solo accede a funciones permitidas y es restringido en otras. → Un usuario sin sesión válida no puede acceder a ninguna pantalla funcional del simulador. El rol operario solo accede a la vista táctil de cálculo y consulta, y recibe un error de autorización en el…
- Fidelity requirement not verifiable (no repair item created): TC008: Validar que el formulario de entrada admite todos los elementos químicos especificados. → El formulario admite al menos los elementos C, Mn, Si, P, S, Cr, Ni, Mo, Cu y V, con su unidad en % en peso. El CEV se recalcula automáticamente al modificar cualquier elemento, en menos de 1 segundo, s…
- Fidelity requirement not verifiable (no repair item created): TC009: Verificar la comparación de costes entre la receta actual y la mezcla optimizada. → La comparativa muestra coste de la receta actual, coste de la mezcla optimizada, ahorro en € y ahorro en %. La comparativa indica si la mezcla optimizada empeora la receta actual en coste.
- Fidelity requirement not verifiable (no repair item created): TC010: Validar el registro de cálculos de CEV y optimizaciones en el histórico. → Cada cálculo y optimización se persiste con fecha, usuario, composición de entrada, parámetros aplicados y resultado.
- Fidelity requirement not verifiable (no repair item created): TC011: Verificar la funcionalidad de filtrado en el histórico de cálculos. → El histórico permite filtrar por rango de fechas, usuario y rango de CEV.
- Fidelity requirement not verifiable (no repair item created): TC012: Validar la generación y descarga de reportes de colada en PDF/CSV. → El reporte se puede descargar en al menos un formato estructurado (PDF o CSV) desde la vista de resultado y desde el histórico. El reporte incluye fecha, usuario, composición, CEV, clasificación de soldabilidad y mezcla opt…
- Fidelity requirement not verifiable (no repair item created): TC013: Verificar el cálculo de CEV cuando la fuente de precios no está disponible. → El sistema continúa calculando el CEV de forma independiente si la fuente de precios no está disponible. La funcionalidad de optimización de costes debería indicar la indisponibilidad de precios o no estar disponib…
- Fidelity requirement not verifiable (no repair item created): TC014: Validar la carga y uso de precios de materias primas desde un archivo CSV. → El sistema debe procesar precios de mercado de chatarra y ferroaleaciones desde CSV o base de datos. Cada cálculo de optimización usa el precio vigente en la fecha del cálculo.
- Fidelity requirement not verifiable (no repair item created): TC015: Validar la exactitud del cálculo del CEV según la fórmula y la precisión de 3 decimales. → El CEV calculado por la aplicación coincide exactamente con el cálculo manual usando la fórmula CEV = C + Mn/6 + (Cr+Mo+V)/5 + (Ni+Cu)/15, mostrando 3 decimales.
- Fidelity requirement not verifiable (no repair item created): TC016: Verificar que el sistema rechaza o maneja correctamente valores de entrada inválidos. → El sistema debe rechazar valores fuera de rango (ej. negativos o no numéricos) y/o mostrar un mensaje de error apropiado. Para valores excesivamente altos, el sistema debe validar la entrada o advertir.
- Fidelity requirement not verifiable (no repair item created): TC017: Validar la actualización de precios desde base de datos SQL interna y su impacto en optimización. → El sistema procesa precios de mercado de chatarra y ferroaleaciones desde CSV o base de datos. La optimización de costes utiliza el precio actualizado desde la base de datos SQL interna.
- Fidelity requirement not verifiable (no repair item created): TC018: Verificar que las llamadas directas a la API para funciones de ingeniero están protegidas. → La API REST debe estar protegida. Un usuario sin sesión válida no puede acceder a ninguna pantalla funcional del simulador. El rol operario recibe un error de autorización en el resto de endpoints (i…
- Fidelity requirement not verifiable (no repair item created): TC019: Verificar el comportamiento cuando no se encuentran materias primas adecuadas para la optimización. → El sistema debe indicar claramente que no se pudo encontrar una mezcla optimizada que cumpla todas las restricciones, o proponer la mejor opción posible con una advertencia.
- Fidelity requirement not verifiable (no repair item created): TC020: Verificar la clasificación de soldabilidad cuando el CEV coincide con los umbrales exactos. → El sistema clasifica la soldabilidad correctamente en los niveles (apta, con precauciones, no apta) según los umbrales de CEV configurables, incluso en los límites exactos. Se muestra una alerta exp…
- Fidelity requirement not verifiable (no repair item created): TC021: Verificar que el reporte de colada incluye todos los campos especificados, incluso sin optimización. → El reporte incluye fecha, usuario, composición, CEV, clasificación de soldabilidad. La sección de mezcla optimizada con coste se maneja adecuadamente cuando no aplica.
- Fidelity requirement not verifiable (no repair item created): NFRNFR_001: Verificar que el cálculo de CEV es instantáneo (<1s) al modificar elementos. → El valor del CEV se recalcula automáticamente en la interfaz en menos de 1 segundo tras cada modificación, sin recargar la página.
- Fidelity requirement not verifiable (no repair item created): NFRNFR_002: Comprobar la disponibilidad del sistema y el rendimiento bajo carga concurrente. → El sistema mantiene una disponibilidad de al menos 99% y los tiempos de respuesta para los cálculos de CEV y optimización se mantienen por debajo de 3 segundos para el 95% de las solicitudes.
- Fidelity requirement not verifiable (no repair item created): NFRNFR_003: Verificar que el rol Operario solo accede a funciones autorizadas. → El Operario accede exitosamente a la vista táctil de cálculo y consulta. Para las demás pantallas (Optimación, Gestión de Materias Primas, Histórico), el sistema muestra un error de autorización o redirige a una página…
- Fidelity requirement not verifiable (no repair item created): NFRNFR_004: Verificar el comportamiento del sistema ante un archivo CSV de precios corrupto. → El sistema debe manejar la corrupción del archivo CSV de precios de manera robusta, registrando un error y, idealmente, utilizando la última configuración válida o mostrando un mensaje claro al usuario so…
- Fidelity requirement not verifiable (no repair item created): NFRNFR_005: Verificar que el cálculo de CEV funciona si la fuente de precios no está disponible. → El cálculo del CEV se realiza correctamente y se muestra el resultado. Al intentar la optimización de costes, el sistema informa que la funcionalidad no está disponible debido a la falta de datos de p…
- Fidelity requirement not verifiable (no repair item created): NFRNFR_006: Medir el tiempo de respuesta para el cálculo de optimización de costes. → El cálculo de la mezcla optimizada se completa y los resultados se muestran en menos de 3 segundos.
- Fidelity requirement not verifiable (no repair item created): NFRNFR_007: Verificar que la optimización de costes usa el precio vigente de las materias primas. → El cálculo de optimización de costes utiliza exclusivamente el precio de la materia prima que está vigente en la fecha del cálculo.
- Fidelity requirement not verifiable (no repair item created): NFR001: Verificar que las APIs de backend requieren autenticación/autorización. → La API debe rechazar la solicitud con un código de estado 401 (No autorizado) o 403 (Prohibido) para todos los endpoints protegidos.
- Fidelity requirement not verifiable (no repair item created): NFR002: Evaluar el tiempo de carga de un catálogo extenso de materias primas. → La lista de materias primas se carga y es completamente navegable en menos de 3 segundos.
- Fidelity requirement not verifiable (no repair item created): NFR003: Verificar la persistencia de cálculos y optimizaciones en el histórico. → Cada cálculo y optimización se persiste con fecha, usuario, composición de entrada, parámetros aplicados y resultado, y es correctamente visible en el histórico.
- Fidelity requirement not verifiable (no repair item created): NFR004: Validar la usabilidad de la pantalla del operario en dispositivos táctiles. → La pantalla es usable en dispositivo táctil, con controles de tamaño adecuado y sin necesidad de scroll horizontal. Muestra el CEV vigente y un indicador visual de aptitud (verde / ámbar / rojo) legible a distanci…
- Fidelity requirement not verifiable (no repair item created): NFR005: Verificar el despliegue y funcionamiento de los servicios en contenedores Docker. → Los contenedores Docker se inician sin errores, la aplicación es accesible y todas las funcionalidades (cálculo de CEV, optimización) operan correctamente.
- Fidelity requirement not verifiable (no repair item created): NFR006: Comprobar la capacidad de restaurar la base de datos de precios desde un backup. → La base de datos de precios se restaura exitosamente desde el backup y los datos de materias primas son consistentes con el estado del backup.
- Fidelity requirement not verifiable (no repair item created): NFR007: Medir la latencia de las llamadas a la API REST del backend. → El tiempo de respuesta promedio de las APIs para el cálculo de CEV y optimización es inferior a 100 ms, y el 95% de las solicitudes se completan en menos de 300 ms.
- Fidelity requirement not verifiable (no repair item created): NFR008: Probar la vulnerabilidad a inyección SQL en la gestión de materias primas. → El sistema debe sanitizar las entradas, prevenir la ejecución de sentencias SQL maliciosas, no mostrar errores internos de la base de datos ni permitir la manipulación de la estructura de la base de datos.
- Fidelity requirement not verifiable (no repair item created): NFR009: Verificar el tiempo de generación de reportes con un histórico extenso. → El reporte se genera y está disponible para descarga en menos de 5 segundos.
- Fidelity requirement not verifiable (no repair item created): NFR010: Prevenir ataques de Cross-Site Scripting (XSS) en campos de entrada. → El sistema sanitiza las entradas de usuario, impidiendo la ejecución de scripts maliciosos y mostrando el contenido de forma segura.
- Fidelity requirement not verifiable (no repair item created): NFR011: Validar el manejo de errores del backend para solicitudes API inválidas. → La API debe devolver un código de estado HTTP 400 (Bad Request) y un mensaje de error claro y descriptivo que indique la causa del fallo en cada caso.
- Fidelity requirement not verifiable (no repair item created): NFR012: Evaluar el rendimiento de la base de datos SQLite bajo carga de actualizaciones concurrentes. → Las operaciones de actualización y creación de materias primas se completan en menos de 1 segundo, y la integridad de los datos en la base de datos SQLite se mantiene sin conflictos ni corrupcion…
- Fidelity requirement not verifiable (no repair item created): Web application for real-time CEV calculation and cost optimization
- Fidelity requirement not verifiable (no repair item created): Audience includes Operario de Horno and Ingeniero Metalúrgico
- Fidelity requirement not verifiable (no repair item created): CEV recalculated instantly when values change
- Fidelity requirement not verifiable (no repair item created): Canonical CEV formula
- Fidelity requirement not verifiable (no repair item created): 100% CEV calculation accuracy against the approved reference
- Fidelity requirement not verifiable (no repair item created): Less than three-second calculation time
- Fidelity requirement not verifiable (no repair item created): Visual soldability code
- Fidelity requirement not verifiable (no repair item created): Warn or block when material percentages exceed 100%
- Fidelity requirement not verifiable (no repair item created): Optimized mixture must not raise CEV above the safe limit
- Fidelity requirement not verifiable (no repair item created): Maximize profit per tonne by reducing expensive raw-material use
- Fidelity requirement not verifiable (no repair item created): At least 3% average cost reduction target
- Fidelity requirement not verifiable (no repair item created): CEV remains usable when the price source is unavailable
- Fidelity requirement not verifiable (no repair item created): Ingest prices from `precios_mercado.csv`
- Fidelity requirement not verifiable (no repair item created): Internal SQL database for manual price updates
- Fidelity requirement not verifiable (no repair item created): Current prices and historical price records
- Fidelity requirement not verifiable (no repair item created): Manual price update by the metallurgical engineer
- Fidelity requirement not verifiable (no repair item created): Synchronous REST communication between frontend and backend
- Fidelity requirement not verifiable (no repair item created): Thousands of local calculations and 50–100 backend calls daily
- Fidelity requirement not verifiable (no repair item created): Record and audit alloy calculations
- Fidelity requirement not verifiable (no repair item created): Analytics by date and heat
- Fidelity requirement not verifiable (no repair item created): Trends and historical data
- Fidelity requirement not verifiable (no repair item created): Adoption KPI of at least 80% of heats recorded
- Fidelity requirement not verifiable (no repair item created): CSAT target of at least 4/5
- Fidelity requirement not verifiable (no repair item created): At least 99% uptime
- Fidelity requirement not verifiable (no repair item created): Security through input validation and REST API protection
- Fidelity requirement not verifiable (no repair item created): Input validation
- Fidelity requirement not verifiable (no repair item created): Exactly 100% CEV match to reference
- Fidelity requirement not verifiable (no repair item created): Optimize without compromising alloy quality
- Fidelity requirement not verifiable (no repair item created): Existing optimization engine integration
- Fidelity requirement not verifiable (no repair item created): External AI support constraint
- Fidelity requirement not verifiable (no repair item created): Apply optimized mixture
- Fidelity requirement not verifiable (no repair item created): Figma fidelity
- Fidelity requirement not verifiable (no repair item created): Interfaz de Entrada de Composición page
- Fidelity requirement not verifiable (no repair item created): Sugerencias de Optimización de Costes page
- Fidelity requirement not verifiable (no repair item created): Vistas Analíticas page
- Fidelity requirement not verifiable (no repair item created): Design System Overview frame
- Fidelity requirement not verifiable (no repair item created): Components Library frame
- Fidelity requirement not verifiable (no repair item created): Seven mandatory base components
- Fidelity requirement not verifiable (no repair item created): Responsive tablet use
- Fidelity requirement not verifiable (no repair item created): Docker portability
- Fidelity requirement not verifiable (no repair item created): CI/CD for development, staging, and production
- Fidelity requirement not verifiable (no repair item created): Daily SQLite backups
- Fidelity requirement not verifiable (no repair item created): Redundant backend instances
- Fidelity requirement not verifiable (no repair item created): Documentation for price updates
- Ítem no completado: Foundation — shared types, interfaces, DB schemas, config
- Ítem no completado: Backend - Health, Raw Materials API & Price Ingestion
- Ítem no completado: Backend - CEV Calculation & Alloy Records API
- Ítem no completado: Backend - Cost Optimization API
- Ítem no completado: Backend - Analytics API
- Ítem no completado: Frontend - Design System & Base Components (1/3)
- Ítem no completado: Frontend - Design System & Base Components (2/3)
- Ítem no completado: Frontend - Design System & Base Components (3/3)
- Ítem no completado: Frontend - CEV Input Interface
- Ítem no completado: Frontend - Cost Optimization Suggestions
- Ítem no completado: Frontend - Analytical Views
- Ítem no completado: Infrastructure & Deployment (1/2)
- Ítem no completado: Infrastructure & Deployment (2/2)
- Ítem no completado: Integration gate failed after repair attempts
- [project] UI/UX design contract is not implemented: the approved contract defines design tokens but no frontend `styles/tokens.ts` file exists. Create it, export a single `tokens` object, and use the exact approved values.
- [project] Functional requirement not implemented: **Web application for real-time CEV calculation and cost optimization**
Implementation approach: React SPA with three operational routes plus the two Figma reference frames; FastAPI REST backend
Missing file(s) that must be created/completed:
  - `frontend/src/App.tsx`
  - `frontend/src/pages/*.tsx`
  - `backend/app/main.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Audience includes Operario de Horno and Ingeniero Metalúrgico**
Implementation approach: Role label in navigation; touch-oriented composition interface; analytical interface for engineering review
Missing file(s) that must be created/completed:
  - `frontend/src/components/ui/NavegacionPrincipal.tsx`
  - `frontend/src/pages/InterfazEntradaComposicion.tsx`
  - `frontend/src/pages/VistasAnaliticas.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **CEV recalculated instantly when values change**
Implementation approach: Synchronous TypeScript calculation in `useCEVCalculator`; every valid update immediately updates CEV, soldability, validity, and safety
Missing file(s) that must be created/completed:
  - `frontend/src/hooks/useCEVCalculator.ts`
  - `frontend/src/pages/InterfazEntradaComposicion.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Canonical CEV formula**
Implementation approach: Identical Python and TypeScript implementation of `C + Mn/6 + (Cr + Mo + V)/5 + (Ni + Cu)/15`
Missing file(s) that must be created/completed:
  - `backend/app/cev.py`
  - `frontend/src/hooks/useCEVCalculator.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **100% CEV calculation accuracy against the approved reference**
Implementation approach: Single documented formula, shared field semantics, four-decimal presentation rounding, and exhaustive QA validation against reference values
Missing file(s) that must be created/completed:
  - `backend/app/cev.py`
  - `frontend/src/hooks/useCEVCalculator.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Less than three-second calculation time**
Implementation approach: Local calculation does not await network requests; backend uses Pandas only for price data and avoids per-call CSV reads
Missing file(s) that must be created/completed:
  - `frontend/src/hooks/useCEVCalculator.ts`
  - `backend/app/cev.py`
  - `backend/app/pricing.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Visual soldability code**
Implementation approach: `IndicadorEstado` maps `Bajo`, `Medio`, and `Alto` to token-based semantic classes
Missing file(s) that must be created/completed:
  - `frontend/src/components/ui/IndicadorEstado.tsx`
  - `frontend/src/hooks/useCEVCalculator.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Warn or block when material percentages exceed 100%**
Implementation approach: `sumaPorcentajes`, `superaCien`, and `esValida`; warning is displayed and optimization is disabled
Missing file(s) that must be created/completed:
  - `backend/app/cev.py`
  - `backend/app/schemas.py`
  - `frontend/src/hooks/useCEVCalculator.ts`
  - `frontend/src/pages/InterfazEntradaComposicion.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Optimized mixture must not raise CEV above the safe limit**
Implementation approach: Optimizer enforces strict `CEV < 0.45` and `cumple_cev_seguro`; invalid candidates are rejected
Missing file(s) that must be created/completed:
  - `backend/app/optimizer.py`
  - `backend/app/schemas.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Maximize profit per tonne by reducing expensive raw-material use**
Implementation approach: Cost minimization is the primary optimizer objective; response includes current cost, optimized cost, absolute saving, and weighted percentage saving
Missing file(s) that must be created/completed:
  - `backend/app/optimizer.py`
  - `frontend/src/pages/SugerenciasOptimizacionCostes.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **At least 3% average cost reduction target**
Implementation approach: Optimizer prioritizes lower cost under the CEV and composition constraints; analytics exposes average percentage saving
Missing file(s) that must be created/completed:
  - `backend/app/optimizer.py`
  - `backend/app/analytics.py`
  - `frontend/src/pages/VistasAnaliticas.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **CEV remains usable when the price source is unavailable**
Implementation approach: CEV endpoint does not access prices; frontend computes locally; optimization returns honest HTTP `503` rather than fabricated costs
Missing file(s) that must be created/completed:
  - `backend/app/routers/calculos.py`
  - `backend/app/optimizer.py`
  - `frontend/src/hooks/useCEVCalculator.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Ingest prices from `precios_mercado.csv`**
Implementation approach: Startup CSV ingestion and manual CSV import endpoint
Missing file(s) that must be created/completed:
  - `backend/app/pricing.py`
  - `backend/app/routers/materiales.py`
  - `backend/data/precios_mercado.csv`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Internal SQL database for manual price updates**
Implementation approach: SQLite persistence for `MATERIA_PRIMA` and `HISTORIAL_PRECIOS`, with manual create/update endpoints
Missing file(s) that must be created/completed:
  - `backend/app/models.py`
  - `backend/app/routers/materiales.py`
  - `backend/app/persistence.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Current prices and historical price records**
Implementation approach: Update of `precio_actual` always appends a history row; history endpoint returns descending records
Missing file(s) that must be created/completed:
  - `backend/app/pricing.py`
  - `backend/app/persistence.py`
  - `backend/app/routers/materiales.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Manual price update by the metallurgical engineer**
Implementation approach: Editable material price UI on the analytical/engineering workflow, calling the update endpoint
Missing file(s) that must be created/completed:
  - `frontend/src/api/materiales.ts`
  - `frontend/src/pages/VistasAnaliticas.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Synchronous REST communication between frontend and backend**
Implementation approach: Axios REST client with typed request and response modules
Missing file(s) that must be created/completed:
  - `frontend/src/api/*.ts`
  - `frontend/src/api/client.ts`
  - `backend/app/routers/*.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Thousands of local calculations and 50–100 backend calls daily**
Implementation approach: Local-first calculation, efficient SQLite price lookup, no price access for CEV calculation
Missing file(s) that must be created/completed:
  - `frontend/src/hooks/useCEVCalculator.ts`
  - `backend/app/pricing.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Record and audit alloy calculations**
Implementation approach: `ALEACION` stores elemental percentages, CEV, soldability, and estimated cost; composition rows store material percentages
Missing file(s) that must be created/completed:
  - `backend/app/models.py`
  - `backend/app/routers/aleaciones.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Analytics by date and heat**
Implementation approach: `fecha_inicio`, `fecha_fin`, and `colada` filters; heat identifier maps to `ALEACION.nombre`
Missing file(s) that must be created/completed:
  - `backend/app/analytics.py`
  - `backend/app/routers/analiticas.py`
  - `frontend/src/hooks/useAnalytics.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Trends and historical data**
Implementation approach: Analytics response includes summary, trend points, and historical rows; charts and table render them
Missing file(s) that must be created/completed:
  - `frontend/src/pages/VistasAnaliticas.tsx`
  - `frontend/src/components/ui/Grafico.tsx`
  - `frontend/src/components/ui/TablaDatos.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Adoption KPI of at least 80% of heats recorded**
Implementation approach: Persisted `ALEACION` records provide the denominator for registered heats; analytics exposes `coladas_registradas`
Missing file(s) that must be created/completed:
  - `backend/app/routers/aleaciones.py`
  - `backend/app/analytics.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **CSAT target of at least 4/5**
Implementation approach: Product usability and immediate feedback support the target; CSAT is a business KPI and not represented by an invented database entity
Missing file(s) that must be created/completed:
  - `frontend/src/pages/InterfazEntradaComposicion.tsx`
  - `frontend/src/pages/SugerenciasOptimizacionCostes.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **At least 99% uptime**
Implementation approach: Health endpoint, rolling-update deployment workflow, redundant backend instances in Compose, and no local state dependency for CEV
Missing file(s) that must be created/completed:
  - `backend/app/main.py`
  - `docker-compose.yml`
  - `github/workflows/ci-cd.yml`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Security through input validation and REST API protection**
Implementation approach: Pydantic validation, bounded query parameters, restrictive CORS, request-size validation, conflict responses, no secret logging
Missing file(s) that must be created/completed:
  - `backend/app/schemas.py`
  - `backend/app/config.py`
  - `backend/app/main.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Input validation**
Implementation approach: Numeric ranges, allowed material type, composition total validation, strict CEV limit, malformed CSV handling
Missing file(s) that must be created/completed:
  - `backend/app/schemas.py`
  - `backend/app/cev.py`
  - `backend/app/pricing.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Exactly 100% CEV match to reference**
Implementation approach: Formula, rounding, classification boundaries, and frontend/backend parity are fixed in this specification
Missing file(s) that must be created/completed:
  - `backend/app/cev.py`
  - `frontend/src/hooks/useCEVCalculator.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Optimize without compromising alloy quality**
Implementation approach: Target elemental profile is a hard constraint; candidate must preserve target composition and remain below safe CEV
Missing file(s) that must be created/completed:
  - `backend/app/optimizer.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Existing optimization engine integration**
Implementation approach: `optimizar_mezcla` is the stable integration boundary for the existing engine; the implementation uses a deterministic constrained optimizer and contains no mock engine or fabricated result
Missing file(s) that must be created/completed:
  - `backend/app/optimizer.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **External AI support constraint**
Implementation approach: No unavailable AI service is required at runtime; the deterministic optimizer is production code. An external AI optimizer would require a separately approved integration and contract
Missing file(s) that must be created/completed:
  - `backend/app/optimizer.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Apply optimized mixture**
Implementation approach: Save the optimized alloy and composition transactionally through the alloy endpoint
Missing file(s) that must be created/completed:
  - `frontend/src/hooks/useOptimization.ts`
  - `backend/app/routers/aleaciones.py`
  - `backend/app/persistence.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Figma fidelity**
Implementation approach: Exact frame order, dimensions, section hierarchy, literal colors, and named base components are implemented in page and component files
Missing file(s) that must be created/completed:
  - `frontend/src/pages/*.tsx`
  - `frontend/src/components/ui/*.tsx`
  - `frontend/src/styles/tokens.ts`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Interfaz de Entrada de Composición page**
Implementation approach: Full page implementation of node `6:2` with header, hero, input panel, results, action, and footer
Missing file(s) that must be created/completed:
  - `frontend/src/pages/InterfazEntradaComposicion.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Sugerencias de Optimización de Costes page**
Implementation approach: Full page implementation of node `5:2` with navigation, header, comparison, actions, and footer
Missing file(s) that must be created/completed:
  - `frontend/src/pages/SugerenciasOptimizacionCostes.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Vistas Analíticas page**
Implementation approach: Full page implementation of node `7:2` with navigation, filters, KPIs, charts, history table, and footer
Missing file(s) that must be created/completed:
  - `frontend/src/pages/VistasAnaliticas.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Design System Overview frame**
Implementation approach: Full implementation of node `3:2` and all six section regions
Missing file(s) that must be created/completed:
  - `frontend/src/pages/DesignSystemOverview.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Components Library frame**
Implementation approach: Full implementation of node `4:2` and all three component rows
Missing file(s) that must be created/completed:
  - `frontend/src/pages/ComponentsLibrary.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Seven mandatory base components**
Implementation approach: Seven individual named component modules under `components/ui`
Missing file(s) that must be created/completed:
  - `frontend/src/components/ui/*.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Responsive tablet use**
Implementation approach: CSS layouts preserve the Figma desktop dimensions while stacking controls and comparison cards on tablet widths without removing content
Missing file(s) that must be created/completed:
  - `frontend/src/styles/global.css`
  - `frontend/src/pages/*.tsx`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Docker portability**
Implementation approach: Backend and frontend Dockerfiles plus Compose deployment
Missing file(s) that must be created/completed:
  - `backend/Dockerfile`
  - `frontend/Dockerfile`
  - `docker-compose.yml`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **CI/CD for development, staging, and production**
Implementation approach: GitHub Actions builds images, validates source and frontend compilation, publishes images, and performs rolling Compose deployment through a self-hosted deployment runner
Missing file(s) that must be created/completed:
  - `github/workflows/ci-cd.yml`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Daily SQLite backups**
Implementation approach: Backup script uses SQLite’s online backup mechanism and writes one backup per day without exposing a database port
Missing file(s) that must be created/completed:
  - `backend/scripts/backup_sqlite.py`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Redundant backend instances**
Implementation approach: Compose defines two backend replicas behind the configured reverse proxy/load-balancer target; no cloud provider is invented
Missing file(s) that must be created/completed:
  - `docker-compose.yml`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
- [project] Functional requirement not implemented: **Documentation for price updates**
Implementation approach: User-facing instructions explain CSV columns, manual updates, validation errors, and fallback behavior
Missing file(s) that must be created/completed:
  - `docs/ACTUALIZACION_PRECIOS.md`
  - `README.md`

Instructions:
1. Create each missing file with complete, production-ready implementation.
2. Wire it into the service that owns it (add imports, register routes, etc.).
3. Do NOT create stub or placeholder files — implement the full logic.
4. Verify the file exists on disk after writing.
