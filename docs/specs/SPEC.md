# SPEC.md

## 1. TECHNOLOGY STACK

### Backend

- **Language:** Python `3.11`
- **API framework:** FastAPI `0.115.6`
- **Data validation:** Pydantic `2.10.4`
- **HTTP server:** Uvicorn `0.32.1`
- **Data processing:** Pandas `2.2.3`
- **ORM:** SQLAlchemy `2.0.36`
- **Database:** SQLite `3.46.1`
- **Database driver:** Built-in Python `sqlite3`, accessed through SQLAlchemy
- **Containerization:** Docker `24.0.9`
- **Container orchestration:** Docker Compose `2.29.7`

The backend is a single FastAPI service. It owns the CEV calculation, the deterministic constrained cost optimizer, CSV/SQLite price ingestion, persistence, REST endpoints, and analytical queries.

### Frontend

- **Language:** TypeScript `5.6.3`
- **UI framework:** React `18.3.1`
- **Build tool:** Vite `5.4.11`
- **HTTP client:** Axios `1.7.9`
- **Routing:** React Router DOM `6.28.0`
- **Charts:** Recharts `2.13.3`
- **Icons:** Lucide React `0.468.0`
- **Styling:** CSS Modules with TypeScript and CSS custom properties; no Tailwind dependency
- **Type checking:** TypeScript compiler `5.6.3`

The browser performs CEV and soldability calculation locally on every input change. Backend optimization is asynchronous and does not block local CEV feedback.

### Domain calculation

The canonical CEV formula is:

```text
CEV = C + Mn/6 + (Cr + Mo + V)/5 + (Ni + Cu)/15
```

All elemental values are percentages by mass. The same formula and rounding rules are implemented in Python and TypeScript:

- Intermediate calculations use full floating-point precision.
- The displayed and persisted `cev_calculado` is rounded to four decimal places.
- The safe CEV condition is strictly `CEV < 0.45`.
- An optimized mixture is accepted only when its CEV is strictly below `0.45`.

Soldability classification is deterministic:

- `Bajo`: `CEV < 0.45`
- `Medio`: `0.45 <= CEV < 0.65`
- `Alto`: `CEV >= 0.65`

The frontend and backend must return the same category for the same unrounded CEV.

### Performance and operating targets

- Local CEV feedback is rendered synchronously after each numeric change, without awaiting the backend.
- Backend calculation and optimization requests must complete in less than three seconds under the expected load.
- The architecture supports thousands of local calculations per day and 50–100 backend calls per day.
- The deployment target is at least 99% uptime.
- The primary business target is at least 3% reduction in average material cost per recorded heat.
- The target user satisfaction is CSAT of at least 4/5.
- The target adoption rate is at least 80% of heats recorded through the application.

---

## 2. DATA CONTRACTS

### 2.1 Authoritative persistence contract

The SQLite schema must use the following table and field names exactly. SQLAlchemy class names may be PascalCase, but each class must set `__tablename__` to the exact uppercase table name below.

```python
from datetime import datetime
from typing import List, Optional

from pydantic import BaseModel, ConfigDict, Field
from sqlalchemy import DateTime, Float, ForeignKey, String
from sqlalchemy.orm import DeclarativeBase, Mapped, mapped_column, relationship


class Base(DeclarativeBase):
    pass


class MateriaPrima(Base):
    __tablename__ = "MATERIA_PRIMA"

    id: Mapped[str] = mapped_column(String, primary_key=True)
    nombre: Mapped[str] = mapped_column(String, nullable=False)
    tipo: Mapped[str] = mapped_column(String, nullable=False)
    precio_actual: Mapped[float] = mapped_column(Float, nullable=False)
    unidad_medida: Mapped[str] = mapped_column(String, nullable=False)
    fecha_actualizacion: Mapped[datetime] = mapped_column(
        DateTime,
        nullable=False,
    )

    composicion_aleacion: Mapped[List["ComposicionAleacion"]] = relationship(
        back_populates="materia_prima",
        cascade="all, delete-orphan",
    )
    historial_precios: Mapped[List["HistorialPrecios"]] = relationship(
        back_populates="materia_prima",
        cascade="all, delete-orphan",
    )


class Aleacion(Base):
    __tablename__ = "ALEACION"

    id: Mapped[str] = mapped_column(String, primary_key=True)
    nombre: Mapped[str] = mapped_column(String, nullable=False)
    porcentaje_carbon: Mapped[float] = mapped_column(Float, nullable=False)
    porcentaje_manganeso: Mapped[float] = mapped_column(Float, nullable=False)
    porcentaje_cromo: Mapped[float] = mapped_column(Float, nullable=False)
    porcentaje_molibdeno: Mapped[float] = mapped_column(Float, nullable=False)
    porcentaje_vanadio: Mapped[float] = mapped_column(Float, nullable=False)
    porcentaje_niquel: Mapped[float] = mapped_column(Float, nullable=False)
    porcentaje_cobre: Mapped[float] = mapped_column(Float, nullable=False)
    cev_calculado: Mapped[float] = mapped_column(Float, nullable=False)
    soldabilidad: Mapped[str] = mapped_column(String, nullable=False)
    coste_estimado: Mapped[float] = mapped_column(Float, nullable=False)

    composicion: Mapped[List["ComposicionAleacion"]] = relationship(
        back_populates="aleacion",
        cascade="all, delete-orphan",
    )


class ComposicionAleacion(Base):
    __tablename__ = "COMPOSICION_ALEACION"

    aleacion_id: Mapped[str] = mapped_column(
        String,
        ForeignKey("ALEACION.id"),
        primary_key=True,
    )
    materia_prima_id: Mapped[str] = mapped_column(
        String,
        ForeignKey("MATERIA_PRIMA.id"),
        primary_key=True,
    )
    porcentaje_peso: Mapped[float] = mapped_column(Float, nullable=False)

    aleacion: Mapped[Aleacion] = relationship(back_populates="composicion")
    materia_prima: Mapped[MateriaPrima] = relationship(
        back_populates="composicion_aleacion"
    )


class HistorialPrecios(Base):
    __tablename__ = "HISTORIAL_PRECIOS"

    id: Mapped[str] = mapped_column(String, primary_key=True)
    materia_prima_id: Mapped[str] = mapped_column(
        String,
        ForeignKey("MATERIA_PRIMA.id"),
        nullable=False,
    )
    precio: Mapped[float] = mapped_column(Float, nullable=False)
    fecha_registro: Mapped[datetime] = mapped_column(
        DateTime,
        nullable=False,
    )

    materia_prima: Mapped[MateriaPrima] = relationship(
        back_populates="historial_precios"
    )
```

