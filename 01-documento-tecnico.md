**DOCUMENTO TÉCNICO**

**Meridian Ruta 1**

Optimización con priors informados

*Marco conceptual, matemática, estimación de parámetros y aplicación práctica*

Caso aplicado: Meta Ads + Google Ads

*Pipeline de datos vía Windsor.ai*

Documento preparado en abril de 2026

# Contenido

**Parte 1.** Marco conceptual del modelo Meridian

**Parte 2.** Las funciones matemáticas al detalle

**Parte 3.** Estimación de parámetros desde datos históricos

**Parte 4.** Aplicación al caso Meta + Google con Windsor

**Parte 5.** Consideraciones prácticas y riesgos

**Parte 6.** Variables necesarias: checklist completo

# Parte 1. Marco conceptual del modelo Meridian

Antes de implementar priors informados, es imprescindible entender qué fórmula matemática está detrás de Meridian y qué representa cada uno de sus componentes. En esta sección desglosamos la ecuación central, término por término, con ejemplos aplicados.

## 1.1 La ecuación central de Meridian

El modelo Meridian predice la variable de respuesta (KPI) como una suma de contribuciones provenientes de distintas fuentes: una base estructural, un componente temporal, los efectos de cada canal de marketing transformados por adstock y saturación, variables de control y un error aleatorio.

  ------------------------------------------------------------------------------------------
  *y(g,t) = τ(g) + μ(t) + Σ β(m,g) · Hill(Adstock(x(m,g,t))) + Σ γ(c) · z(c,g,t) + ε(g,t)*

  ------------------------------------------------------------------------------------------

Cada término tiene un rol específico. Los subíndices indican sobre qué dimensión varía cada variable:

-   **g:** región geográfica (Bogotá, Medellín, Cali, etc.). Va de 1 a G.

-   **t:** período temporal, típicamente semana. Va de 1 a T.

-   **m:** canal de marketing (Meta Prospecting, Google Search, etc.). Va de 1 a M.

-   **c:** variable de control (precio, estacionalidad, etc.). Va de 1 a C.

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Ejemplo de dimensionalidad**                                                                                                                                                                                                                                |
|                                                                                                                                                                                                                                                               |
| Si tu dataset tiene 3 ciudades, 104 semanas de histórico, 5 subcanales de marketing y 2 variables de control, entonces G=3, T=104, M=5 y C=2. Meridian estima parámetros para cada combinación relevante, aprovechando la estructura jerárquica de los datos. |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## 1.2 Término 1: y(g,t) --- la variable dependiente

Es el KPI que quieres explicar y optimizar. Debe ser numérico, continuo y estar agregado al nivel región × semana. En tu caso, podría ser el número de conversiones en Bogotá durante la semana 34, o el revenue total en Medellín durante la semana 17.

**Decisión crítica: ¿revenue o conversiones?**

Esta elección condiciona toda la configuración del modelo mediante el parámetro kpi_type. Modelar revenue conviene cuando el producto tiene precios variables o promociones frecuentes; modelar conversiones es mejor cuando el precio es estable y te interesa maximizar unidades vendidas.

  --------------------------------------------------------------------------
  **Criterio**            **Modelar revenue**     **Modelar conversiones**
  ----------------------- ----------------------- --------------------------
  Precio del producto     Varía en el tiempo      Estable

  Objetivo de negocio     Maximizar ingresos      Maximizar unidades

  kpi_type en Meridian    \'revenue\'             \'non_revenue\'
  --------------------------------------------------------------------------

## 1.3 Término 2: τ(g) --- el intercepto por región

Representa la línea base de ventas de cada región, independiente del marketing. Es lo que esa región vendería si todos los canales publicitarios tuvieran gasto cero. Este término captura el tamaño estructural del mercado, la penetración histórica de la marca y la demanda orgánica natural.

La importancia de este parámetro es grande: sin él, el modelo asumiría que toda diferencia de ventas entre Bogotá y Neiva se debe al marketing, lo cual es absurdo. Bogotá vende más simplemente porque tiene más habitantes, más poder adquisitivo y mayor penetración de marca.

Meridian estima τ(g) como un parámetro libre con un prior normal por defecto. Si no le pasas un prior específico, lo aprende directamente de los datos observados.

## 1.4 Término 3: μ(t) --- el efecto temporal

Captura cómo evolucionan las ventas en el tiempo por razones no relacionadas con marketing. Incluye tendencias de crecimiento del negocio, estacionalidad general y efectos externos como pandemias o eventos macroeconómicos.

Meridian puede modelar μ(t) de varias formas. La más común son splines suaves con nodos (knots) distribuidos en el tiempo, típicamente uno cada 10-12 semanas. Esto permite capturar patrones flexibles sin sobreajustar.

+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Diferencia con variables de control**                                                                                                                                                                                                                                                                                   |
|                                                                                                                                                                                                                                                                                                                           |
| μ(t) captura efectos temporales que no sabes explicar con variables específicas. Si conoces la causa de un efecto temporal (por ejemplo, Black Friday), es mejor modelarlo como una variable de control explícita que dejarlo en μ(t). Entre más efectos temporales puedas explicar con controles, más limpio queda μ(t). |
+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## 1.5 Término 4: el efecto de los canales (el corazón del modelo)

Esta es la parte más importante del modelo y la que más nos importa para la optimización:

  -----------------------------------------------------------------------
  *Σ β(m,g) · Hill(Adstock(x(m,g,t)))*

  -----------------------------------------------------------------------

La suma Σ significa que el efecto total del marketing es la suma de contribuciones de cada canal. Si tienes 5 canales, el modelo calcula el efecto de cada uno por separado y luego los agrega. Analicemos cada componente dentro de la suma.

### 1.5.1 x(m,g,t) --- el gasto bruto

Es simplemente cuánto invertiste en el canal m, en la región g, durante la semana t. Por ejemplo, x(Meta, Bogotá, semana 34) = 8.500.000 COP. Meridian también admite impresiones o alcance en lugar de gasto; para YouTube es común usar reach y frequency en vez de spend.

### 1.5.2 Adstock(·) --- la transformación de persistencia

Esta función convierte el gasto observable en el gasto efectivo, considerando que los anuncios tienen efectos persistentes en el tiempo. No es lo mismo el impacto del dinero gastado hoy que el dinero gastado hace 3 semanas: ambos contribuyen, pero con pesos distintos.

  ----------------------------------------------------------------------------
  *Adstock(x_t) = x_t + α · x\_(t-1) + α² · x\_(t-2) + α³ · x\_(t-3) + \...*

  ----------------------------------------------------------------------------

Esto es una progresión geométrica. El gasto de hace 1 semana contribuye ponderado por α, el de hace 2 semanas por α², y así sucesivamente. El parámetro α (alpha) está entre 0 y 1 y controla qué tan rápido decae el efecto.

**Ejemplo numérico: Meta con α = 0.5 en Bogotá**

Supongamos los siguientes gastos semanales en Meta:

-   Semana 32: \$10.000

-   Semana 33: \$8.000

-   Semana 34: \$12.000

