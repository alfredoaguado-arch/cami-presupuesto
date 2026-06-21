# CLAUDE.md — cami-presupuesto

## 1. Contexto general del ecosistema CAMI

Este repo (`cami-presupuesto`) es **uno de varios** que conforman el sistema CAMI de Aceros Manufacturados. Antes de tocar este repo, conviene saber dónde encaja.

CAMI es una plataforma modular para operación interna (construcción / manufactura). Cada módulo es una app web separada, mobile-first, en su propio repo de GitHub bajo `alfredoaguado-arch/`. Todos comparten un backend de autenticación común y se montan vía GitHub Pages.

**Repos del ecosistema:**

| Repo | Propósito |
|---|---|
| `cami-app2` | Hub de login + lanzador de módulos |
| `cami-ot` | Órdenes de trabajo |
| `cami-almacen` | Almacén |
| `cami-presupuesto` | **Este repo.** Cotizaciones / presupuestos |
| `cami-requisicion` | Requisiciones de pago |
| `cami-nomina` | Nómina quincenal |
| `cami-reportes` | Reportes fotográficos |

**Stack global:**
- Frontend: HTML/CSS/JS puro, sin frameworks ni build. Un solo `index.html` por módulo.
- Backend: Google Apps Script. Cada módulo tiene su propio script *bound* a un Google Sheet. Existe además un **Apps Script central** para auth.
- Datos: Google Sheets.
- Documentos: Google Drive (PDFs).
- Hosting: GitHub Pages hoy; migración planeada a Hostinger (`aceroscami.com`).

## 2. Qué es este módulo

`cami-presupuesto` es el módulo de **cotizaciones / presupuestos**. Genera cotizaciones formales con folio, calcula totales, exporta a PDF y guarda en Drive.

**Versión actual:** v1.2 (desde 8-may-2026, agregó duración numérica, vigencia, tiempo de entrega, condiciones).

## 3. Patrón de PDF (compartido con todo el ecosistema)

Este módulo usa el patrón estándar de PDF de CAMI:

**Convención de folio:** `COT-CLIENTE-NNN-YYYY-MM-DD`
- `COT` = identificador del módulo
- `CLIENTE` = clave corta del cliente
- `NNN` = consecutivo de 3 dígitos
- Fecha en ISO

**Metadata embebida** en el campo `subject` del PDF:
```
CAMI_PRESUPUESTO_DATA::<json>
```

El PDF es **re-leíble**: el módulo puede recargar este PDF desde Drive y leer la metadata para reconstituir los datos sin recapturar (útil para editar o duplicar cotizaciones).

**Mecanismos de carga:**
- Selector desde Drive (endpoints `listar` / `descargar`)
- Fallback: botón "Cargar local" con PDF.js

## 4. Patrón de autenticación

Como todos los módulos del ecosistema, este módulo:

1. Lee `sessionStorage.cami_session` (JSON con `{token, nombre, rol, apps, proyectos}`)
2. Manda `token` en cada request a su propio Apps Script
3. El Apps Script de presupuesto valida el token vía HTTP contra el central antes de procesar
4. Si el token expiró (4h), redirige a login

**App key requerida:** `presupuesto`

**Importante:** Existe legacy del endpoint `handleCotizacion` en el Apps Script central que sigue funcionando como hotfix con validación de token relajada. Esto está marcado como pendiente de limpieza pre-migración a Hostinger.

## 5. Convenciones técnicas

**Estilo visual:**
- Tipografía: Courier New (monoespaciada)
- Mobile-first, sin frameworks
- Diseño coherente con cami-app2

**JavaScript:**
- Sin frameworks. Vanilla JS.
- `fetch` directo al Apps Script con `Content-Type: text/plain;charset=utf-8` (para evitar preflight CORS)
- Cuerpo siempre `JSON.stringify({action, token, ...payload})`

**HTML:**
- Un solo archivo. CSS embebido en `<style>`, JS embebido en `<script>`.

## 6. Vínculo con otros módulos

**Con `cami-procesos` (futuro):**
Cuando exista cami-procesos, será posible importar la lista de conceptos de una cotización para que cada concepto se vuelva un item del proyecto. Esto requiere coordinar formato de exportación.

## 7. Reglas de modificación

**SÍ tocar este repo cuando:**
- Cambios al flujo de captura de cotización
- Ajustes al PDF (layout, campos, metadata)
- Nuevos campos en cotización (vigencia, condiciones, etc.)
- Bugs en cálculos de totales

**NO tocar este repo cuando:**
- Cambios al patrón de auth global (eso es cami-app2 + central)
- Cambios al patrón de folio o metadata (eso afecta a todos los módulos)

**Antes de cualquier cambio:**
- Confirmar el plan conmigo (Alfredo) antes de generar código
- Si el cambio toca el formato del PDF, verificar que la metadata sigue siendo re-leíble

## 8. Despliegue

**GitHub Pages:**
- Push a `main` despliega automáticamente
- URL: `https://alfredoaguado-arch.github.io/cami-presupuesto/`
- Tarda 1-2 minutos en propagar

**Backend (Apps Script):**
- Editar en el editor de Apps Script bound al sheet del módulo
- Deploy → Manage deployments → ✏️ → New version → Deploy

## 9. Marca dual CAMI / CAMI-PRIMA (pendiente)

PRIMA es un socio comercial cuyos proyectos también se fabrican en taller CAMI. Está pendiente implementar marca dual en los PDFs (encabezado CAMI vs CAMI/PRIMA según el proyecto). Esto se trabajará después de la migración a Hostinger.

## 10. Migración planeada

A futuro, este módulo se mueve a Hostinger bajo `aceroscami.com/presupuesto/` para mantener `sessionStorage` compartido con los demás módulos en subcarpetas.

## 11. Patrón de colaboración con Claude

- Alfredo confirma plan antes de codear cualquier cambio
- Los cambios se prueban primero en local antes de hacer commit
- Siempre commit con mensaje descriptivo (`vX.Y — qué cambia`)
- Después de commit, recarga forzada en la app para validar