The following relationships are mandatory:

- One `MATERIA_PRIMA` may be referenced by many `COMPOSICION_ALEACION` rows.
- One `ALEACION` may be referenced by many `COMPOSICION_ALEACION` rows.
- One `MATERIA_PRIMA` may have many `HISTORIAL_PRECIOS` rows.
- `COMPOSICION_ALEACION.aleacion_id` and `COMPOSICION_ALEACION.materia_prima_id` form a composite primary key.
- Deleting a `MATERIA_PRIMA` used by a composition must be rejected with HTTP `409`.
- Updating `MATERIA_PRIMA.precio_actual` must append a `HISTORIAL_PRECIOS` row.

### 2.2 Python API models

#### Entity response models

```python
class MateriaPrimaResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: str
    nombre: str
    tipo: str
    precio_actual: float
    unidad_medida: str
    fecha_actualizacion: datetime


class ComposicionAleacionResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    aleacion_id: str
    materia_prima_id: str
    porcentaje_peso: float


class HistorialPreciosResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: str
    materia_prima_id: str
    precio: float
    fecha_registro: datetime


class AleacionResponse(BaseModel):
    model_config = ConfigDict(from_attributes=True)

    id: str
    nombre: str
    porcentaje_carbon: float
    porcentaje_manganeso: float
    porcentaje_cromo: float
    porcentaje_molibdeno: float
    porcentaje_vanadio: float
    porcentaje_niquel: float
    porcentaje_cobre: float
    cev_calculado: float
    soldabilidad: str
    coste_estimado: float
    composicion: List[ComposicionAleacionResponse] = []
```

#### Operational models

```python
class ElementosAleacion(BaseModel):
    porcentaje_carbon: float = Field(ge=0)
    porcentaje_manganeso: float = Field(ge=0)
    porcentaje_cromo: float = Field(ge=0)
    porcentaje_molibdeno: float = Field(ge=0)
    porcentaje_vanadio: float = Field(ge=0)
    porcentaje_niquel: float = Field(ge=0)
    porcentaje_cobre: float = Field(ge=0)


class ComposicionElementalInput(BaseModel):
    porcentaje_carbon: float = Field(ge=0)
    porcentaje_manganeso: float = Field(ge=0)
    porcentaje_cromo: float = Field(ge=0)
    porcentaje_molibdeno: float = Field(ge=0)
    porcentaje_vanadio: float = Field(ge=0)
    porcentaje_niquel: float = Field(ge=0)
    porcentaje_cobre: float = Field(ge=0)


class MaterialOptimizable(BaseModel):
    materia_prima_id: str
    porcentaje_peso: float = Field(ge=0, le=100)
    composicion_elemental: ComposicionElementalInput


class CEVRequest(BaseModel):
    nombre: str = Field(min_length=1, max_length=120)
    elementos: ElementosAleacion
    composicion: List[MaterialOptimizable] = []
    coste_estimado: float | None = Field(default=None, ge=0)


class CEVResponse(BaseModel):
    nombre: str
    porcentaje_carbon: float
    porcentaje_manganeso: float
    porcentaje_cromo: float
    porcentaje_molibdeno: float
    porcentaje_vanadio: float
    porcentaje_niquel: float
    porcentaje_cobre: float
    cev_calculado: float
    soldabilidad: str
    suma_porcentajes: float
    supera_cien: bool
    es_valida: bool
    es_segura: bool
    coste_estimado: float | None


class OptimizacionRequest(BaseModel):
    nombre: str = Field(min_length=1, max_length=120)
    elementos_objetivo: ElementosAleacion
    materiales: List[MaterialOptimizable] = Field(min_length=1)
    precio_maximo_porcentaje: float = Field(default=100.0, gt=0, le=100)


class MezclaOptimizadaItem(BaseModel):
    materia_prima_id: str
    nombre: str
    porcentaje_peso: float
    coste_fuente: float
    coste_porcentaje: float
    cev_aportado: float


class OptimizacionResponse(BaseModel):
    nombre: str
    cev_actual: float
    soldabilidad_actual: str
    cev_optimizado: float
    soldabilidad_optimizada: str
    coste_actual: float
    coste_optimizado: float
    ahorro_estimado: float
    porcentaje_ahorro: float
    porcentaje_ahorro_pesado: float
    cumple_cev_seguro: bool
    cumple_objetivo_composicional: bool
    mezcla: List[MezclaOptimizadaItem]
    advertencias: List[str]
    mensaje: str


class ActualizarPrecioRequest(BaseModel):
    precio_actual: float = Field(ge=0)
    unidad_medida: str = Field(min_length=1, max_length=30)


class ImportarCSVRequest(BaseModel):
    csv_content: str = Field(min_length=1)


class ImportarCSVResponse(BaseModel):
    filas_procesadas: int
    filas_actualizadas: int
    filas_insertadas: int
    filas_rechazadas: int
    errores: List[str]


class GuardarAleacionRequest(BaseModel):
    nombre: str = Field(min_length=1, max_length=120)
    elementos: ElementosAleacion
    cev_calculado: float = Field(ge=0)
    soldabilidad: str
    coste_estimado: float = Field(ge=0)
    composicion: List[ComposicionAleacionCreate] = []


class ComposicionAleacionCreate(BaseModel):
    materia_prima_id: str
    porcentaje_peso: float = Field(ge=0, le=100)


class ResumenAnalitico(BaseModel):
    numero_coladas: int
    coste_medio: float
    cev_medio: float
    porcentaje_coladas_seguras: float
    ahorro_medio_porcentaje: float
    coladas_registradas: int


class PuntoTendenciaAnalitica(BaseModel):
    fecha: datetime
    cev_medio: float
    coste_medio: float


class HistoricoAnaliticoItem(BaseModel):
    id: str
    nombre: str
    fecha_actualizacion: datetime | None = None
    cev_calculado: float
    soldabilidad: str
    coste_estimado: float


class AnaliticaResponse(BaseModel):
    resumen: ResumenAnalitico
    tendencias: List[PuntoTendenciaAnalitica]
    historico: List[HistoricoAnaliticoItem]


class ErrorResponse(BaseModel):
    detail: str
    code: str
```