El adstock efectivo en la semana 34 sería:

  -----------------------------------------------------------------------
  *Adstock(34) = 12.000 + 0,5×8.000 + 0,25×10.000 + \... ≈ 18.500*

  -----------------------------------------------------------------------

Aunque el gasto bruto de la semana es de \$12.000, el efecto publicitario efectivo es mayor porque todavía reverberan inversiones de semanas previas.

**Rangos típicos de α por canal**

  -----------------------------------------------------------------------------------------------
  **Canal**                **α típico**    **Interpretación**
  ------------------------ --------------- ------------------------------------------------------
  Google Search            0.1 - 0.3       Respuesta casi inmediata; el efecto se disipa rápido

  Google Performance Max   0.3 - 0.5       Efecto medio, dura 2-4 semanas

  YouTube                  0.6 - 0.8       Efecto largo; construye marca a mediano plazo

  Meta Prospecting         0.3 - 0.5       Efecto medio sobre audiencias nuevas

  Meta Retargeting         0.2 - 0.4       Efecto corto sobre usuarios con intención
  -----------------------------------------------------------------------------------------------

### 1.5.3 Hill(·) --- la función de saturación

Mientras adstock modela la persistencia temporal, Hill modela los rendimientos decrecientes. Captura una realidad fundamental del marketing: duplicar la inversión no duplica los resultados. Hay un punto donde cada peso adicional rinde menos que el anterior.

  -----------------------------------------------------------------------
  *Hill(a) = a\^s / (a\^s + ec\^s)*

  -----------------------------------------------------------------------

La función Hill toma el valor del adstock (ya transformado) y devuelve un número entre 0 y 1 que representa el nivel de efectividad alcanzado. Tiene dos parámetros clave:

-   **ec (half-saturation point):** el nivel de gasto donde Hill alcanza 0.5, es decir, el 50% del efecto máximo posible. Es específico del canal y de la escala del negocio.

-   **s (slope):** qué tan abrupta es la curva. Con s=1 la curva es suave y cóncava; con s\>1 forma una S (arranque lento, luego crecimiento rápido, luego saturación).

**Propiedades matemáticas de Hill**

-   Hill(0) = 0 (sin gasto, sin efecto)

-   Hill(ec) = 0.5 (en el punto de media saturación)

-   Hill(∞) → 1 (saturación total, no puedes extraer más)

Gráficamente, la curva Hill típica comienza en cero, crece rápido en la zona de bajo gasto, pasa por 0.5 en ec y se aplana asintóticamente hacia 1 a medida que el gasto se hace muy grande.

### 1.5.4 β(m,g) --- el coeficiente de escala

Hill devuelve un número entre 0 y 1, pero tus ventas son números grandes (cientos, miles, millones). β es el multiplicador que convierte esa efectividad en unidades reales del KPI.

El subíndice g indica que Meridian permite que cada región tenga su propio β. Bogotá puede responder más fuerte a Meta que Pasto por razones estructurales: mayor penetración de smartphones, más uso de redes sociales, demografía distinta.

+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Clave para la Ruta 1: parametrización por ROI**                                                                                                                                                                                                                                                                                                                                                     |
|                                                                                                                                                                                                                                                                                                                                                                                                       |
| Meridian permite parametrizar el modelo directamente en términos de ROI en vez de β mediante el parámetro media_prior_type=\'roi\'. En lugar de poner priors sobre β (que no son interpretables), pones priors sobre el ROI del canal (que sí lo es). Meridian internamente calcula el β correspondiente. Esta es la puerta de entrada para implementar la Ruta 1 con tus ROIs históricos observados. |
+-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## 1.6 Término 5: Σ γ(c) · z(c,g,t) --- las variables de control

Este término suma los efectos de variables externas que no son marketing pero afectan las ventas. Son críticas para no atribuirle a los canales efectos que en realidad provienen de factores ambientales o estructurales.

**Ejemplos típicos de z(c,g,t)**

-   Precio promedio del producto esa semana en esa región

-   Flag binario de días festivos o Black Friday

-   Google Query Volume (demanda orgánica de búsquedas)

-   Temperatura promedio (relevante si vendes estacionales)

-   Tipo de cambio USD/COP (si importas productos)

-   Inversión en canales que no modelas (orgánico, email) para descontar su efecto

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Por qué son críticas**                                                                                                                                                                                                                                                                                                                                                 |
|                                                                                                                                                                                                                                                                                                                                                                          |
| Si en diciembre aumentas el gasto de Meta de \$10k a \$20k semanales, y las ventas suben de 100 a 180, el modelo sin controles concluye que Meta duplicó su impacto. La realidad puede ser que diciembre naturalmente vende 60% más por estacionalidad, y Meta solo aportó el 20% restante. Sin un control de estacionalidad, el modelo sobreestima Meta de forma grave. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## 1.7 Término 6: ε(g,t) --- el error

Es todo aquello que el modelo no puede explicar. Meridian asume que ε(g,t) es ruido aleatorio con distribución normal y media cero. Incluye variaciones aleatorias genuinas, efectos de variables omitidas y ruido de medición.

Si ε(g,t) es grande, significa que tu modelo explica poco de la variabilidad real. Es una señal para añadir más controles, revisar la calidad de los datos o aceptar que hay incertidumbre estructural que no se puede reducir.

## 1.8 Los 4 parámetros críticos por canal: visión integrada

De toda la maquinaria anterior, estos son los cuatro parámetros que vas a informar con priors estrechos en la Ruta 1:

  ------------------------------------------------------------------------------------------------------------------------------------
  **Parámetro**   **Dónde aparece**                                      **Qué controla**                           **Rango típico**
  --------------- ------------------------------------------------------ ------------------------------------------ ------------------
  **ROI(m)**      Implícito vía β cuando usas media_prior_type=\'roi\'   Cuánto KPI genera cada peso invertido      0.5 a 6.0

  **α(m)**        En Adstock(·)                                          Qué tan largo es el efecto del canal       0 a 1

  **ec(m)**       En Hill(·)                                             Dónde está la media saturación del canal   Según escala

  **s(m)**        En Hill(·)                                             Qué tan abrupta es la saturación           0.5 a 3.0
  ------------------------------------------------------------------------------------------------------------------------------------

## 1.9 Ejemplo integrado: predecir conversiones de Meta en Bogotá

Para cerrar la Parte 1, veamos cómo se encadenan todos los términos en un caso concreto. Objetivo: predecir las conversiones de Meta en Bogotá durante la semana 34.

**Datos conocidos**

-   x(Meta, Bogotá, 34) = \$12.000 (gasto esta semana)

-   x(Meta, Bogotá, 33) = \$8.000 (semana anterior)

-   x(Meta, Bogotá, 32) = \$10.000 (hace dos semanas)

**Priors informados que le pasamos (Ruta 1)**

-   ROI_Meta = 2.3

-   α_Meta = 0.4

-   ec_Meta = 15.000

-   s_Meta = 1.5

**Paso 1 --- Adstock**

  -----------------------------------------------------------------------
  *Adstock = 12.000 + 0,4·8.000 + 0,16·10.000 ≈ 17.000*

  -----------------------------------------------------------------------

