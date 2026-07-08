# 3-chamonate-packing

Dashboard de rentabilidad de **Packing** de Chamonate. Ingresos vs costos contables desde el Libro Mayor, de lo más general a lo más específico, con **Estado de Resultado (EERR) por cuenta** en cada nivel.

## Stack

- HTML estático + Tailwind (CDN) + Chart.js + chartjs-plugin-datalabels + Supabase JS
- Sin build · deploy en Vercel
- Proyecto Vercel: `3-chamonate-kpi-packing`
- URL producción: `https://3-chamonate-kpi-packing.vercel.app`

## Visualizaciones

Mismo sistema visual que los dashboards de **Campo** y **Maquinaria**: gráficos en contenedores de altura fija (`chart-box`), **etiquetas de datos siempre visibles** sobre barras/puntos, combo **barras (ingresos) + línea (costos)**, gráfico **"Costos por cuenta"** (barras por cuenta contable), KPIs con borde de color y el **EERR al lado del gráfico**. Acento naranja (`#c2410c`).

## Datos (Supabase `wjbwccacjdkuejcriode`)

| Vista / Tabla | Uso |
| --- | --- |
| `v_3_pack_cc` | Totales por centro de costo · año + mes |
| `v_3_pack_grupo` | Totales por línea (grupo) |
| `v_3_pack_cuenta` | Ingresos/costos **por cuenta contable** — alimenta el EERR (cuadra exacto con `v_3_pack_cc`) |
| `v_3_pack_detalle` | Detalle línea a línea del Libro Mayor — alimenta drill-down y descarga CSV |
| `3_chamonate_centros_costo` (`linea_negocio='packing'`) | Catálogo de centros + línea |
| `3_chamonate_libro_mayor` | Movimientos contables (drill-down) |

> Todas las cargas (`v_3_pack_cc`, `v_3_pack_cuenta`) están **paginadas** (>1000 filas) para no truncar datos.

## Pestañas

General · Mensual · **Por Línea** · Por Centro.

- **General / Mensual**: KPIs, comparativo anual/mensual, combo ingresos+costos, margen %, **EERR por cuenta** y **"Costos por cuenta"**.
- **Por Línea**: tarjetas por línea de packing con drill **Línea → Centro de costo**; cada vista termina con EERR + "Costos por cuenta" del scope activo.
- **Por Centro**: histórico anual, mensual del año, EERR del centro, **ver movimientos del Libro Mayor** y descarga CSV.

En el **Estado de Resultado por cuenta** (en todas las pestañas) cada monto es **clickeable para descargar su detalle**: el de **INGRESOS**, el de **cada cuenta de costo** y el de **TOTAL COSTOS**. La descarga respeta el scope activo (temporada, mes, línea, centro) y filtra `v_3_pack_detalle` por `n_cuenta`.

## Temporada (octubre → septiembre) — particularidad clave

Packing **no** se analiza por año calendario (ene–dic) sino por **temporada de 12 meses, del 1 de octubre al 30 de septiembre** (para no partir la campaña de cerezas nov–ene entre dos años). La temporada T va de **octubre del año T a septiembre del año T+1** y se etiqueta `Temp. YY-YY` (ej: `Temp. 24-25` = oct 2024 → sep 2025).

- Asignación: `temporada = (mes >= 10) ? año : año - 1` (octubre inicia la nueva temporada).
- Todo el eje temporal (pills, comparativo, mensual, EERR, histórico, drill-down y CSV) razona en temporadas; los meses se ordenan **oct → sep**.
- El drill-down al Libro Mayor y la descarga CSV traducen la temporada a los dos años contables reales que cruza. Mismo enfoque que Campo (que usa jun→may).

## Líneas (grupo) de packing

`Cerezas`, `Cítricos`, `Manzanas`, `Carozos · Uva · Frigorífico`, `General Packing`, `Maquinaria Packing`, `Inversiones / Construcción`.

## Reglas de negocio (contables)

- **Ingresos:** cuentas `4%` → `sum(haber − debe)`.
- **Costos:** cuentas `3%` con `debe > 0` → `sum(debe)`.
- El EERR (`v_3_pack_cuenta`) usa exactamente la misma lógica que `v_3_pack_cc`, por lo que los totales cuadran al peso.

### Reclasificación de cierre de manzanas (temporada corrida)

La contadora contabiliza en **diciembre** un asiento de reclasificación (`TRASPASO`, glosa `DISTRIBUCION INGRESOS MANZANAS TEMP X`, cuentas `4102%` = servicios packing/frío/fruta terceros) que en realidad corresponde a la **temporada anterior** (la fruta se procesó en la campaña previa). Ej.: el asiento `2025121561` del 31/12/2025 ($584,7M) es de la campaña de manzanas de la **Temp 24-25**, pero al estar fechado en dic-2025 caería en Temp 25-26.

Regla (angosta) aplicada en las **vistas** de Packing: si `mes_contable = 12` **y** `cuenta LIKE '4102%'` **y** `glosa_enc ILIKE 'DISTRIBUCION INGRESOS MANZANAS%'` → el **año de temporada = `ano_contable − 1`**. Se expone como columnas `ano_temp` / `mes_temp` en `v_3_pack_detalle`, y las vistas de agregación ya emiten el `ano` corrido.

- El **Libro Mayor no se toca**: `fecha` y `ano_contable` reales siguen mostrando dic-2025 (concilia con contabilidad). Solo cambia la **asignación de temporada**.
- KPIs, gráficos, **drill-down y descarga CSV** razonan sobre `ano_temp` / `mes_temp`, por lo que pantalla y CSV quedan consistentes: al descargar **Temp 24-25** aparece la línea reclasificada (con su fecha real 31/12/2025); en Temp 25-26 ya no.
- Es el **único** asiento de este tipo en la base (no existe equivalente en Temp 22-23 ni 23-24). Cada temporada queda "abierta" en su reclasificación de cierre hasta el diciembre siguiente.

### Reclasificación de cuentas químicas (solo visualización/descarga)

`FERTILIZANTES AL SUELO` (`3101000025`) y `PESTICIDAS` (`3101000027`) se muestran dentro de **`PRODUCTOS QUIMICOS`** (`3101000014`). Se unifica también la variante `3101000014.0` (mismo nombre, artefacto de importación) para que el EERR muestre **una sola línea** de Productos Químicos. Aplicado en las vistas `v_3_pack_cuenta` (agrupa) y `v_3_pack_detalle` (relabela `cuenta`/`n_cuenta` en drill-down y CSV). Es solo relabelado de costo: **no cambia el total de costos**, solo consolida la presentación. LM intacto.

### Exclusión de Venta de Activo Fijo (leasing)

`VENTA ACTIVO FIJO` (`4201000005`) se **excluye** de Packing en las 4 vistas (`WHERE cuenta NOT LIKE '4201000005%'`): no es ingreso operativo sino un leasing. Impacto: −$1,2M en Temp 24-25 y −$526,75M en Temp 25-26 (5 líneas de $105,35M en feb-2026, CC 860 Packing Cerezas). Desaparece de KPIs, EERR y descargas. LM intacto.

## Desarrollo

Abrir `index.html` directamente en el navegador (lee Supabase con la anon key embebida).
</content>

## Dominio

URL oficial: **https://kpi-packing.chamonate.portalcenade.cl**. Cualquier acceso vía `*.vercel.app` redirige (308) al dominio oficial — configurado en `vercel.json` (`redirects` con condición `host`).