`ComposicionAleacionCreate` must be declared before `GuardarAleacionRequest` in the module.

### 2.3 TypeScript frontend contracts

```typescript
export interface ElementosAleacion {
  porcentaje_carbon: number;
  porcentaje_manganeso: number;
  porcentaje_cromo: number;
  porcentaje_molibdeno: number;
  porcentaje_vanadio: number;
  porcentaje_niquel: number;
  porcentaje_cobre: number;
}

export interface ComposicionElementalInput {
  porcentaje_carbon: number;
  porcentaje_manganeso: number;
  porcentaje_cromo: number;
  porcentaje_molibdeno: number;
  porcentaje_vanadio: number;
  porcentaje_niquel: number;
  porcentaje_cobre: number;
}

export interface MaterialOptimizable {
  materia_prima_id: string;
  porcentaje_peso: number;
  composicion_elemental: ComposicionElementalInput;
}

export interface MateriaPrima {
  id: string;
  nombre: string;
  tipo: string;
  precio_actual: number;
  unidad_medida: string;
  fecha_actualizacion: string;
}

export interface ComposicionAleacion {
  aleacion_id: string;
  materia_prima_id: string;
  porcentaje_peso: number;
}

export interface HistorialPrecios {
  id: string;
  materia_prima_id: string;
  precio: float;
  fecha_registro: string;
}

export interface Aleacion {
  id: string;
  nombre: string;
  porcentaje_carbon: number;
  porcentaje_manganeso: number;
  porcentaje_cromo: number;
  porcentaje_molibdeno: number;
  porcentaje_vanadio: number;
  porcentaje_niquel: number;
  porcentaje_cobre: number;
  cev_calculado: number;
  soldabilidad: string;
  coste_estimado: number;
  composicion: ComposicionAleacion[];
}

export interface CEVRequest {
  nombre: string;
  elementos: ElementosAleacion;
  composicion: MaterialOptimizable[];
  coste_estimado: number | null;
}

export interface CEVResponse {
  nombre: string;
  porcentaje_carbon: number;
  porcentaje_manganeso: number;
  porcentaje_cromo: number;
  porcentaje_molibdeno: number;
  porcentaje_vanadio: number;
  porcentaje_niquel: number;
  porcentaje_cobre: number;
  cev_calculado: number;
  soldabilidad: string;
  suma_porcentajes: number;
  supera_cien: boolean;
  es_valida: boolean;
  es_segura: boolean;
  coste_estimado: number | null;
}

export interface OptimizacionRequest {
  nombre: string;
  elementos_objetivo: ElementosAleacion;
  materiales: MaterialOptimizable[];
  precio_maximo_porcentaje: number;
}

export interface MezclaOptimizadaItem {
  materia_prima_id: string;
  nombre: string;
  porcentaje_peso: number;
  coste_fuente: number;
  coste_porcentaje: number;
  cev_aportado: number;
}

export interface OptimizacionResponse {
  nombre: string;
  cev_actual: number;
  soldabilidad_actual: string;
  cev_optimizado: number;
  soldabilidad_optimizada: string;
  coste_actual: number;
  coste_optimizado: number;
  ahorro_estimado: number;
  porcentaje_ahorro: number;
  porcentaje_ahorro_pesado: number;
  cumple_cev_seguro: boolean;
  cumple_objetivo_composicional: boolean;
  mezcla: MezclaOptimizadaItem[];
  advertencias: string[];
  mensaje: string;
}

export interface ResumenAnalitico {
  numero_coladas: number;
  coste_medio: number;
  cev_medio: number;
  porcentaje_coladas_seguras: number;
  ahorro_medio_porcentaje: number;
  coladas_registradas: number;
}

export interface PuntoTendenciaAnalitica {
  fecha: string;
  cev_medio: number;
  coste_medio: number;
}

export interface HistoricoAnaliticoItem {
  id: string;
  nombre: string;
  fecha_actualizacion: string | null;
  cev_calculado: number;
  soldabilidad: string;
  coste_estimado: number;
}

export interface AnaliticaResponse {
  resumen: ResumenAnalitico;
  tendencias: PuntoTendenciaAnalitica[];
  historico: HistoricoAnaliticoItem[];
}
```

The TypeScript `HistorialPrecios.precio` field must be `number`, not `float`; `float` is a Python type only. The corrected frontend contract is:

```typescript
export interface HistorialPrecios {
  id: string;
  materia_prima_id: string;
  precio: number;
  fecha_registro: string;
}
```

---

## 3. API ENDPOINTS

All application endpoints use the prefix `/api/v1`. JSON uses UTF-8. Validation failures return HTTP `422`; malformed CSV returns HTTP `400`; unavailable price data returns HTTP `503`; conflicting persistence operations return HTTP `409`.

### 3.1 Service health

#### `GET /health`

No request body.

Response:

```json
{
  "estado": "ok",
  "servicio": "simulador-cev",
  "base_datos": "ok",
  "fuente_precios": "sqlite"
}
```

The endpoint must verify that SQLite is reachable. It must remain healthy when the CSV source is unavailable if SQLite contains valid prices.

### 3.2 CEV calculation

#### `POST /api/v1/calculos/cev`

Request body: `CEVRequest`.

Response body: `CEVResponse`.

The endpoint calculates CEV and soldability independently of price availability. `coste_estimado` may be `null`. It returns:

- `suma_porcentajes`: sum of all material weight percentages.
- `supera_cien`: `true` when the sum is greater than `100`.
- `es_valida`: `false` for negative values, invalid material references, or a composition above `100%`.
- `es_segura`: `true` only when the composition is valid and CEV is below `0.45`.