**Paso 2 --- Hill**

  -----------------------------------------------------------------------
  *Hill(17.000) = 17.000\^1.5 / (17.000\^1.5 + 15.000\^1.5) ≈ 0,547*

  -----------------------------------------------------------------------

Meta está alcanzando el 54,7% de su efectividad máxima en Bogotá esa semana.

**Paso 3 --- Escalamiento β a través del ROI**

Con ROI = 2.3, \$12.000 deberían generar aproximadamente \$27.600 en revenue (o las conversiones equivalentes). El β se ajusta internamente para que la combinación β × 0.547 produzca esa cifra.

**Paso 4 --- Suma con los otros términos**

  ---------------------------------------------------------------------------------
  *y = τ(Bogotá) + μ(34) + β_Meta·0,547 + β_Google·Hill_Google(\...) + Σ γ·z + ε*

  ---------------------------------------------------------------------------------

Meridian hace este cálculo simultáneamente para todas las combinaciones de región × semana y ajusta los parámetros (dentro de los priors que le pasaste) para minimizar el error. En la Ruta 1, como los priors son muy estrechos, el resultado será muy cercano a tus estimaciones empíricas.

# Parte 2. Las funciones matemáticas al detalle

Esta sección profundiza en las dos funciones clave (Adstock y Hill) con todas sus variantes, propiedades matemáticas y comportamiento en casos borde.

## 2.1 La función Adstock geométrica

El adstock formaliza la idea de que la publicidad tiene efectos persistentes. Fue propuesto por Simon Broadbent en 1979 y es un pilar del MMM moderno. La versión geométrica, que usa Meridian por defecto, es la más simple y robusta.

### 2.1.1 Forma recursiva

  -----------------------------------------------------------------------
  *A_t = x_t + α · A\_(t-1)*

  -----------------------------------------------------------------------

Esta expresión dice: el adstock de hoy es el gasto nuevo de hoy más una fracción α del adstock de ayer. Es elegante y fácil de calcular computacionalmente.

### 2.1.2 Forma expandida equivalente

  -----------------------------------------------------------------------
  *A_t = Σ (k=0 a L) α\^k · x\_(t-k)*

  -----------------------------------------------------------------------

Expandiendo la recursión, el adstock es una suma ponderada del gasto actual y todos los gastos pasados, donde el peso de cada período decrece geométricamente con potencias de α.

### 2.1.3 Visualización del decaimiento

Con un gasto único de \$1.000 solo en la semana 1 y cero después, así se ve el adstock resultante con distintos valores de α:

  -----------------------------------------------------------------------------
  **Semana**    **1**   **2**   **3**   **4**   **5**   **6**   **7**   **8**
  ------------- ------- ------- ------- ------- ------- ------- ------- -------
  **α = 0.2**   1000    200     40      8       1.6     0.3     0.06    0.01

  **α = 0.5**   1000    500     250     125     63      31      16      8

  **α = 0.8**   1000    800     640     512     410     328     262     210
  -----------------------------------------------------------------------------

### 2.1.4 La vida media (half-life) del canal

Un concepto derivado útil para explicar adstock a audiencias no técnicas: cuántas semanas tarda el efecto en reducirse a la mitad.

  -----------------------------------------------------------------------
  *half-life = log(0,5) / log(α)*

  -----------------------------------------------------------------------

  ---------------------------------------------------------------------------------------------
  **α**             **Vida media (semanas)**   **Interpretación**
  ----------------- -------------------------- ------------------------------------------------
  0.5               1.0                        Efecto corto: en una semana se pierde la mitad

  0.7               ≈ 2.0                      Efecto medio: la mitad persiste dos semanas

  0.85              ≈ 4.3                      Efecto largo: un mes para perder la mitad

  0.9               ≈ 6.6                      Efecto muy largo: típico de branding y TV
  ---------------------------------------------------------------------------------------------

### 2.1.5 El parámetro max_lag

Aunque matemáticamente el adstock geométrico nunca llega exactamente a cero, en la práctica Meridian corta la recursión después de L períodos mediante max_lag. Los valores típicos son:

-   max_lag = 8 (default): considera el efecto acumulado de las últimas 8 semanas

-   max_lag = 13: más conservador, para canales de branding

-   max_lag = 4: para canales de respuesta muy inmediata

Un max_lag mayor requiere más datos históricos para estimarse de forma estable. Si tu histórico es corto, mantén max_lag bajo.

## 2.2 La función Hill de saturación

La función Hill tiene su origen en bioquímica (cinética enzimática) pero modela bien el comportamiento de saturación publicitaria.

### 2.2.1 Forma matemática

  -----------------------------------------------------------------------
  *Hill(a) = a\^s / (a\^s + ec\^s)*

  -----------------------------------------------------------------------

### 2.2.2 Comportamiento del parámetro slope

El slope s determina la forma cualitativa de la curva:

  ------------------------------------------------------------------------------------------------------------------------------------------------
  **Slope s**       **Forma**                     **Interpretación de marketing**
  ----------------- ----------------------------- ------------------------------------------------------------------------------------------------
  **s \< 1**        Cóncava desde el origen       Rendimientos decrecientes muy rápidos. El primer dólar rinde mucho, el segundo bastante menos.

  **s = 1**         Michaelis-Menten clásica      Saturación suave y estándar. Punto de referencia en muchos modelos.

  **1 \< s \< 2**   Forma de S suave              Hay un umbral mínimo de gasto antes de ver efecto fuerte. Común en digital.

  **s \> 2**        S pronunciada                 Umbral muy marcado. El canal prácticamente no funciona por debajo de cierto gasto.
  ------------------------------------------------------------------------------------------------------------------------------------------------

### 2.2.3 El ROI marginal implícito en Hill

Una propiedad importante: la derivada de Hill respecto al gasto disminuye monotónicamente una vez pasado el punto de inflexión. Esto significa que cada peso adicional rinde menos que el anterior. Matemáticamente:

  -----------------------------------------------------------------------
  *dHill/da = s · ec\^s · a\^(s-1) / (a\^s + ec\^s)\^2*

  -----------------------------------------------------------------------

Esta es la base matemática para decidir cuándo escalar un canal: mientras el ROI marginal esté por encima de tu umbral de rentabilidad, sigue invirtiendo. Cuando cae por debajo, deja de escalar o mueve presupuesto a otro canal.

## 2.3 La composición completa: Adstock + Hill + ROI

Poniendo todo junto, la contribución de un canal al KPI en Meridian es:

  -----------------------------------------------------------------------
  *efecto = ROI · Hill(Adstock(x)) · factor_escala*

  -----------------------------------------------------------------------

Esta cadena de transformaciones es lo que el optimizador navega. Cada canal tiene su propia curva definida por sus cuatro parámetros (ROI, α, ec, s), y el optimizador busca la combinación de gastos que maximiza la suma de efectos sujeto a la restricción de presupuesto total.

## 2.4 Problema de optimización que resuelve Meridian

Formalmente, dado un presupuesto total B y los parámetros estimados (o informados), Meridian resuelve:

  -----------------------------------------------------------------------
  *max Σ efecto_m(x_m) sujeto a Σ x_m = B, x_m ≥ 0*

  -----------------------------------------------------------------------

