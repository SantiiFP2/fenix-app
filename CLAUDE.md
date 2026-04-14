# CLAUDE.md — Fénix Solutions

## Regla #1 — NUNCA modificar lógica de cálculo existente
Todos los cambios deben ser estrictamente aditivos. `index.html` es la referencia sagrada. No tocar funciones de cálculo que ya funcionan.

## Stack
- `index.html` ~12,000+ líneas (single-file vanilla HTML/JS, Inter Tight font, dark theme)
- Colores: `--negro:#0A0A0A` / `--rojo:#DA0000`
- `server.js` v3.0.2, Node.js/Express
- PostgreSQL en DigitalOcean App Platform
- GitHub → DigitalOcean automatic deployment
- Bearer token auth (8h sessions), roles: admin / supervisor / usuario
- Frontend: `https://santiifp2.github.io/fenix-app/`
- Backend: `https://fenix-solutions-g6edy.ondigitalocean.app/`

## CDN Libraries
- SheetJS (XLSX) 0.18.5
- Chart.js 4.4.1
- Google Fonts (Inter Tight)

## PostgreSQL Tables
`usuarios`, `sesiones`, `metas`, `indicadores_custom`, `cosechas_meta`, `cosechas_r5_datos`, `cosechas_factura_datos`, `cvcs`, `proyecciones_guardadas`

## Módulos activos
1. **Proyección Móvil** — filtrado por SITE (BOGOTA 2, MEDELLIN, GENERAL)
2. **CVCs** — persiste en DB vía tabla `cvcs`; endpoints GET/POST/DELETE `/cvcs`
3. **Metas**
4. **Cosechas** — tabs: Postpago, Power, Acelerador, Análisis, Proyección, Por Sede
5. **Panel de Inicio** — facturas con pill selectors, cards financieras/operativas, análisis histórico

## Contexto de negocio
- Facturación opera con mes diferido (ventas de marzo se pagan en abril)
- Ciclo 1 = primera mitad del mes, Ciclo 2 = segunda mitad
- R5 files se cargan al cierre del mes; Factura llega el último día del mes siguiente
- Comisiones: P1/P2/P3 para Postpago y Migración, P1-P4 para Power, tres pagos iguales para Acelerador

## Fórmulas de Cosechas (exactas)
- **Mes 1:** ParteA = `((P1 × tasa_P2%) − P2) × ARPU_P2`; ParteB = `((P2 × tasa_P3%) − P3) × ARPU_P3`
- **Mes 2/3/4:** `(P1 × tasa% − P3) × ARPU_P3`
- Reducción global: `Total × (1 − caída_global%)`
- **ARPUs Postpago:** P1=$49,989 / P2=$33,984 / P3=$15,977
- **Power:** 4 pagos con ARPUs propios
- **Acelerador:** valor variable por mes de origen

## Variables críticas de estado
| Variable | Rol |
|---|---|
| `_resultadosCacheGlobal` | Datos sin filtrar, solo se setea en `renderResultados` |
| `_sedeFiltroActual` | Filtro de sede activo |
| `_ultimosDatosKpi` | Últimos datos filtrados por sede |
| `_bonoCalculado[sede\|'general']` | Valores de bono por sede |

## Reglas de desarrollo (NO violar)
- **No deduplicar por `co_id+mes_liq`** — rompe registros legítimos de multi-pago
- **`SITE` y `CODIGO CVC` son dimensiones independientes** — un agente en Bogotá puede vender bajo CVC de Medellín
- **Bono/acelerador se calculan por línea** según cumplimiento real del CVC
- **`_resultadosCacheGlobal`** es el almacén permanente sin filtrar — `_global` para evaluación de cumplimiento, `_dk` para cálculo proporcional de pago
- **`_bonoCalculado`** — `actualizarKpiTotal` debe leer de aquí, no recalcular
- **`renderBonoCumplimiento`** debe ejecutarse ANTES de `actualizarKpiTotal`
- **`mesCerrado(mesVenta, anio)`** suprime toda proyección automáticamente cuando el mes ya pasó
- **Chunked uploads** (500 filas/request) y **paginated downloads** (`?limit=5000&offset=0`) para evitar 504
- **SheetJS:** usar `raw: true` con conversión manual de fecha serial Excel `(mesRaw - 25569) * 86400 * 1000`
- **Formato monetario:** Colombian peso (es-CO locale) en toda la UI

## Flujo de trabajo
Santiago describe cambios en español → Claude Code edita `index.html` → Santiago hace push a GitHub → auto-deploy a DigitalOcean

## Filosofía
- Análisis profundo del código ANTES de implementar
- Cambios aditivos únicamente
- Nunca tocar lógica de cálculo que funciona
- Santiago comparte contexto de forma incremental — esperar toda la información antes de implementar