The endpoint must not query the price source to calculate CEV.

### 3.3 Cost optimization

#### `POST /api/v1/optimize`

Request body: `OptimizacionRequest`.

Response body: `OptimizacionResponse`.

The optimizer must:

1. Load current prices from SQLite.
2. Validate that every referenced `materia_prima_id` exists and has a price.
3. Reject a request whose material percentages exceed `100%`.
4. Calculate the current CEV and current cost from the supplied composition.
5. Generate feasible candidate mixtures from the supplied materials.
6. Preserve the target elemental profile as a hard optimization constraint.
7. Require the candidate CEV to be strictly below `0.45`.
8. Minimize estimated material cost.
9. Return the cheapest feasible candidate and the estimated saving.
10. Include actionable warnings when no exact feasible candidate exists.

The optimizer is deterministic: identical requests and prices must produce identical output. It must not fabricate prices. If no usable price is available, return HTTP `503` with code `FUENTE_PRECIOS_NO_DISPONIBLE`; local CEV calculation remains usable.

### 3.4 Raw materials

#### `GET /api/v1/materiales-primas`

Optional query parameters:

- `tipo`: optional `chatarra` or `ferroaleacion`.
- `search`: optional case-insensitive name search.

Response:

```json
{
  "items": [],
  "total": 0
}
```

Each item must match `MateriaPrima`.

#### `POST /api/v1/materiales-primas`

Request body:

```json
{
  "id": "chatarra-a",
  "nombre": "Chatarra A",
  "tipo": "chatarra",
  "precio_actual": 420.0,
  "unidad_medida": "EUR/t"
}
```

Response: HTTP `201` with `MateriaPrima`.

The endpoint creates `MATERIA_PRIMA` and an initial `HISTORIAL_PRECIOS` record.

#### `PUT /api/v1/materiales-primas/{materia_prima_id}`

Request body: `ActualizarPrecioRequest`.

Response: `MateriaPrima`.

The endpoint updates `precio_actual`, `unidad_medida`, and `fecha_actualizacion`, and appends a history record.

#### `DELETE /api/v1/materiales-primas/{materia_prima_id}`

Response: HTTP `204` with no body.

The operation is rejected with HTTP `409` if the material is referenced by a composition.

#### `GET /api/v1/materiales-primas/{materia_prima_id}/historial-precios`

Response:

```json
{
  "items": [],
  "total": 0
}
```

Records are ordered by `fecha_registro` descending.

### 3.5 Price ingestion

#### `POST /api/v1/precios/importar-csv`

Request body: `ImportarCSVRequest`.

Response body: `ImportarCSVResponse`.

The CSV must contain these columns:

```csv
id,nombre,tipo,precio_actual,unidad_medida
```

Rules:

- `id`, `nombre`, `tipo`, `precio_actual`, and `unidad_medida` are required.
- `tipo` must be `chatarra` or `ferroaleacion`.
- `precio_actual` must be a non-negative number.
- Existing IDs are updated.
- Unknown IDs are inserted.
- Every accepted price update appends a `HISTORIAL_PRECIOS` row.
- Invalid rows are skipped and reported individually.
- The ingestion operation is transactional: invalid rows do not roll back valid rows, but the request completes with `filas_rechazadas` and a non-empty `errores` list.
- If the supplied CSV contains malformed headers, return HTTP `400`.

On backend startup, the configured CSV file is ingested when it exists and is readable. SQLite remains the operational source for current prices after ingestion.

### 3.6 Alloy records

#### `GET /api/v1/aleaciones`

Optional query parameters:

- `nombre`: optional case-insensitive heat or alloy-name filter.
- `limit`: integer from 1 to 500, default 100.
- `offset`: integer greater than or equal to 0, default 0.

Response:

```json
{
  "items": [],
  "total": 0
}
```

The `nombre` field is the persisted heat identifier used by the analytics filters, including the design example `COLADA 24-081`.

#### `POST /api/v1/aleaciones`

Request body: `GuardarAleacionRequest`.

Response: HTTP `201` with `AleacionResponse`.

Rules:

- The endpoint recalculates CEV and soldability; client-provided values are not trusted.
- The sum of `composicion.porcentaje_peso` must not exceed `100`.
- Every `materia_prima_id` must exist.
- `coste_estimado` is calculated from current prices when materials are provided. If prices are unavailable, the endpoint rejects persistence with HTTP `503`; CEV calculation itself remains available.
- The alloy and its composition rows are saved in one transaction.

#### `GET /api/v1/aleaciones/{aleacion_id}`

Response: `AleacionResponse`.

A missing alloy returns HTTP `404`.

### 3.7 Analytics

#### `GET /api/v1/analiticas/resumen`

Optional query parameters:

- `fecha_inicio`: optional ISO-8601 date or datetime.
- `fecha_fin`: optional ISO-8601 date or datetime.
- `colada`: optional case-insensitive `ALEACION.nombre` filter.

Response: `AnaliticaResponse`.

The response contains:

- Aggregate number of recorded heats.
- Average estimated cost.
- Average CEV.
- Percentage of heats with CEV below `0.45`.
- Average percentage saving.
- Registered heat count.
- Daily or monthly CEV and cost trend points.
- Historical alloy rows matching the filters.

#### `GET /api/v1/analiticas/historico`

The same optional filters as `/api/v1/analiticas/resumen` are accepted. Response: `AnaliticaResponse`.

### 3.8 Common error schema

All errors must use:

```json
{
  "detail": "Human-readable description",
  "code": "STABLE_MACHINE_READABLE_CODE"
}
```

The API must validate request body size, reject malformed identifiers, validate all numeric ranges, and set restrictive CORS origins from configuration.

---

## 4. FILE STRUCTURE