Este no es un problema lineal (sería un simplex clásico); es un problema de optimización no lineal con restricciones, por la forma cóncava de las curvas Hill. Meridian usa métodos numéricos para resolverlo, típicamente descenso de gradiente proyectado o variantes.

# Parte 3. Estimación de parámetros desde datos históricos

Para implementar la Ruta 1 necesitas calcular empíricamente los cuatro parámetros (ROI, α, ec, s) por canal usando tus datos históricos. Esta sección explica el procedimiento completo.

## 3.1 Estimación del ROI

### 3.1.1 ROAS observado: el punto de partida

El primer cálculo es directo a partir de tus datos agregados:

  -----------------------------------------------------------------------
  *ROAS_observado = Revenue_total_canal / Gasto_total_canal*

  -----------------------------------------------------------------------

Por ejemplo, si en dos años gastaste \$500M en Meta y el revenue atribuido fue \$1.500M, tu ROAS observado es 3.0.

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Advertencia crítica: ROAS ≠ ROI incremental**                                                                                                                                                                                                  |
|                                                                                                                                                                                                                                                  |
| El ROAS observado está inflado por ventas que habrían ocurrido sin el anuncio (demanda orgánica capturada), canibalización de otros canales y atribución que premia al último click. Para Meridian necesitas ROI incremental, no ROAS observado. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

### 3.1.2 Factores de ajuste empíricos por canal

En ausencia de tests de incrementalidad propios, la práctica estándar de la industria aplica factores de ajuste para convertir ROAS en ROI incremental:

  -----------------------------------------------------------------------
  *ROI_incremental ≈ ROAS_observado × factor_ajuste*

  -----------------------------------------------------------------------

  -------------------------------------------------------------------------------------------------
  **Canal**                **Factor**      **Razón**
  ------------------------ --------------- --------------------------------------------------------
  Google Search Brand      0.3 - 0.5       Captura mucha demanda orgánica que habría llegado sola

  Google Search Genérico   0.7 - 0.9       Más incremental: atrae intención nueva

  Google Performance Max   0.6 - 0.8       Mezcla de intención y prospecting

  YouTube                  0.6 - 0.8       Construcción de marca, incremental por naturaleza

  Meta Prospecting         0.7 - 0.9       Llega a usuarios nuevos

  Meta Retargeting         0.3 - 0.5       Muchas conversiones habrían ocurrido igual
  -------------------------------------------------------------------------------------------------

Si tienes resultados de tests de incrementalidad propios (Conversion Lift, Geo Lift Test), reemplaza estos factores por tus mediciones reales; son mucho más precisas que los benchmarks genéricos.

## 3.2 Estimación del parámetro adstock α

La estrategia más común y robusta es un grid search: probar distintos valores de α y elegir el que maximiza la correlación entre el gasto transformado y las conversiones observadas.

### 3.2.1 Pseudocódigo del algoritmo

+-----------------------------------------------------------------------+
| import numpy as np                                                    |
|                                                                       |
| from scipy.stats import pearsonr                                      |
|                                                                       |
| def apply_adstock(spend_series, alpha, max_lag=8):                    |
|                                                                       |
| adstocked = np.zeros(len(spend_series))                               |
|                                                                       |
| for t in range(len(spend_series)):                                    |
|                                                                       |
| for l in range(min(t + 1, max_lag)):                                  |
|                                                                       |
| adstocked\[t\] += (alpha \*\* l) \* spend_series\[t - l\]             |
|                                                                       |
| return adstocked                                                      |
|                                                                       |
| def estimate_alpha(spend, conversions, max_lag=8):                    |
|                                                                       |
| best_alpha, best_corr = None, -1                                      |
|                                                                       |
| for alpha in np.arange(0.05, 0.95, 0.05):                             |
|                                                                       |
| adstocked = apply_adstock(spend.values, alpha, max_lag)               |
|                                                                       |
| corr, \_ = pearsonr(adstocked, conversions.values)                    |
|                                                                       |
| if corr \> best_corr:                                                 |
|                                                                       |
| best_corr = corr                                                      |
|                                                                       |
| best_alpha = alpha                                                    |
|                                                                       |
| return best_alpha, best_corr                                          |
+-----------------------------------------------------------------------+

### 3.2.2 Consideraciones prácticas

-   Este método puede fallar si el gasto y las conversiones están dominados por una tendencia común; en ese caso, primero remueve la tendencia (detrending).

-   Para canales con mucho ruido, conviene suavizar ambas series con una media móvil de 2 semanas antes del cálculo.

-   Si el mejor α resultante es 0.05 o 0.95 (extremos del grid), investiga: probablemente hay un problema de datos.

## 3.3 Estimación de los parámetros de saturación (ec, s)

Para estimar ec y s de forma empírica se ajusta la función Hill a pares observados de (gasto semanal, conversiones semanales). Se usa optimización no lineal vía scipy.optimize.curve_fit.

### 3.3.1 Implementación

