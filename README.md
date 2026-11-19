# Atlas vial · Ciudad de México

Mapa, series y desgloses de los **hechos de tránsito de 2024** registrados por la
Secretaría de Seguridad Ciudadana de la Ciudad de México, con datos abiertos,
filtros conectados y la cobertura de cada desglose siempre a la vista.

[![Build](https://github.com/Dev-HugoAguilar/cdmx-traffic-atlas/actions/workflows/tests.yml/badge.svg)](https://github.com/Dev-HugoAguilar/cdmx-traffic-atlas/actions/workflows/tests.yml)
[![React](https://img.shields.io/badge/React-TypeScript-3178C6?logo=react&logoColor=white)](app)
[![Datos](https://img.shields.io/badge/datos-SSC%20CDMX%20%C2%B7%20CC--BY--4.0-b7791f)](#datos-y-atribución)

> **Alcance.** Aplicación de visualización; no sustituye los datos abiertos de la
> SSC. Los archivos de personas y vehículos traen **un registro por folio** —la
> persona y el vehículo principales del expediente—, no todas las víctimas ni
> todos los vehículos.

---

## Ejecutar

Aplicación React + Vite sin backend propio: consume la API compartida
[`portfolio-api`](https://github.com/Dev-HugoAguilar/portfolio-api) (FastAPI + MongoDB)
en el puerto **8010**. La API no se copia en este repositorio — se clona aparte —
para no duplicar código ni datos.

### 1. Requisitos

- **Python 3.12** para la API.
- **Node.js 20 o superior** con `npm` (verificado con Node 24.11.0 y npm 11.6.1).
- **MongoDB**, Atlas o local: solo hace falta la URI de conexión.

### 2. API en el puerto 8010

```bash
git clone https://github.com/Dev-HugoAguilar/portfolio-api.git
cd portfolio-api
python -m venv .venv
```

Activa el entorno virtual:

```bash
# Windows (PowerShell o CMD)
.venv\Scripts\activate
# Linux / macOS
source .venv/bin/activate
```

Instala y arranca:

```bash
pip install -r requirements-dev.txt
cp .env.example .env        # Windows PowerShell: Copy-Item .env.example .env
python -m uvicorn app.main:app --port 8010
```

Variables que hay que rellenar en `.env` (no se versiona):

| Variable | Qué poner |
|---|---|
| `MONGODB_URI` | **Obligatoria.** URI de conexión de MongoDB: cadena de Atlas (`mongodb+srv://…`, con el usuario y la clave del clúster) o local (`mongodb://127.0.0.1:27017/hugo_portfolio`). |
| `MONGODB_DB` | Base donde están las colecciones publicadas (`hugo_portfolio`). |
| `ALLOWED_ORIGINS` | Orígenes permitidos por CORS; ver el paso 3. |
| `MONGO_ROOT_*`, `APP_DB_*`, `BACKEND_HOST_PORT` | Solo si levantas MongoDB/API con Docker Compose. |
| `MARKETING_GCP_PROJECT` | Solo para volver a extraer datos de GA4 desde BigQuery; la API y el tablero leen de MongoDB. |

`/health/ready` comprueba proceso **y** MongoDB:

```bash
curl http://127.0.0.1:8010/api/v1/health/ready
# {"status":"ready","mongo":"ok","version":"1.0.0"}
```

### 3. Aplicación en el puerto 5173

```bash
cd app
npm ci
npm run dev
```

Vite sirve en <http://127.0.0.1:5173>. El cliente consulta la API en
`http://127.0.0.1:8010/api/v1/atlas-vial` (ver `app/src/api/cliente.ts`), así que
la petición es **de otro origen** (5173 → 8010).

**CORS.** Si la API no autoriza el origen del dev server, el navegador bloquea la
respuesta: error de CORS en la consola y «La API no responde» en la interfaz. Añade
los dos orígenes de desarrollo a `ALLOWED_ORIGINS` en el `.env` de la API y
reinicia uvicorn:

```env
ALLOWED_ORIGINS=http://localhost:8000,http://127.0.0.1:8000,http://127.0.0.1:5173,http://localhost:5173
```

La lista va separada por comas, sin espacios.

### 4. Compilación de producción

```bash
npm run build
```

Ejecuta `tsc --noEmit`, `vite build` y el minificador de `app/tools/`. La salida
queda en **`app/dist/`** (`app.js` y `app.css`), el módulo que el portafolio carga
con `import()`.

### 5. Comprobación funcional

Endpoint real de este tablero (definido en `app/api/v1/routes/atlas.py` de la API):

```bash
curl http://127.0.0.1:8010/api/v1/atlas-vial/estado
```

Debe responder `HTTP 200` con la versión publicada y su cobertura. Obtenido al
verificar este repositorio: `"publicada": true` y
`"version": "ssc-2024-68a5871469"`, junto con `periodo`, `cifras`, `fuente`,
`recursos` y `validacion`. Si `publicada` es `false`, la base no tiene datos
importados: cárgalos con `python -m scripts.importar_atlas_vial` en el repo de la
API (la API no inventa cifras).

Con la API en marcha, abre <http://127.0.0.1:5173>: el mapa, la serie y la tabla
deben mostrar la misma versión que devolvió `/estado`.

> **Versión compatible:** probado con `portfolio-api` en el commit `8a27227`.

![Portada del Atlas vial](docs/cover.jpg)

## Qué demuestra

- **Un mapa que declara su cobertura**: hechos georreferenciados con el límite de
  alcaldías como referencia territorial, no como atribución del hecho.
- **Desgloses que no se suman entre sí**: sexo, edad y tipo de persona se
  presentan por separado, con su cobertura explícita, porque unirlos por folio
  inventaría combinaciones.
- **Datos faltantes visibles**: los hechos sin edad o sin sexo se muestran como
  «Sin dato», nunca como cero.
- **Filtros conectados**: mapa, series y tabla responden al mismo estado.

## Estructura

```text
app/                 Aplicación React + TypeScript (Vite)
  src/               Componentes, estado, consultas y visualizaciones D3
  tools/             Verificadores de navegador y minificador del bundle
  index.html         Entrada de la aplicación
  vite.config.ts     Build hacia app/dist (sin escribir fuera del repositorio)
docs/LEEME.md        Documentación original del proyecto
docs/cover.jpg       Captura real de la aplicación
```

## Datos y atribución

**SSC CDMX**, conjunto «Hechos de tránsito registrados por la SSC 2024 (serie de
datos ampliada, no comparativa)», catálogo de datos abiertos de la CDMX, licencia
**CC-BY-4.0-ESP**. Cobertura: 1 de enero a 31 de diciembre de 2024, cinco recursos
(hechos; personas por sexo, por edad y por tipo de persona; vehículos). Límites de
alcaldías: «Límite de Alcaldías», CC-BY-4.0-ESP, a partir del Marco Geoestadístico
2020 del INEGI.

## Límites

- Un registro por folio: no son todas las víctimas ni todos los vehículos del
  expediente; se presentan como «hechos con registro de…».
- Los desgloses de sexo, edad y tipo de persona no se suman ni se cruzan entre sí.
- 1 654 hechos no tienen edad registrada y 1 005 no tienen sexo: aparecen como
  «Sin dato».
- La entidad o alcaldía del registro no es necesariamente el lugar del hecho.

## Seguridad y contribución

Sin credenciales ni datos personales en el repositorio. Ver [`SECURITY.md`](SECURITY.md)
y [`CONTRIBUTING.md`](CONTRIBUTING.md).