```text
.
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

No test files or test-only dependencies are included. Exhaustive CEV, soldability, optimization, and performance validation remains a mandatory QA activity performed against the production modules and documented API examples rather than introducing test-only production artifacts.

### Port table

| Service | Listening Port | Path |
|---|---:|---|
| backend | 21000 | `backend/` |
| frontend | 21300 | `frontend/` |

The backend Docker image must expose and start on `21000`. The frontend Docker image must expose and start on `21300`; Vite must be invoked with `--host 0.0.0.0 --port 21300`.

### Figma page files

Every frame in the UI/UX contract has a corresponding page file:

- `DesignSystemOverview.tsx` implements frame `Design System Overview`, node `3:2`.
- `ComponentsLibrary.tsx` implements frame `Components Library`, node `4:2`.
- `InterfazEntradaComposicion.tsx` implements frame `Interfaz de Entrada de Composición`, node `6:2`.
- `SugerenciasOptimizacionCostes.tsx` implements frame `Sugerencias de Optimización de Costes`, node `5:2`.
- `VistasAnaliticas.tsx` implements frame `Vistas Analíticas`, node `7:2`.

There is no shared backend directory because the approved architecture contains one backend service.

---

## 5. ENVIRONMENT VARIABLES

| Variable | Type | Description | Example |
|---|---|---|---|
| `BACKEND_HOST` | string | Address on which Uvicorn listens. | `0.0.0.0` |
| `BACKEND_PORT` | integer | Backend listening port. Must be `21000`. | `21000` |
| `DATABASE_URL` | string | SQLAlchemy SQLite connection URL. | `sqlite:////app/data/cev.db` |
| `CSV_PATH` | string | Absolute or container-relative path to `precios_mercado.csv`. | `/app/data/precios_mercado.csv` |
| `FRONTEND_PORT` | integer | Frontend listening port. Must be `21300`. | `21300` |
| `VITE_API_BASE_URL` | string | Base URL used by Axios for all API calls. | `http://localhost:21000/api/v1` |
| `CORS_ORIGINS` | string | Comma-separated allowed frontend origins. | `http://localhost:21300` |
| `CEV_MAX_SAFE` | float | Strict safe CEV boundary. | `0.45` |
| `LOG_LEVEL` | string | Application log level. | `INFO` |
| `SQLITE_BACKUP_DIR` | string | Directory where daily SQLite backups are written. | `/app/backups` |
| `SQLITE_BACKUP_FILENAME` | string | Filename template for a daily SQLite backup. | `cev-YYYY-MM-DD.db` |

No third-party API keys or external service URLs are required. The MVP has no email, messaging, ERP, cloud-provider, or external AI API dependency.

---

## 6. IMPORT CONTRACTS

### Backend

```python
from app.config import Settings, get_settings
from app.database import Base, SessionLocal, engine, get_db
from app.models import MateriaPrima, Aleacion, ComposicionAleacion, HistorialPrecios
from app.schemas import (
    ElementosAleacion,
    ComposicionElementalInput,
    MaterialOptimizable,
    CEVRequest,
    CEVResponse,
    OptimizacionRequest,
    OptimizacionResponse,
    MezclaOptimizadaItem,
    ActualizarPrecioRequest,
    ImportarCSVRequest,
    ImportarCSVResponse,
    GuardarAleacionRequest,
    ComposicionAleacionCreate,
    ResumenAnalitico,
    PuntoTendenciaAnalitica,
    HistoricoAnaliticoItem,
    AnaliticaResponse,
    ErrorResponse,
    MateriaPrimaResponse,
    ComposicionAleacionResponse,
    HistorialPreciosResponse,
    AleacionResponse,
)
from app.cev import calcular_cev, clasificar_soldabilidad, calcular_cev_material
from app.optimizer import optimizar_mezcla
from app.pricing import (
    cargar_precios_desde_csv,
    obtener_precios_actuales,
    actualizar_precio,
    importar_csv,
)
from app.persistence import guardar_aleacion, listar_aleaciones, obtener_aleacion
from app.analytics import obtener_analiticas
from app.routers.calculos import router as calculos_router
from app.routers.optimizacion import router as optimizacion_router
from app.routers.materiales import router as materiales_router
from app.routers.aleaciones import router as aleaciones_router
from app.routers.analiticas import router as analiticas_router
```

The FastAPI application must export:

```python
from app.main import app, lifespan
```

### Frontend

```typescript
import { tokens } from './styles/tokens';
import { apiClient } from './api/client';
import { calcularCEV } from './api/calculos';
import { optimizarMezcla } from './api/optimizacion';
import { listarMateriales, actualizarPrecio, importarPreciosCSV } from './api/materiales';
import { listarAleaciones, guardarAleacion } from './api/aleaciones';
import { obtenerAnaliticas } from './api/analiticas';
import { useCEVCalculator } from './hooks/useCEVCalculator';
import { useOptimization } from './hooks/useOptimization';
import { useAnalytics } from './hooks/useAnalytics';
import { NavegacionPrincipal } from './components/ui/NavegacionPrincipal';
import { BotonCTA } from './components/ui/BotonCTA';
import { TarjetaContenido } from './components/ui/TarjetaContenido';
import { CampoEntradaNumerico } from './components/ui/CampoEntradaNumerico';
import { IndicadorEstado } from './components/ui/IndicadorEstado';
import { TablaDatos } from './components/ui/TablaDatos';
import { Grafico } from './components/ui/Grafico';
import type {
  ElementosAleacion,
  CEVResponse,
  OptimizacionResponse,
  AnaliticaResponse,
} from './types/api';
```

Each base component must be a named export. `tokens` is the only source of color values in React components. Components must not invent additional hex colors or use inline styles for colors.

---

## 7. FRONTEND STATE & COMPONENT CONTRACTS

### `useCEVCalculator()`

Returns exactly:

```typescript
{
  nombre,
  setNombre,
  elementos,
  updateElemento,
  composicion,
  setComposicion,
  cev,
  soldabilidad,
  sumaPorcentajes,
  superaCien,
  esValida,
  esSegura,
  reset
}
```

Behavior:

- `updateElemento(campo, valor)` accepts only the seven exact elemental fields.
- The CEV and soldability values are recalculated locally and synchronously after every valid update.
- `sumaPorcentajes` is the sum of `composicion[].porcentaje_peso`.
- `superaCien` is true only when the sum is greater than `100`.
- `esValida` is false when the sum exceeds `100` or a negative percentage is entered.
- `esSegura` is true only when `esValida` is true and CEV is below `0.45`.
- `reset()` restores the initial empty elemental composition and clears the alloy name and material composition.