+-----------------------------------------------------------------------+
| from scipy.optimize import curve_fit                                  |
|                                                                       |
| def hill_function(x, ec, slope, vmax):                                |
|                                                                       |
| return vmax \* (x \*\* slope) / (x \*\* slope + ec \*\* slope)        |
|                                                                       |
| def estimate_saturation(spend_series, conv_series):                   |
|                                                                       |
| ec_init = spend_series.median()                                       |
|                                                                       |
| slope_init = 1.0                                                      |
|                                                                       |
| vmax_init = conv_series.max() \* 1.5                                  |
|                                                                       |
| popt, \_ = curve_fit(                                                 |
|                                                                       |
| hill_function,                                                        |
|                                                                       |
| spend_series.values,                                                  |
|                                                                       |
| conv_series.values,                                                   |
|                                                                       |
| p0=\[ec_init, slope_init, vmax_init\],                                |
|                                                                       |
| bounds=(\[spend_series.min(), 0.1, 0\],                               |
|                                                                       |
| \[spend_series.max() \* 10, 5, vmax_init \* 10\]),                    |
|                                                                       |
| maxfev=5000                                                           |
|                                                                       |
| )                                                                     |
|                                                                       |
| return {\'ec\': popt\[0\], \'slope\': popt\[1\], \'vmax\': popt\[2\]} |
+-----------------------------------------------------------------------+

### 3.3.2 Problema común: falta de variación en el gasto

+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Si el gasto histórico es muy estable, no puedes estimar saturación**                                                                                                                                                                                                                                                   |
|                                                                                                                                                                                                                                                                                                                          |
| La curva Hill se estima bien solo si hay suficiente variación en los niveles de gasto. Si siempre gastaste entre \$8k y \$10k semanales en Meta, el modelo no puede saber qué pasa cuando gastas \$30k o \$50k. En estos casos, usa benchmarks de la industria como aproximación inicial y valida con tests controlados. |
+--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## 3.4 Conversión de estimaciones empíricas en priors de Meridian

Ya tienes tus estimaciones (ROI, α, ec, s) por canal. Ahora debes convertirlas en distribuciones de probabilidad (priors) que Meridian pueda consumir. Cada parámetro usa una familia de distribución apropiada a su rango:

  -------------------------------------------------------------------------------------------
  **Parámetro**     **Distribución**   **Razón de la elección**
  ----------------- ------------------ ------------------------------------------------------
  ROI               Log-Normal         Es siempre positivo, con cola derecha larga

  α (adstock)       Beta               Está acotado en \[0, 1\]; Beta es la familia natural

  ec                Log-Normal         Gasto siempre positivo, con escala exponencial

  slope             Log-Normal         Siempre positivo, típicamente entre 0.5 y 3
  -------------------------------------------------------------------------------------------

### 3.4.1 Código de construcción de priors

+-----------------------------------------------------------------------+
| import tensorflow_probability as tfp                                  |
|                                                                       |
| import numpy as np                                                    |
|                                                                       |
| tfd = tfp.distributions                                               |
|                                                                       |
| \# ROI como log-normal estrecha                                       |
|                                                                       |
| def roi_prior(roi_mean, uncertainty=0.10):                            |
|                                                                       |
| return tfd.LogNormal(                                                 |
|                                                                       |
| loc=np.log(roi_mean) - (uncertainty \*\* 2) / 2,                      |
|                                                                       |
| scale=uncertainty                                                     |
|                                                                       |
| )                                                                     |
|                                                                       |
| \# Adstock α como Beta concentrada                                    |
|                                                                       |
| def adstock_prior(alpha_mean, strength=50):                           |
|                                                                       |
| a = alpha_mean \* strength                                            |
|                                                                       |
| b = (1 - alpha_mean) \* strength                                      |
|                                                                       |
| return tfd.Beta(concentration1=a, concentration0=b)                   |
|                                                                       |
| \# Half-saturation y slope como log-normal                            |
|                                                                       |
| def ec_prior(ec_mean, uncertainty=0.15):                              |
|                                                                       |
| return tfd.LogNormal(                                                 |
|                                                                       |
| loc=np.log(ec_mean) - (uncertainty \*\* 2) / 2,                       |
|                                                                       |
| scale=uncertainty                                                     |
|                                                                       |
| )                                                                     |
+-----------------------------------------------------------------------+

### 3.4.2 Nivel de confianza: el parámetro uncertainty

Controla qué tan estrecho es el prior. Entre menor el uncertainty, más fuerte es tu señal previa y menos flexibilidad tiene el modelo para desviarse.

  -------------------------------------------------------------------------------------------------------------
  **Uncertainty**   **Significado**            **Cuándo usarlo**
  ----------------- -------------------------- ----------------------------------------------------------------
  **0.05 - 0.10**   Muy alta confianza         Tienes tests de incrementalidad propios o muchos datos sólidos

  **0.15 - 0.25**   Confianza media            Estimación basada en datos históricos sin validación externa

  **0.30 - 0.50**   Confianza baja             Usas benchmarks de industria o estimación gruesa
  -------------------------------------------------------------------------------------------------------------

# Parte 4. Aplicación al caso Meta + Google con Windsor

Esta sección aterriza todo lo anterior en tu caso específico: campañas de Meta Ads y Google Ads con datos extraídos vía Windsor.ai. Vas a ver el flujo completo, desde la extracción hasta el reporte final de optimización.

## 4.1 Arquitectura del pipeline

El flujo de datos completo para tu caso funciona en cinco bloques encadenados:

1.  Windsor.ai extrae datos diarios de Meta y Google Ads por campaña y región.

2.  Un script Python agrega a nivel semanal y categoriza campañas en subcanales.

3.  Se calculan los parámetros empíricos por subcanal (ROI, α, ec, s).

4.  Se construyen los priors informados y se entrena Meridian.

5.  Se corre el optimizador y se genera el reporte HTML con recomendaciones.

## 4.2 Estructura de datos esperada por Meridian

El dataset final que alimenta a Meridian debe tener una fila por combinación región × semana, con columnas separadas por subcanal. Un fragmento mínimo se vería así:

  ------------------------------------------------------------------------------------------------------------------------------
  **date**     **geo**    **gsearch_spend**   **gpmax_spend**   **meta_prosp_spend**   **meta_rtg_spend**   **conv**   **hol**
  ------------ ---------- ------------------- ----------------- ---------------------- -------------------- ---------- ---------
  2024-01-07   Bogotá     3500                2100              2800                   900                  142        0

  2024-01-07   Medellín   1800                1000              1500                   500                  78         0

  2024-01-14   Bogotá     4200                2400              3100                   1100                 165        0
  ------------------------------------------------------------------------------------------------------------------------------

## 4.3 Paso 1: extracción desde Windsor

+-----------------------------------------------------------------------+
| import requests                                                       |
|                                                                       |
| import pandas as pd                                                   |
|                                                                       |
| def fetch_windsor_data(api_key, start_date, end_date):                |
|                                                                       |
| base_url = \'https://connect.windsor.ai/meridian\'                    |
|                                                                       |
| params = {                                                            |
|                                                                       |
| \'api_key\': api_key,                                                 |
|                                                                       |
| \'date_from\': start_date,                                            |
|                                                                       |
| \'date_to\': end_date,                                                |
|                                                                       |
| \'fields\': \'date,source,campaign,region,spend,impressions,          |
|                                                                       |
| clicks,conversions,revenue\',                                         |
|                                                                       |
| \'connector\': \[\'google_ads\', \'facebook\'\]                       |
|                                                                       |
| }                                                                     |
|                                                                       |
| response = requests.get(base_url, params=params)                      |
|                                                                       |
| return pd.DataFrame(response.json()\[\'data\'\])                      |
|                                                                       |
| df_raw = fetch_windsor_data(                                          |
|                                                                       |
| api_key=\'TU_API_KEY\',                                               |
|                                                                       |
| start_date=\'2022-01-01\',                                            |
|                                                                       |
| end_date=\'2024-12-31\'                                               |
|                                                                       |
| )                                                                     |
+-----------------------------------------------------------------------+

## 4.4 Paso 2: categorización de campañas en subcanales

Esto es crítico. En lugar de tratar Meta y Google como dos canales, los desagregas en subcanales con comportamientos muy diferentes. La lógica típica:

+-----------------------------------------------------------------------+
| def categorize_campaign(row):                                         |
|                                                                       |
| source = row\[\'source\'\].lower()                                    |
|                                                                       |
| campaign = row\[\'campaign\'\].lower()                                |
|                                                                       |
| if source == \'google_ads\':                                          |
|                                                                       |
| if \'search\' in campaign and \'brand\' in campaign:                  |
|                                                                       |
| return \'google_search_brand\'                                        |
|                                                                       |
| elif \'search\' in campaign:                                          |
|                                                                       |
| return \'google_search_generic\'                                      |
|                                                                       |
| elif \'pmax\' in campaign or \'performance\' in campaign:             |
|                                                                       |
| return \'google_pmax\'                                                |
|                                                                       |
| elif \'youtube\' in campaign or \'video\' in campaign:                |
|                                                                       |
| return \'youtube\'                                                    |
|                                                                       |
| else:                                                                 |
|                                                                       |
| return \'google_display\'                                             |
|                                                                       |
| elif source == \'facebook\':                                          |
|                                                                       |
| if \'retargeting\' in campaign or \'rmkt\' in campaign:               |
|                                                                       |
| return \'meta_retargeting\'                                           |
|                                                                       |
| elif \'prospecting\' in campaign:                                     |
|                                                                       |
| return \'meta_prospecting\'                                           |
|                                                                       |
| else:                                                                 |
|                                                                       |
| return \'meta_other\'                                                 |
|                                                                       |
| return \'unknown\'                                                    |
|                                                                       |
| df_raw\[\'sub_channel\'\] = df_raw.apply(categorize_campaign, axis=1) |
+-----------------------------------------------------------------------+

## 4.5 Paso 3: pivot y agregación semanal

+-------------------------------------------------------------------------------------------+
| \# Pivotar para tener una columna por subcanal                                            |
|                                                                                           |
| df_pivoted = df_raw.pivot_table(                                                          |
|                                                                                           |
| index=\[\'date\', \'region\'\],                                                           |
|                                                                                           |
| columns=\'sub_channel\',                                                                  |
|                                                                                           |
| values=\[\'spend\', \'impressions\'\],                                                    |
|                                                                                           |
| aggfunc=\'sum\',                                                                          |
|                                                                                           |
| fill_value=0                                                                              |
|                                                                                           |
| )                                                                                         |
|                                                                                           |
| df_pivoted.columns = \[f\'{col\[1\]}\_{col\[0\]}\' for col in df_pivoted.columns\]        |
|                                                                                           |
| df_pivoted = df_pivoted.reset_index()                                                     |
|                                                                                           |
| \# Añadir conversiones                                                                    |
|                                                                                           |
| conv_df = df_raw.groupby(\[\'date\', \'region\'\])\[\'conversions\'\].sum().reset_index() |
|                                                                                           |
| df_final = df_pivoted.merge(conv_df, on=\[\'date\', \'region\'\])                         |
|                                                                                           |
| \# Añadir controles                                                                       |
|                                                                                           |
| df_final\[\'holiday_flag\'\] = df_final\[\'date\'\].isin(HOLIDAYS).astype(int)            |
|                                                                                           |
| df_final\[\'price\'\] = \... \# tu precio promedio por semana                             |
|                                                                                           |
| df_final\[\'gqv\'\] = \... \# Google Query Volume                                         |
|                                                                                           |
| df_final.to_csv(\'meridian_input.csv\', index=False)                                      |
+-------------------------------------------------------------------------------------------+

## 4.6 Paso 4: estimar parámetros empíricos por subcanal

+--------------------------------------------------------------------------------+
| CHANNELS = \[                                                                  |
|                                                                                |
| \'google_search_brand\',                                                       |
|                                                                                |
| \'google_search_generic\',                                                     |
|                                                                                |
| \'google_pmax\',                                                               |
|                                                                                |
| \'youtube\',                                                                   |
|                                                                                |
| \'meta_prospecting\',                                                          |
|                                                                                |
| \'meta_retargeting\'                                                           |
|                                                                                |
| \]                                                                             |
|                                                                                |
| adjustment_factors = {                                                         |
|                                                                                |
| \'google_search_brand\': 0.4,                                                  |
|                                                                                |
| \'google_search_generic\': 0.8,                                                |
|                                                                                |
| \'google_pmax\': 0.7,                                                          |
|                                                                                |
| \'youtube\': 0.7,                                                              |
|                                                                                |
| \'meta_prospecting\': 0.8,                                                     |
|                                                                                |
| \'meta_retargeting\': 0.4                                                      |
|                                                                                |
| }                                                                              |
|                                                                                |
| empirical_params = {}                                                          |
|                                                                                |
| for channel in CHANNELS:                                                       |
|                                                                                |
| spend_col = f\'{channel}\_spend\'                                              |
|                                                                                |
| total_spend = df_final\[spend_col\].sum()                                      |
|                                                                                |
| roas = df_final\[\'revenue_attributed\'\].sum() / total_spend                  |
|                                                                                |
| roi = roas \* adjustment_factors\[channel\]                                    |
|                                                                                |
| alpha, \_ = estimate_alpha(df_final\[spend_col\], df_final\[\'conversions\'\]) |
|                                                                                |
| sat = estimate_saturation(df_final\[spend_col\], df_final\[\'conversions\'\])  |
|                                                                                |
| empirical_params\[channel\] = {                                                |
|                                                                                |
| \'roi\': roi,                                                                  |
|                                                                                |
| \'alpha\': alpha,                                                              |
|                                                                                |
| \'ec\': sat\[\'ec\'\],                                                         |
|                                                                                |
| \'slope\': sat\[\'slope\'\]                                                    |
|                                                                                |
| }                                                                              |
+--------------------------------------------------------------------------------+

## 4.7 Paso 5: construir priors informados

+--------------------------------------------------------------------------+
| from meridian.model import prior_distribution                            |
|                                                                          |
| import tensorflow_probability as tfp                                     |
|                                                                          |
| import numpy as np                                                       |
|                                                                          |
| tfd = tfp.distributions                                                  |
|                                                                          |
| roi_means = \[empirical_params\[ch\]\[\'roi\'\] for ch in CHANNELS\]     |
|                                                                          |
| alpha_means = \[empirical_params\[ch\]\[\'alpha\'\] for ch in CHANNELS\] |
|                                                                          |
| ec_means = \[empirical_params\[ch\]\[\'ec\'\] for ch in CHANNELS\]       |
|                                                                          |
| slope_means = \[empirical_params\[ch\]\[\'slope\'\] for ch in CHANNELS\] |
|                                                                          |
| CONFIDENCE = 0.10 \# prior estrecho                                      |
|                                                                          |
| informed_prior = prior_distribution.PriorDistribution(                   |
|                                                                          |
| roi_m=tfd.LogNormal(                                                     |
|                                                                          |
| loc=np.log(roi_means) - (CONFIDENCE \*\* 2) / 2,                         |
|                                                                          |
| scale=\[CONFIDENCE\] \* len(CHANNELS)                                    |
|                                                                          |
| ),                                                                       |
|                                                                          |
| alpha_m=tfd.Beta(                                                        |
|                                                                          |
| concentration1=np.array(alpha_means) \* 50,                              |
|                                                                          |
| concentration0=(1 - np.array(alpha_means)) \* 50                         |
|                                                                          |
| ),                                                                       |
|                                                                          |
| ec_m=tfd.LogNormal(                                                      |
|                                                                          |
| loc=np.log(ec_means) - (0.15 \*\* 2) / 2,                                |
|                                                                          |
| scale=\[0.15\] \* len(CHANNELS)                                          |
|                                                                          |
| ),                                                                       |
|                                                                          |
| slope_m=tfd.LogNormal(                                                   |
|                                                                          |
| loc=np.log(slope_means) - (0.20 \*\* 2) / 2,                             |
|                                                                          |
| scale=\[0.20\] \* len(CHANNELS)                                          |
|                                                                          |
| )                                                                        |
|                                                                          |
| )                                                                        |
+--------------------------------------------------------------------------+

## 4.8 Paso 6: cargar datos y especificar el modelo

+-----------------------------------------------------------------------+
| from meridian.data import load                                        |
|                                                                       |
| from meridian.model import model, spec                                |
|                                                                       |
| loader = load.CsvDataLoader(                                          |
|                                                                       |
| csv_path=\'meridian_input.csv\',                                      |
|                                                                       |
| kpi_type=\'non_revenue\',                                             |
|                                                                       |
| coord_to_columns=load.CoordToColumns(                                 |
|                                                                       |
| time=\'date\',                                                        |
|                                                                       |
| geo=\'region\',                                                       |
|                                                                       |
| kpi=\'conversions\',                                                  |
|                                                                       |
| controls=\[\'holiday_flag\', \'price\', \'gqv\'\],                    |
|                                                                       |
| media_spend=\[f\'{ch}\_spend\' for ch in CHANNELS\],                  |
|                                                                       |
| media=\[f\'{ch}\_impressions\' for ch in CHANNELS\]                   |
|                                                                       |
| ),                                                                    |
|                                                                       |
| media_to_channel={f\'{ch}\_impressions\': ch for ch in CHANNELS},     |
|                                                                       |
| media_spend_to_channel={f\'{ch}\_spend\': ch for ch in CHANNELS}      |
|                                                                       |
| )                                                                     |
|                                                                       |
| data = loader.load()                                                  |
|                                                                       |
| model_spec = spec.ModelSpec(                                          |
|                                                                       |
| prior=informed_prior,                                                 |
|                                                                       |
| media_effects_dist=\'log_normal\',                                    |
|                                                                       |
| hill_before_adstock=False,                                            |
|                                                                       |
| max_lag=8,                                                            |
|                                                                       |
| media_prior_type=\'roi\' \# clave para Ruta 1                         |
|                                                                       |
| )                                                                     |
+-----------------------------------------------------------------------+

## 4.9 Paso 7: entrenamiento rápido con priors estrechos

+-----------------------------------------------------------------------+
| mmm = model.Meridian(input_data=data, model_spec=model_spec)          |
|                                                                       |
| \# Entrenamiento acelerado porque los priors son estrechos            |
|                                                                       |
| mmm.sample_prior(n_draws=500)                                         |
|                                                                       |
| mmm.sample_posterior(                                                 |
|                                                                       |
| n_chains=2,                                                           |
|                                                                       |
| n_adapt=200,                                                          |
|                                                                       |
| n_burnin=200,                                                         |
|                                                                       |
| n_keep=500,                                                           |
|                                                                       |
| seed=42                                                               |
|                                                                       |
| )                                                                     |
|                                                                       |
| \# Validación obligatoria de convergencia                             |
|                                                                       |
| from meridian.analysis import analyzer                                |
|                                                                       |
| diagnostics = analyzer.ModelDiagnostics(mmm)                          |
|                                                                       |
| r_hat = diagnostics.get_rhat()                                        |
|                                                                       |
| assert r_hat.max() \< 1.1, \'Modelo no convergió\'                    |
+-----------------------------------------------------------------------+

## 4.10 Paso 8: optimización del presupuesto

+-----------------------------------------------------------------------+
| from meridian.analysis import optimizer                               |
|                                                                       |
| budget_optimizer = optimizer.BudgetOptimizer(mmm)                     |
|                                                                       |
| \# Escenario A: mismo presupuesto, mejor distribución                 |
|                                                                       |
| opt_fixed = budget_optimizer.optimize(                                |
|                                                                       |
| use_kpi=True,                                                         |
|                                                                       |
| fixed_budget=True,                                                    |
|                                                                       |
| budget=None                                                           |
|                                                                       |
| )                                                                     |
|                                                                       |
| \# Escenario B: buscar presupuesto óptimo dado target de ROI          |
|                                                                       |
| opt_target = budget_optimizer.optimize(                               |
|                                                                       |
| use_kpi=True,                                                         |
|                                                                       |
| fixed_budget=False,                                                   |
|                                                                       |
| target_roi=2.0                                                        |
|                                                                       |
| )                                                                     |
|                                                                       |
| \# Escenario C: con restricciones por canal                           |
|                                                                       |
| constraints = {                                                       |
|                                                                       |
| \'meta_prospecting\': {\'min_spend\': 5000, \'max_spend\': 20000},    |
|                                                                       |
| \'google_search_brand\': {\'min_spend\': 2000, \'max_spend\': 8000}   |
|                                                                       |
| }                                                                     |
|                                                                       |
| opt_custom = budget_optimizer.optimize(                               |
|                                                                       |
| use_kpi=True,                                                         |
|                                                                       |
| fixed_budget=True,                                                    |
|                                                                       |
| channel_constraints=constraints                                       |
|                                                                       |
| )                                                                     |
|                                                                       |
| \# Generar reporte HTML                                               |
|                                                                       |
| opt_fixed.output_budget_optimization_summary(                         |
|                                                                       |
| filepath=\'./optimization_report.html\'                               |
|                                                                       |
| )                                                                     |
+-----------------------------------------------------------------------+

## 4.11 Paso 9: validación antes de implementar

Crítico: antes de ejecutar las recomendaciones en el plan real, valida que el modelo respetó tus priors:

+---------------------------------------------------------------------------------+
| for channel in CHANNELS:                                                        |
|                                                                                 |
| modeled_roi = mmm.inference_data\[\'roi_m\'\]\\                                 |
|                                                                                 |
| .mean(dim=\[\'chain\', \'draw\'\]).values\[CHANNELS.index(channel)\]            |
|                                                                                 |
| empirical_roi = empirical_params\[channel\]\[\'roi\'\]                          |
|                                                                                 |
| diff_pct = abs(modeled_roi - empirical_roi) / empirical_roi                     |
|                                                                                 |
| print(f\'{channel}: empírico {empirical_roi:.2f} vs modelo {modeled_roi:.2f}\', |
|                                                                                 |
| f\'(diff {diff_pct:.1%})\')                                                     |
+---------------------------------------------------------------------------------+

+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Regla de validación**                                                                                                                                                                                                                                                                                   |
|                                                                                                                                                                                                                                                                                                           |
| Si la diferencia entre ROI empírico y ROI del modelo es menor al 5-10%, tus priors estrechos funcionaron correctamente y el modelo aprendió lo que le dijiste. Si la diferencia supera el 20%, los datos observados contradicen tus estimaciones y debes investigar antes de aplicar las recomendaciones. |
+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

# Parte 5. Consideraciones prácticas y riesgos

## 5.1 Ventajas concretas de la Ruta 1

-   Aprovechas el optimizador oficial de Meridian, probado y confiable.

-   No construyes el optimizador matemático desde cero.

-   Obtienes reportes HTML listos para compartir con stakeholders.

-   Iteraciones rápidas: el entrenamiento toma minutos, no horas.

-   Puedes probar múltiples escenarios what-if fácilmente.

## 5.2 Riesgos principales y mitigaciones

### Riesgo 1: estimaciones empíricas incorrectas

Si tus ROI o adstock están mal estimados, las recomendaciones serán subóptimas. Mitigación: valida al menos algunos parámetros con tests de incrementalidad puntuales (Conversion Lift o Geo Lift) antes de implementar cambios grandes.

### Riesgo 2: multicolinealidad entre Meta y Google

Si siempre mueves ambos canales a la par, el modelo no puede separar sus efectos ni con priors informados. Mitigación: introduce variaciones intencionales de gasto o corre tests controlados donde modifiques uno solo.

### Riesgo 3: overfit a patrones espurios

Los datos históricos pueden tener correlaciones no causales. Si siempre aumentaste Meta en Black Friday, el modelo puede atribuirle a Meta el lift natural de la temporada. Mitigación: incluye controles fuertes (holiday_flag, gqv, price).

### Riesgo 4: cambios de plataforma invalidan las curvas

Si Meta cambia su algoritmo o Google lanza nuevos formatos, tus curvas históricas pueden quedar obsoletas. Mitigación: reestima los parámetros cada 3-6 meses.

## 5.3 Cuándo la Ruta 1 tiene sentido

La Ruta 1 está diseñada para tres escenarios específicos:

6.  Quieres el optimizador y los reportes de Meridian sin el costo completo del entrenamiento bayesiano.

7.  Ya tienes conocimiento fuerte sobre tus canales (tests previos, años de experiencia con los mismos clientes).

8.  Necesitas un entregable formal llamado \'modelo Meridian\' (cliente, certificación, partnership con Google).

Si no te aplica ninguno de estos tres, es probable que te convenga más un optimizador custom en scipy. Pero si alguno aplica, esta guía cubre el flujo completo.

## 5.4 Plan de implementación gradual

9.  Mes 1: extracción y limpieza de datos desde Windsor; estimación de parámetros empíricos.

10. Mes 2: construcción de priors, entrenamiento inicial y validación de convergencia.

11. Mes 3: optimización con distintos escenarios; comparación contra asignación actual.

12. Mes 4: aplicación gradual de recomendaciones (15-20% de reasignación inicial).

13. Mes 5: monitoreo de resultados reales vs predichos y calibración.

14. Mes 6: si la desviación es baja, aplicar más agresivamente; si es alta, revisar priors.

# Parte 6. Variables necesarias: checklist completo

Esta sección resume todas las variables que debes extraer, calcular o definir para implementar la Ruta 1 en tu caso específico.

## 6.1 Variables a extraer desde Windsor

### 6.1.1 Por campaña, diarias, últimos 2 años mínimo

  ------------------------------------------------------------------------------------------------
  **Variable**      **Tipo**    **Observaciones**
  ----------------- ----------- ------------------------------------------------------------------
  date              fecha       Granularidad diaria; se agregará a semanal

  source            string      \'google_ads\' o \'facebook\'

  campaign_name     string      Para categorizar en subcanales (Search, PMax, Prospecting, etc.)

  region            string      Ciudad o departamento; esencial para modelado jerárquico

  spend             numérico    Gasto en moneda local; variable predictora principal

  impressions       numérico    Puede usarse en vez de o además del gasto

  clicks            numérico    Útil como variable auxiliar; no imprescindible

  conversions       numérico    Tu KPI principal si kpi_type=\'non_revenue\'

  revenue           numérico    Tu KPI principal si kpi_type=\'revenue\'

  reach             numérico    Específico para YouTube y Meta video (personas únicas)

  frequency         numérico    Específico para canales con reach (impresiones/personas)
  ------------------------------------------------------------------------------------------------

## 6.2 Variables de control a añadir (externas a Windsor)

  -----------------------------------------------------------------------------------------------------------------------------------
  **Variable**           **Fuente**            **Razón para incluirla**
  ---------------------- --------------------- --------------------------------------------------------------------------------------
  holiday_flag           Calendario manual     Feriados, Black Friday, Navidad, día de la madre, etc.

  price                  Sistema interno       Precio promedio del producto en esa semana y región

  promo_flag             Sistema interno       Descuentos, cupones, campañas promocionales internas

  gqv                    Google Trends         Google Query Volume: demanda orgánica de búsquedas relacionadas

  competitor_activity    SimilarWeb, Semrush   Inversión publicitaria de competidores si puedes estimarla

  other_channels_spend   Otras plataformas     TikTok, prensa, TV, email si los usas; evita que su efecto se atribuya a Meta/Google
  -----------------------------------------------------------------------------------------------------------------------------------

## 6.3 Variables calculadas por el pipeline

Estas no las extraes directamente: las genera el pipeline a partir de las anteriores.

-   **sub_channel:** categoría asignada a cada fila usando la función categorize_campaign (google_search_brand, google_pmax, meta_prospecting, etc.).

-   **week:** semana calendario; agregación de los datos diarios.

-   **roas_observado:** por canal, calculado como revenue_total / spend_total.

-   **roi_incremental:** roas_observado × factor_ajuste_por_canal.

-   **alpha_empirico:** estimado por grid search de correlación.

-   **ec_empirico y slope_empirico:** estimados por curve_fit con la función Hill.

## 6.4 Parámetros que debes definir manualmente

### 6.4.1 Factores de ajuste ROAS a ROI

Uno por canal, basado en benchmarks de industria o en tus tests de incrementalidad si los tienes. Los valores sugeridos se mencionaron en la sección 3.1.2.

### 6.4.2 Uncertainty de los priors

Un valor entre 0.05 y 0.50 que refleja qué tan confiado estás en tus estimaciones empíricas. Recomendación inicial: 0.10 para ROI, 0.15 para ec, 0.20 para slope.

### 6.4.3 Restricciones por canal (opcional)

Gasto mínimo y máximo permitido por subcanal. Útil cuando hay compromisos contractuales, presupuesto mínimo por seniority del canal, o techos operativos.

## 6.5 Resumen consolidado: checklist final

+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| **Lo que debes tener listo antes de empezar**                                                                                                                                                                                                                                                                                                                                                                          |
|                                                                                                                                                                                                                                                                                                                                                                                                                        |
| Un dataset semanal de al menos 2 años con columnas de gasto e impresiones por subcanal, conversiones o revenue como KPI, y al menos 3 variables de control (holiday_flag, price, gqv). Los parámetros empíricos ROI, α, ec y slope calculados por canal. Factores de ajuste por canal definidos y uncertainty de priors fijado. Ambiente Python 3.11+ con Meridian, TensorFlow Probability, pandas y scipy instalados. |
+------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

## 6.6 Cierre

Con todos los elementos anteriores implementados, ya tienes un sistema de optimización que utiliza el optimizador oficial de Meridian aprovechando tu conocimiento empírico acumulado, sin depender por completo de la inferencia bayesiana desde cero. Es un balance pragmático entre rigor metodológico y eficiencia operativa.

El siguiente paso natural es automatizar el pipeline para que corra periódicamente (mensual o trimestralmente), comparar las predicciones vs resultados reales y calibrar los priors cada 3-6 meses con los resultados observados. Con esta disciplina, el sistema mejora con el tiempo y las recomendaciones se vuelven progresivamente más precisas.
