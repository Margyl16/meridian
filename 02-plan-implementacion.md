**PLAN DE IMPLEMENTACIÓN**

**Meridian**

Guía paso a paso con ejemplos reales

*Cada fase ilustrada con datos, código y outputs esperados*

Ruta 1: Priors informados con datos históricos

*Meta Ads + Google Ads · Windsor.ai · Python + Colab*

Abril de 2026

# Introducción

Este documento presenta el plan completo de implementación de Meridian paso a paso, con un ejemplo práctico ilustrando cada fase. El ejemplo sigue a un cliente ficticio llamado ModaViva a través de todas las fases del proyecto, mostrando datos reales, código funcional y outputs esperados en cada punto.

**Caso de ejemplo: ModaViva**

Para ilustrar cada paso del plan, trabajaremos con ModaViva, un cliente ficticio con características típicas de un proyecto Meridian real:

+-----------------------------------------------------------------------+
| **📌 Perfil del cliente ModaViva**                                    |
|                                                                       |
| -   Industria: e-commerce de ropa femenina                            |
|                                                                       |
| -   Mercado: Colombia, 5 ciudades principales                         |
|                                                                       |
| -   Inversión publicitaria: USD 60.000/mes (aprox. 240 millones COP)  |
|                                                                       |
| -   Histórico disponible: 18 meses (78 semanas)                       |
|                                                                       |
| -   Canales activos: Meta Ads y Google Ads                            |
|                                                                       |
| -   Objetivos de campaña: awareness y tráfico                         |
|                                                                       |
| -   KPI de negocio: conversiones (compras en el sitio web)            |
|                                                                       |
| -   Fuente de datos: Windsor.ai conectado a ambas plataformas         |
|                                                                       |
| -   Stack técnico: Python + Google Colab Pro+                         |
+-----------------------------------------------------------------------+

**Estructura del plan**

El plan se divide en 7 fases secuenciales que cubren desde la validación inicial hasta la operación continua:

-   **Fase 0:** Validación y preparación (1 semana)

-   **Fase 1:** Extracción y estructuración de datos (2-3 semanas)

-   **Fase 2:** Estimación empírica de parámetros (1 semana)

-   **Fase 3:** Modelado con Meridian (2 semanas)

-   **Fase 4:** Optimización de presupuesto (1 semana)

-   **Fase 5:** Aplicación gradual y validación (4-6 semanas)

-   **Fase 6:** Ciclo operativo continuo (ongoing)

Tiempo total estimado del setup inicial: 11-13 semanas. Después entra en operación con mantenimiento mensual/trimestral.

# Fase 0. Validación y preparación

Duración: 1 semana

Objetivo: confirmar que el caso es viable y dejar el entorno listo antes de invertir tiempo en implementación. Esta fase puede ahorrar meses de trabajo si detecta bloqueadores tempranos.

## Paso 0.1: Checklist de viabilidad

Se evalúan los criterios mínimos que el cliente debe cumplir:

  ---------------------------------------------------------------------------------------
  **Criterio**                                   **Crítico**   **Acción si no cumple**
  ---------------------------------------------- ------------- --------------------------
  ¿Hay 18+ meses de histórico en Windsor?        Sí            Esperar o usar tests A/B

  ¿Las campañas tienen KPI de negocio medible?   Sí            Reconsiderar Meridian

  ¿3+ regiones con datos consistentes?           Recomendado   Usar modelo nacional

  ¿Inversión mensual \>USD 30k?                  Sí            Evaluar costo-beneficio

  ¿Analista con Python disponible?               Sí            Contratar o partner
  ---------------------------------------------------------------------------------------

+-----------------------------------------------------------------------------------------------------------+
| **📌 Ejemplo aplicado a ModaViva**                                                                        |
|                                                                                                           |
| Evaluación del caso:                                                                                      |
|                                                                                                           |
| -   Histórico: 18 meses ✅ (límite inferior, pero viable)                                                 |
|                                                                                                           |
| -   KPI medible: conversiones trackeadas en GA4 + pixel Meta ✅                                           |
|                                                                                                           |
| -   Regiones: 5 ciudades con datos consistentes ✅                                                        |
|                                                                                                           |
| -   Inversión: USD 60k/mes ✅ (por encima del mínimo)                                                     |
|                                                                                                           |
| -   Recurso técnico: data scientist interno disponible ✅                                                 |
|                                                                                                           |
| Veredicto: PROYECTO VIABLE con ajustes por histórico corto (priors más informados, max_lag reducido a 6). |
+-----------------------------------------------------------------------------------------------------------+

## Paso 0.2: Definición del KPI final

Se determina qué variable respuesta se va a modelar. Para campañas de awareness y tráfico, se confirma que hay una conversión posterior medible que es el verdadero KPI de negocio.

+-------------------------------------------------------------------------------------------------+
| **📌 Ejemplo aplicado a ModaViva**                                                              |
|                                                                                                 |
| Opciones evaluadas:                                                                             |
|                                                                                                 |
| -   KPI = conversiones (compras completadas): elegido ✅                                        |
|                                                                                                 |
| -   KPI = revenue: descartado porque el precio medio varía mucho por categoría de producto      |
|                                                                                                 |
| -   KPI = clicks o impresiones: descartado (métrica intermedia, no de negocio)                  |
|                                                                                                 |
| Decisión final: kpi_type = \'non_revenue\', midiendo conversiones totales semanales por región. |
+-------------------------------------------------------------------------------------------------+

## Paso 0.3: Inventario inicial de campañas

Exploración rápida desde Windsor para confirmar la estructura de datos disponibles.

