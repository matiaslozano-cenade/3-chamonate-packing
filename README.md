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

## Desarrollo

Abrir `index.html` directamente en el navegador (lee Supabase con la anon key embebida).
</content>