### `useOptimization()`

Returns exactly:

```typescript
{
  resultado,
  loading,
  error,
  optimizar,
  aplicar,
  aplicarLoading,
  aplicado
}
```

Behavior:

- `optimizar(request)` calls `POST /api/v1/optimize`.
- `aplicar(request)` saves the returned optimized mixture as an `ALEACION` and its `COMPOSICION_ALEACION` records.
- `aplicado` becomes true only after persistence succeeds.
- `error` contains the backend error detail when an operation fails.

### `useAnalytics()`

Returns exactly:

```typescript
{
  filtros,
  setFiltros,
  resumen,
  tendencias,
  historico,
  loading,
  error,
  refresh
}
```

`filtros` has exactly:

```typescript
{
  fecha_inicio: string;
  fecha_fin: string;
  colada: string;
}
```

### Base component contracts

```typescript
export interface NavegacionPrincipalProps {
  activePage: 'entrada' | 'optimizacion' | 'analiticas';
  userRole: 'Operario de Horno' | 'Ingeniero Metalúrgico';
  onNavigate: (
    page: 'entrada' | 'optimizacion' | 'analiticas'
  ) => void;
}
```

```typescript
export interface BotonCTAProps {
  label: string;
  onClick: () => void;
  variant?: 'primary' | 'secondary' | 'danger';
  disabled?: boolean;
  loading?: boolean;
  type?: 'button' | 'submit';
}
```

```typescript
export interface TarjetaContenidoProps {
  title: string;
  subtitle?: string;
  children: React.ReactNode;
  className?: string;
}
```

```typescript
export interface CampoEntradaNumericoProps {
  label: string;
  value: number;
  onChange: (value: number) => void;
  unit?: string;
  min?: number;
  max?: number;
  step?: number;
  error?: string;
  helperText?: string;
  disabled?: boolean;
}
```

```typescript
export type EstadoOperativo =
  | 'seguro'
  | 'advertencia'
  | 'critico'
  | 'neutral';

export interface IndicadorEstadoProps {
  label: string;
  status: EstadoOperativo;
  value?: string | number;
  helperText?: string;
}
```

```typescript
export interface ColumnaTabla {
  key: string;
  label: string;
  align?: 'left' | 'center' | 'right';
  format?: 'text' | 'number' | 'currency' | 'percentage' | 'date';
}

export interface TablaDatosProps {
  columns: ColumnaTabla[];
  rows: Record<string, string | number | null>[];
  emptyMessage?: string;
  caption?: string;
}
```

```typescript
export type TipoGrafico = 'linea' | 'area' | 'barras';

export interface SerieGrafico {
  key: string;
  label: string;
  colorToken: string;
}

export interface GraficoProps {
  title: string;
  data: Record<string, string | number>[];
  series: SerieGrafico[];
  xKey: string;
  kind?: TipoGrafico;
  height?: number;
  emptyMessage?: string;
}
```

`colorToken` must identify a key within `tokens.colors`; components must resolve it through the token object rather than embedding a hex value.

### Page behavior

#### `InterfazEntradaComposicion`

- Implements the complete hierarchy of frame `6:2`.
- Renders a 92-pixel header with brand, spacer, and live status.
- Renders the hero, elemental input panel, CEV and soldability results, “Calcular Optimizar” action, and footer note.
- Uses touch-friendly numeric controls suitable for a tablet.
- Shows an immediate warning when the composition exceeds `100%` and disables the optimization action in that state.
- Shows soldability through `IndicadorEstado`.
- Continues showing local CEV if the backend or price source is unavailable.

#### `SugerenciasOptimizacionCostes`

- Implements the complete hierarchy of frame `5:2`.
- Uses the 76-pixel navigation bar.
- Displays `OPTIMIZACIÓN DE COSTES / COLADA 24-081`, the title, explanatory subtitle, and real-time summary.
- Displays side-by-side “Mezcla Actual” and “Mezcla Optimizada” cards.
- Displays CEV, soldability, material percentages, current cost, optimized cost, and saving.
- Includes the validation note, “Aplicar mezcla” CTA, and secondary action.
- Displays system status and compliance control in the footer.
- The “Aplicar mezcla” action calls `aplicar`; it does not mutate local state as if persistence had already succeeded.

#### `VistasAnaliticas`

- Implements the complete hierarchy of frame `7:2`.
- Uses the 72-pixel navigation bar.
- Provides date and heat filters.
- Displays aggregate KPI cards.
- Displays CEV and cost trend charts.
- Displays historical data in `TablaDatos`.
- Includes the footer content.
- Filters use the exact `fecha_inicio`, `fecha_fin`, and `colada` names from `useAnalytics`.

#### `DesignSystemOverview`

- Implements the 1200 × 1481 frame `3:2`.
- Preserves section order: Header, Color Palette, Typography Scale, Foundations, Component Reference, Product Notes.
- Displays the literal “CEV seguro < 0.45” label.

#### `ComponentsLibrary`

- Implements the 1200 × 903 frame `4:2`.
- Displays the title `AI Factory · Component Library`.
- Displays the literal description `Reusable base components for material intelligence and CEV weldability workflows.`
- Displays all seven base components in Component Row 1, Component Row 2, and Component Row 3 according to the Figma hierarchy.

---

## 8. FILE EXTENSION CONVENTION

- The frontend is a **TypeScript project**.
- All React components, hooks, API modules, and type definitions use `.ts` or `.tsx`.
- All component and page files use `.tsx`.
- No `.jsx` files are permitted.
- The application entry module is `frontend/src/main.tsx`.
- The HTML entry point is `frontend/index.html`.
- Vite mounts the application from `/src/main.tsx`; `index.html` contains `<script type="module" src="/src/main.tsx"></script>`.

---

## 9. DESIGN TOKENS

