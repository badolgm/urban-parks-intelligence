# Propuesta de Ejecución: Roadmap Operacional
**Enfoque:** Implementación del Modelo Predictivo — Pipeline Parques AMVA

---

## 1. Metodología Técnica — 7 Tarjetas

| # | Tarjeta | Detalle Técnico | Estado |
|:--|:--------|:-----------|:-------|
| 1 | **ETL e Ingeniería de Calidad de Datos** | `unicodedata` + colapso de corrupciones de encoding | Implementado |
| 2 | **Feature Engineering RFM+** | 12 features incluyendo categóricas one-hot | Implementado |
| 3 | **K-Means** | 6 features de clustering (RFM + `pct_fin_semana` + `n_meses_activo`) | Implementado |
| 4 | **Modelado Supervisado** | `pd.get_dummies` sobre género, franja horaria y categoría de edad | Implementado |
| 5 | **Evaluación** | AUC=0.7537, verificado por ejecución directa contra los datos originales | Verificado |
| 6 | **Smart Pricing** | Umbral de descuento: 11.92% | Calculado |
| 7 | **Estacionalidad y Exportación** | Exporta `df_resultados_analisis_parques.csv` | Implementado |

---

## 2. Plan de Acción — Fases Estratégicas

### Fase 1 — Calidad de Ingesta (inmediata)

Incorporar la normalización de texto (`normalizar_texto` + `colapsar_corrupcion_encoding`) como
paso obligatorio del ETL en el sistema de ingesta, no solo en el análisis posterior. Esto evita
que las variantes sucias se sigan acumulando en producción y garantiza que cualquier reporte de
estacionalidad o mix de producto trabaje sobre categorías reales.

**Criterio de validación:** confirmar en cada ejecución que `tarifa` resuelve a 5 categorías y
`tipo_venta` a 2.

### Fase 2 — Despliegue del Clasificador

- Desplegar el Random Forest de 12 features (`analisis_parques.ipynb`, Tarjeta 4) para generar
  listas mensuales de usuarios con probabilidad de churn elevada.
- Priorizar el segmento **"Usuarios en Riesgo de Deserción"** (74,580 usuarios, recencia mediana
  358 días) para intervención preventiva.

### Fase 3 — Activación Smart Pricing

- **Usuarios VIP Recurrentes:** priorizar servicios premium, sin descuentos — son insensibles
  al precio (correlación descuento–frecuencia ≈ 0).
- **Asistentes Ocasionales Activos / Usuarios en Riesgo de Deserción:** descuentos de hasta
  **11.92%** (umbral técnico calculado).
- **Visitantes Inactivos / Perdidos:** campañas de reactivación de bajo costo (email/push), no
  descuentos — su recencia es demasiado alta para que el incentivo de precio sea la palanca correcta.

### Fase 4 — Mejora Estructural de Datos

Evaluar la incorporación de un identificador de sesión/visita en el sistema de punto de venta,
para distinguir con certeza visitas genuinamente recurrentes de visitantes únicos. Esto es
prerrequisito para sustentar una definición de churn con base conductual sólida — hoy, el 89.8%
de los usuarios tiene una sola visita registrada, lo que limita la interpretación causal del
modelo actual (ver detalle en `AUDITORIA_TECNICA.md`, sección 3).

### Fase 5 — Madurez del Modelo (antes de producción)

Antes de poner el clasificador en producción con impacto operativo real, se recomienda:
- Validación cruzada (`StratifiedKFold`) para reportar el AUC con su intervalo de confianza, no
  como un único número de un solo split.
- Comparación contra un modelo de gradient boosting (XGBoost/LightGBM) y contra un baseline
  ingenuo, como referencia de cuánto valor agrega la complejidad del Random Forest.
- Análisis de costo de falsos positivos/negativos para fijar el umbral de decisión de forma
  deliberada, en vez de usar 0.5 por defecto.

El detalle completo de esta fase está desarrollado en `AUDITORIA_TECNICA.md`, sección 4.

---

## 3. Indicadores de Éxito (KPIs)

| KPI | Valor Actual | Meta Fase 5 |
|:----|:---------------:|:-----------:|
| ROC-AUC del modelo (split único) | 0.7537 | Reportar como media ± std vía cross-validation |
| Cardinalidad `tarifa` | 5 | 5 (estable en producción) |
| Umbral de descuento | 11.92% | Monitoreo mensual de vigencia |
| % usuarios con ≥3 visitas | 2.35% | ≥ 5% (meta de retención) |

---

## 4. Deuda Técnica Documentada

| Item | Prioridad | Descripción |
|:-----|:---------:|:------------|
| Definición de `churn` basada en recencia pura | Alta | Para el 89.8% de usuarios con 1 visita, `churn` equivale a la fecha de esa única visita, no a un patrón observado. Requiere decisión de negocio, no solo técnica — ver `AUDITORIA_TECNICA.md`. |
| Ausencia de validación cruzada y tuning de hiperparámetros | Alta | El AUC reportado proviene de un único split. Ver Fase 5. |
| `grupo` normalizado (95 categorías) pero no usado como feature | Baja | Requiere decisión de negocio: descartar del dataset final o agregar en categorías de mayor nivel para uso futuro. |
| Pipeline como notebook lineal, sin pruebas unitarias | Media | Migrar funciones de limpieza y feature engineering a un módulo `.py` testeable facilitaría mantenimiento y reuso. |

---
*Pipeline ejecutado y validado de extremo a extremo contra los archivos CSV originales.
Detalle técnico completo en `AUDITORIA_TECNICA.md`.*
