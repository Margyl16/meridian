# Google Meridian · Ruta 1: Priors informados

Repositorio de conocimiento sobre la implementación de **Google Meridian** (framework de Marketing Mix Modeling de código abierto) usando la **Ruta 1**: priors informados con datos históricos.

Incluye el marco conceptual y matemático del modelo, la guía técnica de implementación, y un plan paso a paso ilustrado con un caso práctico.

---

## ¿Qué es Google Meridian?

Meridian es el framework open-source de Google para Marketing Mix Modeling (MMM). Permite estimar cuánto contribuye cada canal de marketing a tus ventas y optimizar cómo distribuir el presupuesto entre canales.

A diferencia de un MMM tradicional que aprende todo desde cero con inferencia bayesiana (proceso costoso en datos y tiempo), la **Ruta 1** aprovecha estimaciones empíricas previas (ROI histórico, adstock, curvas de saturación) como *priors informados*, lo que permite:

- Entrenar el modelo en minutos en lugar de horas
- Usar el optimizador oficial de Meridian y sus reportes HTML
- Aprovechar conocimiento acumulado del negocio sin depender 100% de los datos

---

## Estructura del repositorio

```
.
├── README.md                          ← este archivo (portada)
├── docs/
│   ├── meridian_ruta1_tecnico.docx    ← documento técnico (Word original)
│   ├── plan_meridian_ejemplos.docx    ← plan con ejemplos (Word original)
│   ├── 01-documento-tecnico.md        ← documento técnico (versión Markdown)
│   └── 02-plan-implementacion.md      ← plan con ejemplos (versión Markdown)
```

---

## Documentos

### 📘 [1. Documento técnico](docs/01-documento-tecnico.md)

Marco conceptual, matemático y de estimación de parámetros. Cubre:

- La ecuación central de Meridian y cada uno de sus términos
- Las funciones Adstock (persistencia temporal) y Hill (saturación) al detalle
- Cómo estimar los 4 parámetros críticos (ROI, α, ec, slope) desde datos históricos
- Conversión de estimaciones empíricas en priors
- Aplicación al caso Meta Ads + Google Ads con datos de Windsor.ai
- Checklist completo de variables necesarias

### 📗 [2. Plan de implementación con ejemplos](docs/02-plan-implementacion.md)

Plan operativo en 7 fases ilustrado con el caso ficticio de **ModaViva** (e-commerce de ropa, USD 60k/mes en Colombia, 5 ciudades, 18 meses de histórico):

- **Fase 0** · Validación y preparación (1 semana)
- **Fase 1** · Extracción y estructuración de datos (2-3 semanas)
- **Fase 2** · Estimación empírica de parámetros (1 semana)
- **Fase 3** · Modelado con Meridian (2 semanas)
- **Fase 4** · Optimización de presupuesto (1 semana)
- **Fase 5** · Aplicación gradual y validación (4-6 semanas)
- **Fase 6** · Ciclo operativo continuo

Cada fase incluye código funcional, outputs esperados y métricas de validación.

---

## ¿A quién le sirve este repositorio?

- **Analistas de marketing y data scientists** que necesitan implementar MMM en clientes con histórico de datos limitado
- **Agencias de medios** que quieren ofrecer optimización de presupuesto basada en Meridian
- **Equipos in-house** de marketing con campañas en Meta Ads y Google Ads
- Cualquier persona que quiera entender **cómo funciona matemáticamente** Meridian antes de implementarlo

---

## Stack técnico de referencia

- **Python 3.11+**
- `google-meridian` (framework oficial)
- `tensorflow-probability` (distribuciones para priors)
- `pandas`, `numpy`, `scipy` (manipulación y ajuste de curvas)
- `Windsor.ai` (pipeline de datos desde Meta y Google Ads)
- `Google Colab Pro+` (entorno recomendado con GPU T4/A100)

---

## Recursos externos

- [Documentación oficial de Google Meridian](https://developers.google.com/meridian)
- [Repositorio oficial en GitHub](https://github.com/google/meridian)
- [Windsor.ai](https://windsor.ai) · conector de datos

---

## Licencia

Este material se comparte con fines educativos y de divulgación técnica.