+-----------------------------------------------------------------------+
| \# Exploración inicial desde Windsor                                  |
|                                                                       |
| import requests                                                       |
|                                                                       |
| url = \'https://connect.windsor.ai/all\'                              |
|                                                                       |
| params = {                                                            |
|                                                                       |
| \'api_key\': \'TU_API_KEY\',                                          |
|                                                                       |
| \'date_preset\': \'last_18_months\',                                  |
|                                                                       |
| \'connector\': \'facebook,google_ads\',                               |
|                                                                       |
| \'fields\': \'date,source,campaign,objective,region,spend\'           |
|                                                                       |
| }                                                                     |
|                                                                       |
| response = requests.get(url, params=params)                           |
|                                                                       |
| df_preview = pd.DataFrame(response.json()\[\'data\'\])                |
|                                                                       |
| \# Resumen                                                            |
|                                                                       |
| print(f\'Filas: {len(df_preview)}\')                                  |
|                                                                       |
| print(f\'Campañas únicas: {df_preview.campaign.nunique()}\')          |
|                                                                       |
| print(f\'Objetivos: {df_preview.objective.unique()}\')                |
|                                                                       |
| print(f\'Regiones: {df_preview.region.unique()}\')                    |
|                                                                       |
| print(f\'Gasto total: \${df_preview.spend.sum():,.0f}\')              |
+-----------------------------------------------------------------------+

+----------------------------------------------------------------------------------------------------------------------------+
| **📌 Output del inventario para ModaViva**                                                                                 |
|                                                                                                                            |
| Filas: 47.320                                                                                                              |
|                                                                                                                            |
| Campañas únicas: 312                                                                                                       |
|                                                                                                                            |
| Objetivos encontrados: REACH, BRAND_AWARENESS, VIDEO_VIEWS, LINK_CLICKS, LANDING_PAGE_VIEWS, SEARCH, YOUTUBE_VIEW, DISPLAY |
|                                                                                                                            |
| Regiones: Bogotá, Medellín, Cali, Barranquilla, Bucaramanga                                                                |
|                                                                                                                            |
| Gasto total: 1.161 millones COP en 18 meses                                                                                |
+----------------------------------------------------------------------------------------------------------------------------+

## Paso 0.4: Setup del entorno técnico

-   Crear proyecto en Google Cloud (para BigQuery si se requiere)

-   Configurar Google Colab Pro+ con acceso a GPU T4 o A100

-   Instalar librerías necesarias:

+-----------------------------------------------------------------------+
| pip install google-meridian\[and-cuda\]                               |
|                                                                       |
| pip install pandas numpy scipy tensorflow tensorflow-probability      |
|                                                                       |
| pip install requests matplotlib seaborn                               |
+-----------------------------------------------------------------------+

-   Solicitar API key de Windsor.ai para el ambiente de desarrollo

-   Crear repositorio Git para versionar el pipeline

-   Documentar credenciales y accesos en un vault seguro

# Fase 1. Extracción y estructuración de datos

Duración: 2-3 semanas

Esta es la fase más extensa y crítica. La calidad del modelo depende directamente de la calidad de los datos. Típicamente concentra el 70% del éxito del proyecto.

## Paso 1.1: Extracción completa desde Windsor

Se extraen los datos históricos completos con todos los campos necesarios para el modelado.

+-----------------------------------------------------------------------+
| def fetch_complete_data(api_key):                                     |
|                                                                       |
| url = \'https://connect.windsor.ai/all\'                              |
|                                                                       |
| params = {                                                            |
|                                                                       |
| \'api_key\': api_key,                                                 |
|                                                                       |
| \'date_from\': \'2024-01-01\',                                        |
|                                                                       |
| \'date_to\': \'2025-06-30\',                                          |
|                                                                       |
| \'connector\': \'facebook,google_ads\',                               |
|                                                                       |
| \'fields\': \',\'.join(\[                                             |
|                                                                       |
| \'date\', \'source\', \'campaign\', \'objective\', \'region\',        |
|                                                                       |
| \'spend\', \'impressions\', \'clicks\', \'reach\', \'frequency\',     |
|                                                                       |
| \'conversions\', \'revenue\'                                          |
|                                                                       |
| \])                                                                   |
|                                                                       |
| }                                                                     |
|                                                                       |
| response = requests.get(url, params=params)                           |
|                                                                       |
| df = pd.DataFrame(response.json()\[\'data\'\])                        |
|                                                                       |
| df\[\'date\'\] = pd.to_datetime(df\[\'date\'\])                       |
|                                                                       |
| return df                                                             |
|                                                                       |
| df_raw = fetch_complete_data(api_key=\'XXXX\')                        |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------------------------------------------+
| **📌 Resultado de la extracción de ModaViva**                                                                               |
|                                                                                                                             |
| Dataset resultante:                                                                                                         |
|                                                                                                                             |
| -   47.320 filas (una por día × campaña × región)                                                                           |
|                                                                                                                             |
| -   Columnas: date, source, campaign, objective, region, spend, impressions, clicks, reach, frequency, conversions, revenue |
|                                                                                                                             |
| -   Periodo cubierto: 01-ene-2024 a 30-jun-2025                                                                             |
|                                                                                                                             |
| -   Tamaño del archivo: 12.3 MB CSV                                                                                         |
+-----------------------------------------------------------------------------------------------------------------------------+

## Paso 1.2: Auditoría de calidad

Antes de procesar, se verifica la integridad del histórico buscando huecos, inconsistencias, outliers y cambios estructurales.

+------------------------------------------------------------------------------------+
| \# Verificar completitud temporal                                                  |
|                                                                                    |
| weeks = pd.date_range(\'2024-01-01\', \'2025-06-30\', freq=\'W\')                  |
|                                                                                    |
| weeks_with_data = df_raw.groupby(df_raw.date.dt.to_period(\'W\')).size()           |
|                                                                                    |
| missing_weeks = \[w for w in weeks if w not in weeks_with_data.index\]             |
|                                                                                    |
| print(f\'Semanas sin datos: {len(missing_weeks)}\')                                |
|                                                                                    |
| \# Detectar outliers de gasto                                                      |
|                                                                                    |
| spend_by_week = df_raw.groupby(df_raw.date.dt.to_period(\'W\'))\[\'spend\'\].sum() |
|                                                                                    |
| z_scores = (spend_by_week - spend_by_week.mean()) / spend_by_week.std()            |
|                                                                                    |
| outlier_weeks = spend_by_week\[abs(z_scores) \> 3\]                                |
|                                                                                    |
| print(f\'Semanas outlier (\|z\| \> 3): {len(outlier_weeks)}\')                     |
|                                                                                    |
| \# Consistencia por región                                                         |
|                                                                                    |
| region_coverage = df_raw.groupby(\'region\').agg(                                  |
|                                                                                    |
| first_date=(\'date\', \'min\'),                                                    |
|                                                                                    |
| last_date=(\'date\', \'max\'),                                                     |
|                                                                                    |
| weeks_with_data=(\'date\', lambda x: x.dt.to_period(\'W\').nunique())              |
|                                                                                    |
| )                                                                                  |
|                                                                                    |
| print(region_coverage)                                                             |
+------------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------------------------------+
| **📌 Resultado de auditoría en ModaViva**                                                                         |
|                                                                                                                   |
| Hallazgos principales:                                                                                            |
|                                                                                                                   |
| -   Semanas sin datos: 0 (histórico completo)                                                                     |
|                                                                                                                   |
| -   Semanas outlier: 2 (Black Friday 2024 y día de la madre 2025, explicables)                                    |
|                                                                                                                   |
| -   Cobertura regional: todas las 5 regiones tienen datos desde ene-2024                                          |
|                                                                                                                   |
| -   Detectado: cambio de tracking el 15-mar-2024 (implementación de Enhanced Conversions), considerar en modelado |
|                                                                                                                   |
| Conclusión: datos en buen estado, proceder con pipeline.                                                          |
+-------------------------------------------------------------------------------------------------------------------+

## Paso 1.3: Clasificación de campañas en subcanales

Las 312 campañas individuales se agrupan en subcanales estratégicos según objetivo y plataforma.

+----------------------------------------------------------------------------------------+
| def classify_campaign(row):                                                            |
|                                                                                        |
| source = row\[\'source\'\].lower()                                                     |
|                                                                                        |
| objective = str(row.get(\'objective\', \'\')).lower()                                  |
|                                                                                        |
| campaign = row\[\'campaign\'\].lower()                                                 |
|                                                                                        |
| if source == \'facebook\':                                                             |
|                                                                                        |
| if objective in \[\'reach\', \'brand_awareness\', \'video_views\'\]:                   |
|                                                                                        |
| return \'meta_awareness\'                                                              |
|                                                                                        |
| elif objective in \[\'link_clicks\', \'landing_page_views\'\]:                         |
|                                                                                        |
| return \'meta_traffic\'                                                                |
|                                                                                        |
| else:                                                                                  |
|                                                                                        |
| return \'meta_other\'                                                                  |
|                                                                                        |
| elif source == \'google_ads\':                                                         |
|                                                                                        |
| if \'youtube\' in objective or \'video\' in campaign:                                  |
|                                                                                        |
| return \'google_awareness\'                                                            |
|                                                                                        |
| elif objective in \[\'search\', \'display\', \'discovery\'\]:                          |
|                                                                                        |
| return \'google_traffic\'                                                              |
|                                                                                        |
| else:                                                                                  |
|                                                                                        |
| return \'google_other\'                                                                |
|                                                                                        |
| df_raw\[\'sub_channel\'\] = df_raw.apply(classify_campaign, axis=1)                    |
|                                                                                        |
| \# Verificar cobertura de la clasificación                                             |
|                                                                                        |
| print(df_raw.groupby(\'sub_channel\')\[\'spend\'\].sum().sort_values(ascending=False)) |
+----------------------------------------------------------------------------------------+

+----------------------------------------------------------------------------+
| **📌 Distribución de gasto por subcanal en ModaViva**                      |
|                                                                            |
|   ------------------------------------------------------------------------ |
|   **Subcanal**                 **Gasto total**       **% del total**       |
|   ---------------------------- --------------------- --------------------- |
|   meta_traffic                 \$412.580.000         35.5%                 |
|                                                                            |
|   google_traffic               \$385.220.000         33.2%                 |
|                                                                            |
|   meta_awareness               \$198.450.000         17.1%                 |
|                                                                            |
|   google_awareness             \$164.800.000         14.2%                 |
|   ------------------------------------------------------------------------ |
+----------------------------------------------------------------------------+

## Paso 1.4: Agregación semanal por región

Los datos diarios por campaña se agregan a nivel semanal × región × subcanal.

+---------------------------------------------------------------------------------------+
| df_raw\[\'week\'\] = df_raw\[\'date\'\].dt.to_period(\'W\').dt.start_time             |
|                                                                                       |
| \# Agregación por semana, región y subcanal                                           |
|                                                                                       |
| df_weekly = df_raw.groupby(\[\'week\', \'region\', \'sub_channel\'\]).agg({           |
|                                                                                       |
| \'spend\': \'sum\',                                                                   |
|                                                                                       |
| \'impressions\': \'sum\',                                                             |
|                                                                                       |
| \'clicks\': \'sum\',                                                                  |
|                                                                                       |
| \'conversions\': \'sum\',                                                             |
|                                                                                       |
| \'revenue\': \'sum\'                                                                  |
|                                                                                       |
| }).reset_index()                                                                      |
|                                                                                       |
| \# Pivotar para tener columnas por subcanal                                           |
|                                                                                       |
| df_pivoted = df_weekly.pivot_table(                                                   |
|                                                                                       |
| index=\[\'week\', \'region\'\],                                                       |
|                                                                                       |
| columns=\'sub_channel\',                                                              |
|                                                                                       |
| values=\[\'spend\', \'impressions\'\],                                                |
|                                                                                       |
| aggfunc=\'sum\',                                                                      |
|                                                                                       |
| fill_value=0                                                                          |
|                                                                                       |
| )                                                                                     |
|                                                                                       |
| df_pivoted.columns = \[f\'{col\[1\]}\_{col\[0\]}\' for col in df_pivoted.columns\]    |
|                                                                                       |
| df_pivoted = df_pivoted.reset_index()                                                 |
|                                                                                       |
| \# Agregar KPI total                                                                  |
|                                                                                       |
| kpi = df_raw.groupby(\[\'week\', \'region\'\])\[\'conversions\'\].sum().reset_index() |
|                                                                                       |
| df_final = df_pivoted.merge(kpi, on=\[\'week\', \'region\'\])                         |
+---------------------------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------+
| **📌 Muestra del dataset resultante de ModaViva**                                                       |
|                                                                                                         |
|   ----------------------------------------------------------------------------------------------------- |
|   **week**    **region**   **meta_aware**   **meta_traffic**   **g_aware**   **g_traffic**   **conv**   |
|   ----------- ------------ ---------------- ------------------ ------------- --------------- ---------- |
|   2024-W01    Bogotá       8.200            15.600             6.800         14.300          287        |
|                                                                                                         |
|   2024-W01    Medellín     4.100            7.800              3.400         7.200           142        |
|                                                                                                         |
|   2024-W01    Cali         2.500            4.600              2.100         4.300           85         |
|                                                                                                         |
|   2024-W01    B/quilla     1.800            3.500              1.500         3.100           62         |
|                                                                                                         |
|   2024-W01    B/manga      1.200            2.400              1.000         2.200           41         |
|   ----------------------------------------------------------------------------------------------------- |
|                                                                                                         |
| *Total: 78 semanas × 5 regiones = 390 filas, listas para Meridian.*                                     |
+---------------------------------------------------------------------------------------------------------+

## Paso 1.5: Adición de variables de control

Sin estos controles, Meridian atribuirá erróneamente a los canales efectos que son de estacionalidad o factores externos.

+-----------------------------------------------------------------------------------------------------+
| \# Holidays y fechas especiales Colombia                                                            |
|                                                                                                     |
| HOLIDAYS = \[                                                                                       |
|                                                                                                     |
| \'2024-01-01\', \'2024-03-25\', \'2024-05-01\',                                                     |
|                                                                                                     |
| \'2024-05-12\', \# día madre                                                                        |
|                                                                                                     |
| \'2024-11-29\', \# Black Friday                                                                     |
|                                                                                                     |
| \'2024-12-08\', \'2024-12-25\',                                                                     |
|                                                                                                     |
| \'2025-05-11\', \# día madre 2025                                                                   |
|                                                                                                     |
| \# \...                                                                                             |
|                                                                                                     |
| \]                                                                                                  |
|                                                                                                     |
| df_final\[\'holiday_flag\'\] = df_final\[\'week\'\].dt.date.astype(str).isin(HOLIDAYS).astype(int)  |
|                                                                                                     |
| \# Google Query Volume de marca (desde Google Trends)                                               |
|                                                                                                     |
| gqv_data = pd.read_csv(\'modaviva_gqv_weekly.csv\')                                                 |
|                                                                                                     |
| df_final = df_final.merge(gqv_data, on=\[\'week\', \'region\'\])                                    |
|                                                                                                     |
| \# Precio promedio semanal (desde sistema interno del cliente)                                      |
|                                                                                                     |
| price_data = pd.read_csv(\'modaviva_prices_weekly.csv\')                                            |
|                                                                                                     |
| df_final = df_final.merge(price_data, on=\'week\')                                                  |
|                                                                                                     |
| \# Flag de promociones internas                                                                     |
|                                                                                                     |
| promo_weeks = \[\'2024-03-15\', \'2024-07-20\', \'2024-11-25\', \'2025-02-14\'\]                    |
|                                                                                                     |
| df_final\[\'promo_flag\'\] = df_final\[\'week\'\].dt.date.astype(str).isin(promo_weeks).astype(int) |
|                                                                                                     |
| df_final.to_csv(\'modaviva_meridian_input.csv\', index=False)                                       |
+-----------------------------------------------------------------------------------------------------+

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Por qué son críticas las variables de control**                                                                                                                                                                                                                                                                              |
|                                                                                                                                                                                                                                                                                                                                |
| Si en diciembre aumentas Meta de \$10k a \$20k semanales, y las ventas suben de 100 a 180, sin controles el modelo concluye que Meta duplicó su impacto. La realidad puede ser que diciembre naturalmente vende 60% más por estacionalidad, y Meta solo aportó 20%. Sin holiday_flag, el modelo sobreestima gravemente a Meta. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# Fase 2. Estimación empírica de parámetros

Duración: 1 semana

Se calculan los 4 parámetros clave (ROI, adstock α, half-saturation ec, slope s) por cada subcanal usando los datos históricos. Estos parámetros se usarán como priors informados para Meridian.

## Paso 2.1: Cálculo del ROI empírico por subcanal

Se calcula el ROAS observado y se ajusta con factores de incrementalidad específicos por tipo de campaña.

+---------------------------------------------------------------------------------+
| CHANNELS = \[\'meta_awareness\', \'meta_traffic\',                              |
|                                                                                 |
| \'google_awareness\', \'google_traffic\'\]                                      |
|                                                                                 |
| \# Factores de ajuste ROAS -\> ROI incremental                                  |
|                                                                                 |
| adjustment_factors = {                                                          |
|                                                                                 |
| \'meta_awareness\': 0.55,                                                       |
|                                                                                 |
| \'meta_traffic\': 0.70,                                                         |
|                                                                                 |
| \'google_awareness\': 0.60,                                                     |
|                                                                                 |
| \'google_traffic\': 0.65                                                        |
|                                                                                 |
| }                                                                               |
|                                                                                 |
| AOV = 180000 \# ticket promedio en COP                                          |
|                                                                                 |
| roi_empirical = {}                                                              |
|                                                                                 |
| for ch in CHANNELS:                                                             |
|                                                                                 |
| spend_col = f\'{ch}\_spend\'                                                    |
|                                                                                 |
| total_spend = df_final\[spend_col\].sum()                                       |
|                                                                                 |
| total_conv = df_final\[\'conversions\'\].sum()                                  |
|                                                                                 |
| total_spend_all = df_final\[\[f\'{c}\_spend\' for c in CHANNELS\]\].sum().sum() |
|                                                                                 |
| spend_share = total_spend / total_spend_all                                     |
|                                                                                 |
| attributed_conv = total_conv \* spend_share                                     |
|                                                                                 |
| attributed_revenue = attributed_conv \* AOV                                     |
|                                                                                 |
| roas = attributed_revenue / total_spend                                         |
|                                                                                 |
| roi_incremental = roas \* adjustment_factors\[ch\]                              |
|                                                                                 |
| roi_empirical\[ch\] = roi_incremental                                           |
+---------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------+
| **📌 Resultados de ROI para ModaViva**                                                    |
|                                                                                           |
|   --------------------------------------------------------------------------------------- |
|   **Subcanal**       **ROAS bruto**   **Factor**     **ROI incr.**   **Interpretación**   |
|   ------------------ ---------------- -------------- --------------- -------------------- |
|   meta_awareness     2.80x            0.55           1.54x           Brand lift           |
|                                                                                           |
|   meta_traffic       3.20x            0.70           2.24x           Clicks               |
|                                                                                           |
|   google_awareness   2.60x            0.60           1.56x           YouTube              |
|                                                                                           |
|   google_traffic     3.80x            0.65           2.47x           Search + Display     |
|   --------------------------------------------------------------------------------------- |
+-------------------------------------------------------------------------------------------+

## Paso 2.2: Estimación del parámetro adstock α

Se usa grid search para encontrar el valor de α que maximiza la correlación entre gasto transformado (con adstock) y conversiones.

+-----------------------------------------------------------------------+
| from scipy.stats import pearsonr                                      |
|                                                                       |
| def apply_adstock(spend, alpha, max_lag=6):                           |
|                                                                       |
| adstocked = np.zeros(len(spend))                                      |
|                                                                       |
| for t in range(len(spend)):                                           |
|                                                                       |
| for l in range(min(t + 1, max_lag)):                                  |
|                                                                       |
| adstocked\[t\] += (alpha \*\* l) \* spend\[t - l\]                    |
|                                                                       |
| return adstocked                                                      |
|                                                                       |
| def estimate_alpha(df, spend_col, kpi_col=\'conversions\'):           |
|                                                                       |
| agg = df.groupby(\'week\').agg({                                      |
|                                                                       |
| spend_col: \'sum\', kpi_col: \'sum\'                                  |
|                                                                       |
| }).reset_index()                                                      |
|                                                                       |
| best_alpha, best_corr = None, -1                                      |
|                                                                       |
| for alpha in np.arange(0.05, 0.95, 0.05):                             |
|                                                                       |
| adstocked = apply_adstock(agg\[spend_col\].values, alpha, max_lag=6)  |
|                                                                       |
| corr, \_ = pearsonr(adstocked, agg\[kpi_col\].values)                 |
|                                                                       |
| if corr \> best_corr:                                                 |
|                                                                       |
| best_corr = corr                                                      |
|                                                                       |
| best_alpha = alpha                                                    |
|                                                                       |
| return best_alpha, best_corr                                          |
|                                                                       |
| alpha_empirical = {}                                                  |
|                                                                       |
| for ch in CHANNELS:                                                   |
|                                                                       |
| alpha, corr = estimate_alpha(df_final, f\'{ch}\_spend\')              |
|                                                                       |
| alpha_empirical\[ch\] = alpha                                         |
|                                                                       |
| print(f\'{ch}: α = {alpha:.2f} (corr = {corr:.3f})\')                 |
+-----------------------------------------------------------------------+

+------------------------------------------------------------------------------------------------------------------------------------------------------+
| **📌 Adstock estimado para ModaViva**                                                                                                                |
|                                                                                                                                                      |
|   ---------------------------------------------------------------------------                                                                        |
|   **Subcanal**            **α estimado**   **Correlación**   **Vida media**                                                                          |
|   ----------------------- ---------------- ----------------- ----------------                                                                        |
|   meta_awareness          0.70             0.612             ≈ 2 semanas                                                                             |
|                                                                                                                                                      |
|   meta_traffic            0.35             0.783             ≈ 0.7 sem                                                                               |
|                                                                                                                                                      |
|   google_awareness        0.65             0.591             ≈ 1.6 sem                                                                               |
|                                                                                                                                                      |
|   google_traffic          0.25             0.821             ≈ 0.5 sem                                                                               |
|   ---------------------------------------------------------------------------                                                                        |
|                                                                                                                                                      |
| *Observación: awareness (video, branding) tiene α alto (efecto largo) y traffic (clicks) tiene α bajo (efecto inmediato). Tiene sentido de negocio.* |
+------------------------------------------------------------------------------------------------------------------------------------------------------+

## Paso 2.3: Estimación de curvas de saturación

Se ajusta la función Hill a pares (gasto semanal, conversiones semanales) para obtener ec (half-saturation) y slope.

+-----------------------------------------------------------------------+
| from scipy.optimize import curve_fit                                  |
|                                                                       |
| def hill_function(x, ec, slope, vmax):                                |
|                                                                       |
| return vmax \* (x \*\* slope) / (x \*\* slope + ec \*\* slope)        |
|                                                                       |
| def estimate_saturation(df, spend_col):                               |
|                                                                       |
| agg = df.groupby(\'week\').agg({                                      |
|                                                                       |
| spend_col: \'sum\', \'conversions\': \'sum\'                          |
|                                                                       |
| }).reset_index()                                                      |
|                                                                       |
| x = agg\[spend_col\].values                                           |
|                                                                       |
| y = agg\[\'conversions\'\].values                                     |
|                                                                       |
| try:                                                                  |
|                                                                       |
| popt, \_ = curve_fit(                                                 |
|                                                                       |
| hill_function, x, y,                                                  |
|                                                                       |
| p0=\[np.median(x), 1.0, y.max() \* 1.5\],                             |
|                                                                       |
| bounds=(\[x.min(), 0.1, 0\],                                          |
|                                                                       |
| \[x.max() \* 10, 5, y.max() \* 10\]),                                 |
|                                                                       |
| maxfev=5000                                                           |
|                                                                       |
| )                                                                     |
|                                                                       |
| return {\'ec\': popt\[0\], \'slope\': popt\[1\], \'vmax\': popt\[2\]} |
|                                                                       |
| except:                                                               |
|                                                                       |
| return None                                                           |
|                                                                       |
| saturation_empirical = {}                                             |
|                                                                       |
| for ch in CHANNELS:                                                   |
|                                                                       |
| result = estimate_saturation(df_final, f\'{ch}\_spend\')              |
|                                                                       |
| saturation_empirical\[ch\] = result                                   |
+-----------------------------------------------------------------------+

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **📌 Saturación estimada para ModaViva**                                                                                                                                                                                     |
|                                                                                                                                                                                                                              |
|   -----------------------------------------------------------------------------                                                                                                                                              |
|   **Subcanal**            **ec (media saturación)**   **slope**                                                                                                                                                              |
|   ----------------------- --------------------------- -------------------------                                                                                                                                              |
|   meta_awareness          \$42.000 / semana           1.15 (saturación suave)                                                                                                                                                |
|                                                                                                                                                                                                                              |
|   meta_traffic            \$68.000 / semana           1.45 (moderada)                                                                                                                                                        |
|                                                                                                                                                                                                                              |
|   google_awareness        \$38.000 / semana           1.20 (suave)                                                                                                                                                           |
|                                                                                                                                                                                                                              |
|   google_traffic          \$58.000 / semana           1.60 (marcada)                                                                                                                                                         |
|   -----------------------------------------------------------------------------                                                                                                                                              |
|                                                                                                                                                                                                                              |
| *Interpretación: Meta Awareness alcanza media saturación a \$42k/semana, con curva suave (rendimientos decrecientes graduales). Google Traffic satura más rápido: a partir de \$58k cada dólar adicional rinde mucho menos.* |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## Paso 2.4: Tabla consolidada de parámetros

El entregable final de esta fase es una tabla con los 4 parámetros por subcanal, lista para alimentar los priors de Meridian.

+------------------------------------------------------------------------------+
| **📌 Tabla final de parámetros empíricos de ModaViva**                       |
|                                                                              |
|   -------------------------------------------------------------------------- |
|   **Subcanal**       **ROI**      **α**        **ec (COP)**   **slope**      |
|   ------------------ ------------ ------------ -------------- -------------- |
|   meta_awareness     1.54x        0.70         42.000         1.15           |
|                                                                              |
|   meta_traffic       2.24x        0.35         68.000         1.45           |
|                                                                              |
|   google_awareness   1.56x        0.65         38.000         1.20           |
|                                                                              |
|   google_traffic     2.47x        0.25         58.000         1.60           |
|   -------------------------------------------------------------------------- |
+------------------------------------------------------------------------------+

# Fase 3. Modelado con Meridian

Duración: 2 semanas

Con los parámetros empíricos de la Fase 2, se construyen priors informados muy estrechos y se entrena el modelo Meridian. Con priors estrechos, el entrenamiento converge rápidamente.

## Paso 3.1: Construcción de priors informados

Cada parámetro empírico se convierte en una distribución de probabilidad con uncertainty bajo (0.10-0.20) para reflejar alta confianza en los valores estimados.

+------------------------------------------------------------------------------+
| import tensorflow_probability as tfp                                         |
|                                                                              |
| import numpy as np                                                           |
|                                                                              |
| from meridian.model import prior_distribution                                |
|                                                                              |
| tfd = tfp.distributions                                                      |
|                                                                              |
| CONFIDENCE_ROI = 0.15 \# más amplio por histórico corto                      |
|                                                                              |
| CONFIDENCE_EC = 0.20                                                         |
|                                                                              |
| CONFIDENCE_SLOPE = 0.25                                                      |
|                                                                              |
| roi_means = \[roi_empirical\[ch\] for ch in CHANNELS\]                       |
|                                                                              |
| alpha_means = \[alpha_empirical\[ch\] for ch in CHANNELS\]                   |
|                                                                              |
| ec_means = \[saturation_empirical\[ch\]\[\'ec\'\] for ch in CHANNELS\]       |
|                                                                              |
| slope_means = \[saturation_empirical\[ch\]\[\'slope\'\] for ch in CHANNELS\] |
|                                                                              |
| informed_prior = prior_distribution.PriorDistribution(                       |
|                                                                              |
| roi_m=tfd.LogNormal(                                                         |
|                                                                              |
| loc=np.log(roi_means) - (CONFIDENCE_ROI \*\* 2) / 2,                         |
|                                                                              |
| scale=\[CONFIDENCE_ROI\] \* len(CHANNELS)                                    |
|                                                                              |
| ),                                                                           |
|                                                                              |
| alpha_m=tfd.Beta(                                                            |
|                                                                              |
| concentration1=np.array(alpha_means) \* 40,                                  |
|                                                                              |
| concentration0=(1 - np.array(alpha_means)) \* 40                             |
|                                                                              |
| ),                                                                           |
|                                                                              |
| ec_m=tfd.LogNormal(                                                          |
|                                                                              |
| loc=np.log(ec_means) - (CONFIDENCE_EC \*\* 2) / 2,                           |
|                                                                              |
| scale=\[CONFIDENCE_EC\] \* len(CHANNELS)                                     |
|                                                                              |
| ),                                                                           |
|                                                                              |
| slope_m=tfd.LogNormal(                                                       |
|                                                                              |
| loc=np.log(slope_means) - (CONFIDENCE_SLOPE \*\* 2) / 2,                     |
|                                                                              |
| scale=\[CONFIDENCE_SLOPE\] \* len(CHANNELS)                                  |
|                                                                              |
| )                                                                            |
|                                                                              |
| )                                                                            |
+------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------+
| **📌 Priors construidos para ModaViva**                                     |
|                                                                             |
| Distribuciones resultantes (ejemplo para meta_traffic):                     |
|                                                                             |
| -   ROI: LogNormal(mean=2.24, scale=0.15) → 90% prob entre 1.75x y 2.86x    |
|                                                                             |
| -   α: Beta(14, 26) → 90% prob entre 0.22 y 0.50                            |
|                                                                             |
| -   ec: LogNormal(mean=68.000, scale=0.20) → 90% prob entre 49.000 y 94.000 |
|                                                                             |
| -   slope: LogNormal(mean=1.45, scale=0.25) → 90% prob entre 0.96 y 2.19    |
+-----------------------------------------------------------------------------+

## Paso 3.2: Carga de datos y especificación del modelo

+--------------------------------------------------------------------------+
| from meridian.data import load                                           |
|                                                                          |
| from meridian.model import model, spec                                   |
|                                                                          |
| loader = load.CsvDataLoader(                                             |
|                                                                          |
| csv_path=\'modaviva_meridian_input.csv\',                                |
|                                                                          |
| kpi_type=\'non_revenue\',                                                |
|                                                                          |
| coord_to_columns=load.CoordToColumns(                                    |
|                                                                          |
| time=\'week\',                                                           |
|                                                                          |
| geo=\'region\',                                                          |
|                                                                          |
| kpi=\'conversions\',                                                     |
|                                                                          |
| controls=\[\'holiday_flag\', \'gqv\', \'price_index\', \'promo_flag\'\], |
|                                                                          |
| media_spend=\[f\'{ch}\_spend\' for ch in CHANNELS\],                     |
|                                                                          |
| media=\[f\'{ch}\_impressions\' for ch in CHANNELS\]                      |
|                                                                          |
| ),                                                                       |
|                                                                          |
| media_to_channel={f\'{ch}\_impressions\': ch for ch in CHANNELS},        |
|                                                                          |
| media_spend_to_channel={f\'{ch}\_spend\': ch for ch in CHANNELS}         |
|                                                                          |
| )                                                                        |
|                                                                          |
| data = loader.load()                                                     |
|                                                                          |
| model_spec = spec.ModelSpec(                                             |
|                                                                          |
| prior=informed_prior,                                                    |
|                                                                          |
| media_effects_dist=\'log_normal\',                                       |
|                                                                          |
| hill_before_adstock=False,                                               |
|                                                                          |
| max_lag=6, \# reducido por histórico corto                               |
|                                                                          |
| media_prior_type=\'roi\' \# clave para Ruta 1                            |
|                                                                          |
| )                                                                        |
+--------------------------------------------------------------------------+

## Paso 3.3: Entrenamiento con MCMC

Con priors estrechos, el muestreo posterior converge rápidamente en comparación con un modelo sin información previa.

+-----------------------------------------------------------------------+
| mmm = model.Meridian(input_data=data, model_spec=model_spec)          |
|                                                                       |
| \# Sampleo del prior (validación)                                     |
|                                                                       |
| print(\'Sampling prior\...\')                                         |
|                                                                       |
| mmm.sample_prior(n_draws=500)                                         |
|                                                                       |
| \# Entrenamiento principal                                            |
|                                                                       |
| print(\'Sampling posterior\...\')                                     |
|                                                                       |
| mmm.sample_posterior(                                                 |
|                                                                       |
| n_chains=4, \# más chains por histórico corto                         |
|                                                                       |
| n_adapt=300,                                                          |
|                                                                       |
| n_burnin=300,                                                         |
|                                                                       |
| n_keep=600,                                                           |
|                                                                       |
| seed=42                                                               |
|                                                                       |
| )                                                                     |
|                                                                       |
| print(\'Entrenamiento completado\')                                   |
+-----------------------------------------------------------------------+

+-----------------------------------------------------------------------+
| **📌 Métricas de entrenamiento de ModaViva**                          |
|                                                                       |
| -   Hardware: GPU NVIDIA T4 (Google Colab Pro+)                       |
|                                                                       |
| -   Tiempo total de sampling posterior: 32 minutos                    |
|                                                                       |
| -   Total de draws: 4 chains × 600 = 2.400 muestras posteriores       |
|                                                                       |
| -   Memoria RAM usada: 4.2 GB                                         |
|                                                                       |
| -   Status: completado exitosamente sin warnings críticos             |
+-----------------------------------------------------------------------+

## Paso 3.4: Validación de convergencia

Antes de usar el modelo, se valida que el muestreo convergió correctamente.

+------------------------------------------------------------------------------+
| from meridian.analysis import analyzer                                       |
|                                                                              |
| diagnostics = analyzer.ModelDiagnostics(mmm)                                 |
|                                                                              |
| r_hat = diagnostics.get_rhat()                                               |
|                                                                              |
| print(f\'Max R-hat: {float(r_hat.max()):.3f}\') \# debe ser \< 1.1           |
|                                                                              |
| \# Comparación ROI empírico vs estimado por el modelo                        |
|                                                                              |
| print(\'\\nComparación ROI empírico vs modelo:\')                            |
|                                                                              |
| for i, ch in enumerate(CHANNELS):                                            |
|                                                                              |
| modeled_roi = float(mmm.inference_data\[\'roi_m\'\]                          |
|                                                                              |
| .mean(dim=\[\'chain\', \'draw\'\]).values\[i\])                              |
|                                                                              |
| empirical_roi = roi_empirical\[ch\]                                          |
|                                                                              |
| diff = abs(modeled_roi - empirical_roi) / empirical_roi                      |
|                                                                              |
| print(f\'{ch}: empírico {empirical_roi:.2f}x vs modelo {modeled_roi:.2f}x\', |
|                                                                              |
| f\'(diff {diff:.1%})\')                                                      |
+------------------------------------------------------------------------------+

+-------------------------------------------------------------------------------------+
| **📌 Resultados de validación de ModaViva**                                         |
|                                                                                     |
| Diagnósticos:                                                                       |
|                                                                                     |
| -   Max R-hat: 1.045 ✅ (\< 1.1)                                                    |
|                                                                                     |
| -   Effective sample size mínimo: 820 ✅ (\> 400)                                   |
|                                                                                     |
| -   No hay divergencias ni warnings                                                 |
|                                                                                     |
| **Comparación ROI empírico vs modelo:**                                             |
|                                                                                     |
| -   meta_awareness: empírico 1.54x vs modelo 1.58x (diff 2.6%)                      |
|                                                                                     |
| -   meta_traffic: empírico 2.24x vs modelo 2.19x (diff 2.2%)                        |
|                                                                                     |
| -   google_awareness: empírico 1.56x vs modelo 1.61x (diff 3.2%)                    |
|                                                                                     |
| -   google_traffic: empírico 2.47x vs modelo 2.38x (diff 3.6%)                      |
|                                                                                     |
| **✅ Todas las diferencias \< 5%: los priors estrechos funcionaron correctamente.** |
+-------------------------------------------------------------------------------------+

# Fase 4. Optimización de presupuesto

Duración: 1 semana

Con el modelo validado, se usa el Budget Optimizer de Meridian para generar recomendaciones concretas de reasignación. Se corren múltiples escenarios según necesidades del cliente.

## Paso 4.1: Baseline actual

Antes de optimizar, se documenta la distribución actual del presupuesto para poder comparar.

+---------------------------------------------------------------------------+
| **📌 Presupuesto semanal actual de ModaViva**                             |
|                                                                           |
|   ----------------------------------------------------------------------- |
|   **Subcanal**            **Gasto actual**        **% del total**         |
|   ----------------------- ----------------------- ----------------------- |
|   meta_awareness          \$42.000                17.0%                   |
|                                                                           |
|   meta_traffic            \$88.000                35.6%                   |
|                                                                           |
|   google_awareness        \$35.000                14.2%                   |
|                                                                           |
|   google_traffic          \$82.000                33.2%                   |
|                                                                           |
|   **TOTAL**               **\$247.000/sem**       **100%**                |
|   ----------------------------------------------------------------------- |
|                                                                           |
| *Conversiones promedio semanales con esta distribución: 947*              |
+---------------------------------------------------------------------------+

## Paso 4.2: Escenario 1 --- Mismo presupuesto, mejor distribución

La pregunta más común: ¿cómo reasignar lo que ya gastamos para obtener más conversiones?

+-----------------------------------------------------------------------+
| from meridian.analysis import optimizer                               |
|                                                                       |
| budget_opt = optimizer.BudgetOptimizer(mmm)                           |
|                                                                       |
| opt_fixed = budget_opt.optimize(                                      |
|                                                                       |
| use_kpi=True,                                                         |
|                                                                       |
| fixed_budget=True                                                     |
|                                                                       |
| )                                                                     |
|                                                                       |
| print(opt_fixed.spend_summary())                                      |
+-----------------------------------------------------------------------+

+---------------------------------------------------------------------------------------------------------+
| **📌 Recomendación del Escenario 1 para ModaViva**                                                      |
|                                                                                                         |
|   -------------------------------------------------------------------------------                       |
|   **Subcanal**       **Actual**     **Óptimo**     **Cambio \$**   **Cambio %**                         |
|   ------------------ -------------- -------------- --------------- --------------                       |
|   meta_awareness     \$42.000       \$31.000       -\$11.000       -26.2%                               |
|                                                                                                         |
|   meta_traffic       \$88.000       \$96.000       +\$8.000        +9.1%                                |
|                                                                                                         |
|   google_awareness   \$35.000       \$28.000       -\$7.000        -20.0%                               |
|                                                                                                         |
|   google_traffic     \$82.000       \$92.000       +\$10.000       +12.2%                               |
|   -------------------------------------------------------------------------------                       |
|                                                                                                         |
| **Impacto esperado:**                                                                                   |
|                                                                                                         |
| -   Conversiones actuales: 947/sem                                                                      |
|                                                                                                         |
| -   Conversiones con óptimo: 1.058/sem                                                                  |
|                                                                                                         |
| -   Lift esperado: +11.7% sin aumentar presupuesto                                                      |
|                                                                                                         |
| *Interpretación: reducir awareness (saturado) y reasignar a traffic (con más margen de escalabilidad).* |
+---------------------------------------------------------------------------------------------------------+

## Paso 4.3: Escenario 2 --- Con restricciones de negocio

El cliente requiere mantener al menos 25% del presupuesto en awareness por compromisos estratégicos de marca.

+-----------------------------------------------------------------------+
| \# Escenario con restricciones de negocio                             |
|                                                                       |
| constraints = {                                                       |
|                                                                       |
| \'meta_awareness\': {\'min_spend\': 30000, \'max_spend\': 60000},     |
|                                                                       |
| \'google_awareness\': {\'min_spend\': 25000, \'max_spend\': 50000},   |
|                                                                       |
| \'meta_traffic\': {\'min_spend\': 60000, \'max_spend\': 120000},      |
|                                                                       |
| \'google_traffic\': {\'min_spend\': 60000, \'max_spend\': 120000}     |
|                                                                       |
| }                                                                     |
|                                                                       |
| opt_constrained = budget_opt.optimize(                                |
|                                                                       |
| use_kpi=True,                                                         |
|                                                                       |
| fixed_budget=True,                                                    |
|                                                                       |
| channel_constraints=constraints                                       |
|                                                                       |
| )                                                                     |
+-----------------------------------------------------------------------+

+-------------------------------------------------------------------------------------------------------------------------------------+
| **📌 Recomendación del Escenario 2 para ModaViva**                                                                                  |
|                                                                                                                                     |
| Con restricciones (mínimo \$30k en meta_awareness y \$25k en google_awareness):                                                     |
|                                                                                                                                     |
| -   meta_awareness: \$42.000 → \$38.000 (-9.5%)                                                                                     |
|                                                                                                                                     |
| -   meta_traffic: \$88.000 → \$93.000 (+5.7%)                                                                                       |
|                                                                                                                                     |
| -   google_awareness: \$35.000 → \$30.000 (-14.3%)                                                                                  |
|                                                                                                                                     |
| -   google_traffic: \$82.000 → \$86.000 (+4.9%)                                                                                     |
|                                                                                                                                     |
| **Conversiones esperadas: 1.021/sem (+7.8% vs actual)**                                                                             |
|                                                                                                                                     |
| *El lift se reduce de 11.7% a 7.8% por las restricciones, pero sigue siendo mejora significativa y respeta la estrategia de marca.* |
+-------------------------------------------------------------------------------------------------------------------------------------+

## Paso 4.4: Escenario 3 --- Crecimiento marginal

¿Qué pasa si el cliente puede aumentar el presupuesto? ¿En qué canales debe invertir el dinero adicional?

+-------------------------------------------------------------------------+
| \# Simular distintos niveles de presupuesto                             |
|                                                                         |
| budgets = \[200000, 247000, 300000, 400000, 500000\]                    |
|                                                                         |
| for budget in budgets:                                                  |
|                                                                         |
| opt = budget_opt.optimize(                                              |
|                                                                         |
| use_kpi=True,                                                           |
|                                                                         |
| fixed_budget=True,                                                      |
|                                                                         |
| budget=budget                                                           |
|                                                                         |
| )                                                                       |
|                                                                         |
| expected_conv = opt.total_conversions_expected                          |
|                                                                         |
| print(f\'Presupuesto \${budget:,}: {expected_conv:,.0f} conversiones\') |
+-------------------------------------------------------------------------+

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **📌 Curva de escalabilidad de ModaViva**                                                                                                                                          |
|                                                                                                                                                                                    |
|   -----------------------------------------------------------------------------                                                                                                    |
|   **Presupuesto/sem**   **Conversiones**   **CPA**           **Lift vs base**                                                                                                      |
|   --------------------- ------------------ ----------------- ------------------                                                                                                    |
|   \$200.000             892                \$224             -5.8%                                                                                                                 |
|                                                                                                                                                                                    |
|   \$247.000 (actual)    1.048              \$236             baseline                                                                                                              |
|                                                                                                                                                                                    |
|   \$300.000             1.175              \$255             +12.1%                                                                                                                |
|                                                                                                                                                                                    |
|   \$400.000             1.380              \$290             +31.7%                                                                                                                |
|                                                                                                                                                                                    |
|   \$500.000             1.540              \$325             +46.9%                                                                                                                |
|   -----------------------------------------------------------------------------                                                                                                    |
|                                                                                                                                                                                    |
| *Observación: el CPA aumenta con el presupuesto (rendimientos decrecientes esperados). De \$400k a \$500k el lift marginal se desacelera: señal de saturación global del sistema.* |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## Paso 4.5: Generación de reportes

Se produce un reporte HTML interactivo listo para presentar al cliente.

+-----------------------------------------------------------------------+
| \# Reporte HTML con todos los escenarios                              |
|                                                                       |
| opt_fixed.output_budget_optimization_summary(                         |
|                                                                       |
| filepath=\'./modaviva_optimization_report.html\'                      |
|                                                                       |
| )                                                                     |
|                                                                       |
| \# Exportar resultados a CSV para dashboard                           |
|                                                                       |
| opt_fixed.get_optimization_results().to_csv(                          |
|                                                                       |
| \'modaviva_optimization_results.csv\'                                 |
|                                                                       |
| )                                                                     |
+-----------------------------------------------------------------------+

# Fase 5. Aplicación gradual y validación

Duración: 4-6 semanas

Nunca se aplican las recomendaciones al 100% de golpe. Esta fase implementa un rollout gradual con validación empírica mediante tests de incrementalidad.

## Paso 5.1: Plan de rollout en 3 etapas

  -----------------------------------------------------------------------------------------
  **Etapa**     **% del cambio**   **Duración**        **Criterio para avanzar**
  ------------- ------------------ ------------------- ------------------------------------
  **Etapa 1**   20%                2-3 semanas         Desviación real vs predicha \< 15%

  **Etapa 2**   50%                2-4 semanas         ROAS global ≥ baseline

  **Etapa 3**   100%               Operación normal    Monitoreo mensual continuo
  -----------------------------------------------------------------------------------------

+---------------------------------------------------------------------------------------------+
| **📌 Aplicación gradual en ModaViva**                                                       |
|                                                                                             |
| Cambio total recomendado para meta_awareness: -\$4.000/sem (con restricciones). Aplicación: |
|                                                                                             |
| -   Etapa 1 (semanas 1-3): reducir \$800/sem → nuevo gasto \$41.200                         |
|                                                                                             |
| -   Etapa 2 (semanas 4-7): reducir \$2.000/sem adicionales → nuevo gasto \$39.200           |
|                                                                                             |
| -   Etapa 3 (semanas 8+): reducir hasta \$4.000 totales → nuevo gasto \$38.000              |
+---------------------------------------------------------------------------------------------+

## Paso 5.2: Test de incrementalidad paralelo

Durante la Etapa 1 se corre un Meta Conversion Lift para validar externamente las predicciones del modelo.

+---------------------------------------------------------------------------------------------+
| **📌 Test de validación en ModaViva**                                                       |
|                                                                                             |
| Configuración del Meta Conversion Lift:                                                     |
|                                                                                             |
| -   Audiencia: usuarios en Bogotá y Medellín                                                |
|                                                                                             |
| -   División: 60% test (ve campañas de awareness) / 40% control (no ve)                     |
|                                                                                             |
| -   Duración: 3 semanas                                                                     |
|                                                                                             |
| -   KPI medido: compras completadas                                                         |
|                                                                                             |
| **Resultado del test:**                                                                     |
|                                                                                             |
| -   Lift incremental medido: 8.3%                                                           |
|                                                                                             |
| -   ROI implícito: 1.62x                                                                    |
|                                                                                             |
| -   Intervalo de confianza 90%: 1.45x - 1.81x                                               |
|                                                                                             |
| **✅ Meridian estimó 1.54x-1.58x, dentro del IC del test (1.45-1.81). Validación exitosa.** |
+---------------------------------------------------------------------------------------------+

## Paso 5.3: Métricas de seguimiento semanal

Se trackean los indicadores clave durante todo el rollout para detectar desviaciones tempranas.

+------------------------------------------------------------------------------------+
| \# Dashboard de seguimiento semanal                                                |
|                                                                                    |
| def weekly_tracking(df_actual, predictions):                                       |
|                                                                                    |
| metrics = {                                                                        |
|                                                                                    |
| \'predicted_conversions\': predictions\[\'conversions\'\].sum(),                   |
|                                                                                    |
| \'actual_conversions\': df_actual\[\'conversions\'\].sum(),                        |
|                                                                                    |
| \'deviation_pct\': (df_actual\[\'conversions\'\].sum() -                           |
|                                                                                    |
| predictions\[\'conversions\'\].sum()) /                                            |
|                                                                                    |
| predictions\[\'conversions\'\].sum() \* 100,                                       |
|                                                                                    |
| \'roas_current\': df_actual\[\'revenue\'\].sum() / df_actual\[\'spend\'\].sum(),   |
|                                                                                    |
| \'cpa_current\': df_actual\[\'spend\'\].sum() / df_actual\[\'conversions\'\].sum() |
|                                                                                    |
| }                                                                                  |
|                                                                                    |
| return metrics                                                                     |
+------------------------------------------------------------------------------------+

## Paso 5.4: Criterios de rollback

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Cuándo revertir cambios**                                                                                                                                                                                                                                                                                                                                                        |
|                                                                                                                                                                                                                                                                                                                                                                                    |
| Si en cualquier momento la desviación del KPI real vs predicho supera el 25%, o si el ROAS global cae más del 15% respecto al baseline pre-Meridian, se revierte la reasignación a la distribución original y se investiga. Causas posibles: priors mal calibrados, variable de control omitida, cambio estructural del mercado. No se avanza a la siguiente etapa hasta resolver. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# Fase 6. Ciclo operativo continuo

Duración: permanente (desde mes 4 en adelante)

Una vez validado el modelo y aplicadas las recomendaciones, entra en operación continua con un ritmo definido de actividades mensuales, trimestrales y semestrales.

## Paso 6.1: Actividades mensuales

-   Actualización automática de datos desde Windsor (cron job semanal)

-   Generación automatizada de reporte de seguimiento con KPI real vs predicho

-   Revisión de desviaciones y alertas si superan umbrales críticos

-   Reunión mensual de resultados con stakeholders del cliente

-   Ajustes menores de asignación si hay señales claras

## Paso 6.2: Actividades trimestrales

-   Reentrenamiento completo del modelo con los datos más recientes

-   Recalibración de priors si hay nuevos tests de incrementalidad

-   Revisión de la clasificación de subcanales (¿apareció alguno nuevo?)

-   Actualización del dashboard ejecutivo

-   Presentación de resultados trimestrales al cliente

## Paso 6.3: Actividades semestrales

-   Nuevo test de incrementalidad (Conversion Lift o Geo Lift)

-   Comparación de predicciones Meridian vs realidad observada

-   Ajustes de especificación del modelo si es necesario

-   Revisión estratégica del portafolio de canales

+-----------------------------------------------------------------------+
| **📌 Rutina operativa en ModaViva post-implementación**               |
|                                                                       |
| **Calendario anual:**                                                 |
|                                                                       |
| -   Cada semana: extracción automática de datos (lunes 6am)           |
|                                                                       |
| -   Cada mes: reporte de seguimiento (primer viernes del mes)         |
|                                                                       |
| -   Cada trimestre: reentrenamiento del modelo (fin de trimestre)     |
|                                                                       |
| -   Cada 6 meses: test de incrementalidad de validación               |
|                                                                       |
| -   Cada año: revisión estratégica completa con cliente               |
|                                                                       |
| **Dedicación del equipo:**                                            |
|                                                                       |
| -   Data scientist: 10-15% del tiempo (4-6 horas/semana)              |
|                                                                       |
| -   Media manager: 5% del tiempo (2 horas/semana)                     |
|                                                                       |
| -   Marketing lead: 2 horas al mes en reuniones                       |
+-----------------------------------------------------------------------+

## Paso 6.4: Roles y responsabilidades

  -----------------------------------------------------------------------------------------------------------
  **Rol**                     **Responsabilidad**                                    **Frecuencia**
  --------------------------- ------------------------------------------------------ ------------------------
  Data Scientist / Analista   Pipeline, reentrenamiento, diagnósticos del modelo     Semanal / trimestral

  Media Manager               Aplicar reasignaciones en plataformas; tests de lift   Semanal

  Marketing Lead              Aprobación de recomendaciones estratégicas             Mensual

  Stakeholder ejecutivo       Revisión de ROI del proyecto Meridian                  Trimestral
  -----------------------------------------------------------------------------------------------------------

# Resumen consolidado

**Cronograma completo**

  -----------------------------------------------------------------------------------
  **Fase**      **Duración**      **Entregable principal**
  ------------- ----------------- ---------------------------------------------------
  **Fase 0**    1 semana          Checklist de viabilidad + entorno técnico listo

  **Fase 1**    2-3 semanas       Dataset estructurado con controles

  **Fase 2**    1 semana          Tabla de parámetros empíricos por subcanal

  **Fase 3**    2 semanas         Modelo Meridian entrenado y validado

  **Fase 4**    1 semana          Reportes de optimización con múltiples escenarios

  **Fase 5**    4-6 semanas       Aplicación gradual validada con tests

  **Fase 6**    Continuo          Ciclo operativo en producción
  -----------------------------------------------------------------------------------

**Resultados esperados para un cliente tipo**

+-----------------------------------------------------------------------+
| **📌 ROI proyectado del proyecto Meridian (basado en ModaViva)**      |
|                                                                       |
| **Mejora de conversiones:**                                           |
|                                                                       |
| -   Baseline: 947 conv/sem                                            |
|                                                                       |
| -   Post-Meridian: 1.021 conv/sem                                     |
|                                                                       |
| -   Lift: +7.8% sin aumento de presupuesto                            |
|                                                                       |
| **Economía del proyecto:**                                            |
|                                                                       |
| -   Costo implementación: \~USD 15.000                                |
|                                                                       |
| -   Lift anualizado: \~USD 40.000 en margen adicional                 |
|                                                                       |
| -   Payback: 5 meses                                                  |
|                                                                       |
| -   ROI del proyecto a 12 meses: 2.6x                                 |
+-----------------------------------------------------------------------+

**Factores críticos de éxito**

1.  Calidad de los datos históricos en Windsor (granularidad, consistencia, cobertura)

2.  Desagregación correcta de campañas en subcanales estratégicos

3.  Inclusión rigurosa de variables de control (holidays, GQV, precio, promociones)

4.  Disciplina en la aplicación gradual con validación empírica

5.  Reentrenamiento periódico para mantener el modelo actualizado

6.  Compromiso del equipo de marketing con aplicar las recomendaciones

7.  Comunicación clara con stakeholders sobre limitaciones e intervalos de credibilidad

**Próximos pasos para arrancar**

8.  Validar el caso de uso con el checklist de la Fase 0

9.  Seleccionar cliente piloto (si eres agencia)

10. Asignar responsable técnico y aprobar cronograma

11. Solicitar accesos a Windsor y configurar entorno Colab Pro+

12. Arrancar la Fase 1 con la extracción completa de datos

13. Establecer cadencia de reuniones de seguimiento semanal/quincenal

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Cierre**                                                                                                                                                                                                                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                                                                                                                                                                             |
| Este plan cubre las 7 fases de una implementación Meridian con Ruta 1 (priors informados), ilustrada con el ejemplo de ModaViva. Cada paso tiene su output esperado y métrica de validación, lo que permite avanzar con confianza y detectar problemas temprano. Al completar las 7 fases tendrás un sistema de optimización de presupuesto robusto, validado y operando en producción con un ROI positivo. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