The UI/UX contract contains an empty `design_tokens` object and does not provide a standalone typography scale, spacing scale, radius scale, or shadow scale. To avoid inventing values, `frontend/src/styles/tokens.ts` must contain only literal colors, font information, and frame dimensions present in the Figma contract.

```typescript
export const tokens = {
  colors: {
    navy122940: '#122940',
    navy123248: '#123248',
    navy0B3954: '#0B3954',
    navy1C3352: '#1C3352',
    backgroundF2F7FA: '#F2F7FA',
    backgroundF4F7FA: '#F4F7FA',
    backgroundF5F7FA: '#F5F7FA',
    surface: '#FFFFFF',
    muted616E80: '#616E80',
    black: '#000000',
  },
  typography: {
    fontFamily: 'Inter',
  },
  layout: {
    inputFrameWidth: '1200px',
    inputFrameHeight: '900px',
    inputHeaderHeight: '92px',
    inputContentHeight: '808px',
    optimizationFrameWidth: '1440px',
    optimizationFrameHeight: '900px',
    optimizationHeaderHeight: '76px',
    optimizationCabeceraHeight: '156px',
    optimizationComparisonHeight: '448px',
    optimizationActionsHeight: '112px',
    optimizationFooterHeight: '108px',
    analyticsFrameWidth: '1440px',
    analyticsFrameHeight: '1100px',
    analyticsHeaderHeight: '72px',
    analyticsContentHeight: '1028px',
    designSystemFrameWidth: '1200px',
    designSystemFrameHeight: '1481px',
    componentsFrameWidth: '1200px',
    componentsFrameHeight: '903px',
  },
} as const;
```

All React components must import `tokens` from `frontend/src/styles/tokens.ts`. No component may hardcode a hexadecimal color. Status-specific classes may map the available literal colors to `seguro`, `advertencia`, `critico`, and `neutral`; they may not introduce new color values.

---

## 10. FUNCTIONAL REQUIREMENTS COVERAGE

| Requirement | Implementation | Files |
|---|---|---|
| Web application for real-time CEV calculation and cost optimization | React SPA with three operational routes plus the two Figma reference frames; FastAPI REST backend | `frontend/src/App.tsx`, `frontend/src/pages/*.tsx`, `backend/app/main.py` |
| Audience includes Operario de Horno and Ingeniero Metalúrgico | Role label in navigation; touch-oriented composition interface; analytical interface for engineering review | `frontend/src/components/ui/NavegacionPrincipal.tsx`, `frontend/src/pages/InterfazEntradaComposicion.tsx`, `frontend/src/pages/VistasAnaliticas.tsx` |
| CEV recalculated instantly when values change | Synchronous TypeScript calculation in `useCEVCalculator`; every valid update immediately updates CEV, soldability, validity, and safety | `frontend/src/hooks/useCEVCalculator.ts`, `frontend/src/pages/InterfazEntradaComposicion.tsx` |
| Canonical CEV formula | Identical Python and TypeScript implementation of `C + Mn/6 + (Cr + Mo + V)/5 + (Ni + Cu)/15` | `backend/app/cev.py`, `frontend/src/hooks/useCEVCalculator.ts` |
| 100% CEV calculation accuracy against the approved reference | Single documented formula, shared field semantics, four-decimal presentation rounding, and exhaustive QA validation against reference values | `backend/app/cev.py`, `frontend/src/hooks/useCEVCalculator.ts` |
| Less than three-second calculation time | Local calculation does not await network requests; backend uses Pandas only for price data and avoids per-call CSV reads | `frontend/src/hooks/useCEVCalculator.ts`, `backend/app/cev.py`, `backend/app/pricing.py` |
| Visual soldability code | `IndicadorEstado` maps `Bajo`, `Medio`, and `Alto` to token-based semantic classes | `frontend/src/components/ui/IndicadorEstado.tsx`, `frontend/src/hooks/useCEVCalculator.ts` |
| Warn or block when material percentages exceed 100% | `sumaPorcentajes`, `superaCien`, and `esValida`; warning is displayed and optimization is disabled | `backend/app/cev.py`, `backend/app/schemas.py`, `frontend/src/hooks/useCEVCalculator.ts`, `frontend/src/pages/InterfazEntradaComposicion.tsx` |
| Optimized mixture must not raise CEV above the safe limit | Optimizer enforces strict `CEV < 0.45` and `cumple_cev_seguro`; invalid candidates are rejected | `backend/app/optimizer.py`, `backend/app/schemas.py` |
| Maximize profit per tonne by reducing expensive raw-material use | Cost minimization is the primary optimizer objective; response includes current cost, optimized cost, absolute saving, and weighted percentage saving | `backend/app/optimizer.py`, `frontend/src/pages/SugerenciasOptimizacionCostes.tsx` |
| At least 3% average cost reduction target | Optimizer prioritizes lower cost under the CEV and composition constraints; analytics exposes average percentage saving | `backend/app/optimizer.py`, `backend/app/analytics.py`, `frontend/src/pages/VistasAnaliticas.tsx` |
| CEV remains usable when the price source is unavailable | CEV endpoint does not access prices; frontend computes locally; optimization returns honest HTTP `503` rather than fabricated costs | `backend/app/routers/calculos.py`, `backend/app/optimizer.py`, `frontend/src/hooks/useCEVCalculator.ts` |
| Ingest prices from `precios_mercado.csv` | Startup CSV ingestion and manual CSV import endpoint | `backend/app/pricing.py`, `backend/app/routers/materiales.py`, `backend/data/precios_mercado.csv` |
| Internal SQL database for manual price updates | SQLite persistence for `MATERIA_PRIMA` and `HISTORIAL_PRECIOS`, with manual create/update endpoints | `backend/app/models.py`, `backend/app/routers/materiales.py`, `backend/app/persistence.py` |
| Current prices and historical price records | Update of `precio_actual` always appends a history row; history endpoint returns descending records | `backend/app/pricing.py`, `backend/app/persistence.py`, `backend/app/routers/materiales.py` |
| Manual price update by the metallurgical engineer | Editable material price UI on the analytical/engineering workflow, calling the update endpoint | `frontend/src/api/materiales.ts`, `frontend/src/pages/VistasAnaliticas.tsx` |
| Synchronous REST communication between frontend and backend | Axios REST client with typed request and response modules | `frontend/src/api/*.ts`, `frontend/src/api/client.ts`, `backend/app/routers/*.py` |
| Thousands of local calculations and 50–100 backend calls daily | Local-first calculation, efficient SQLite price lookup, no price access for CEV calculation | `frontend/src/hooks/useCEVCalculator.ts`, `backend/app/pricing.py` |
| Record and audit alloy calculations | `ALEACION` stores elemental percentages, CEV, soldability, and estimated cost; composition rows store material percentages | `backend/app/models.py`, `backend/app/routers/aleaciones.py` |
| Analytics by date and heat | `fecha_inicio`, `fecha_fin`, and `colada` filters; heat identifier maps to `ALEACION.nombre` | `backend/app/analytics.py`, `backend/app/routers/analiticas.py`, `frontend/src/hooks/useAnalytics.ts` |
| Trends and historical data | Analytics response includes summary, trend points, and historical rows; charts and table render them | `frontend/src/pages/VistasAnaliticas.tsx`, `frontend/src/components/ui/Grafico.tsx`, `frontend/src/components/ui/TablaDatos.tsx` |
| Adoption KPI of at least 80% of heats recorded | Persisted `ALEACION` records provide the denominator for registered heats; analytics exposes `coladas_registradas` | `backend/app/routers/aleaciones.py`, `backend/app/analytics.py` |
| CSAT target of at least 4/5 | Product usability and immediate feedback support the target; CSAT is a business KPI and not represented by an invented database entity | `frontend/src/pages/InterfazEntradaComposicion.tsx`, `frontend/src/pages/SugerenciasOptimizacionCostes.tsx` |
| At least 99% uptime | Health endpoint, rolling-update deployment workflow, redundant backend instances in Compose, and no local state dependency for CEV | `backend/app/main.py`, `docker-compose.yml`, `.github/workflows/ci-cd.yml` |
| Security through input validation and REST API protection | Pydantic validation, bounded query parameters, restrictive CORS, request-size validation, conflict responses, no secret logging | `backend/app/schemas.py`, `backend/app/config.py`, `backend/app/main.py` |
| Input validation | Numeric ranges, allowed material type, composition total validation, strict CEV limit, malformed CSV handling | `backend/app/schemas.py`, `backend/app/cev.py`, `backend/app/pricing.py` |
| Exactly 100% CEV match to reference | Formula, rounding, classification boundaries, and frontend/backend parity are fixed in this specification | `backend/app/cev.py`, `frontend/src/hooks/useCEVCalculator.ts` |
| Optimize without compromising alloy quality | Target elemental profile is a hard constraint; candidate must preserve target composition and remain below safe CEV | `backend/app/optimizer.py` |
| Existing optimization engine integration | `optimizar_mezcla` is the stable integration boundary for the existing engine; the implementation uses a deterministic constrained optimizer and contains no mock engine or fabricated result | `backend/app/optimizer.py` |
| External AI support constraint | No unavailable AI service is required at runtime; the deterministic optimizer is production code. An external AI optimizer would require a separately approved integration and contract | `backend/app/optimizer.py` |
| Apply optimized mixture | Save the optimized alloy and composition transactionally through the alloy endpoint | `frontend/src/hooks/useOptimization.ts`, `backend/app/routers/aleaciones.py`, `backend/app/persistence.py` |
| Figma fidelity | Exact frame order, dimensions, section hierarchy, literal colors, and named base components are implemented in page and component files | `frontend/src/pages/*.tsx`, `frontend/src/components/ui/*.tsx`, `frontend/src/styles/tokens.ts` |
| Interfaz de Entrada de Composición page | Full page implementation of node `6:2` with header, hero, input panel, results, action, and footer | `frontend/src/pages/InterfazEntradaComposicion.tsx` |
| Sugerencias de Optimización de Costes page | Full page implementation of node `5:2` with navigation, header, comparison, actions, and footer | `frontend/src/pages/SugerenciasOptimizacionCostes.tsx` |
| Vistas Analíticas page | Full page implementation of node `7:2` with navigation, filters, KPIs, charts, history table, and footer | `frontend/src/pages/VistasAnaliticas.tsx` |
| Design System Overview frame | Full implementation of node `3:2` and all six section regions | `frontend/src/pages/DesignSystemOverview.tsx` |
| Components Library frame | Full implementation of node `4:2` and all three component rows | `frontend/src/pages/ComponentsLibrary.tsx` |
| Seven mandatory base components | Seven individual named component modules under `components/ui` | `frontend/src/components/ui/*.tsx` |
| Responsive tablet use | CSS layouts preserve the Figma desktop dimensions while stacking controls and comparison cards on tablet widths without removing content | `frontend/src/styles/global.css`, `frontend/src/pages/*.tsx` |
| Docker portability | Backend and frontend Dockerfiles plus Compose deployment | `backend/Dockerfile`, `frontend/Dockerfile`, `docker-compose.yml` |
| CI/CD for development, staging, and production | GitHub Actions builds images, validates source and frontend compilation, publishes images, and performs rolling Compose deployment through a self-hosted deployment runner | `.github/workflows/ci-cd.yml` |
| Daily SQLite backups | Backup script uses SQLite’s online backup mechanism and writes one backup per day without exposing a database port | `backend/scripts/backup_sqlite.py` |
| Redundant backend instances | Compose defines two backend replicas behind the configured reverse proxy/load-balancer target; no cloud provider is invented | `docker-compose.yml` |
| Documentation for price updates | User-facing instructions explain CSV columns, manual updates, validation errors, and fallback behavior | `docs/ACTUALIZACION_PRECIOS.md`, `README.md` |
| No external integrations in the MVP | Only CSV, SQLite, REST, and Docker are included; no email, ERP, messaging, cloud-provider, or AI endpoint is fabricated | Entire approved file tree |
| 15-day MVP and 5000 € project constraint | Scope is restricted to one backend, one frontend, local CEV, constrained optimization, prices, records, and analytics; no unrelated platform layers are included | Entire approved file tree |