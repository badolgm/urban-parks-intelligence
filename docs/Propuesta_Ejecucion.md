# Propuesta de Ejecución: Roadmap Operacional
**Enfoque:** Implementación del Modelo Predictivo — Pipeline Parques AMVA

---

## 1. Metodología Técnica — 7 Tarjetas

| # | Tarjeta | Detalle Técnico | Estado |
|:--|:--------|:-----------|:-------|
| 1 | **ETL Robusto + Auditoría** | `unicodedata` + colapso de corrupciones de encoding | ✅ Implementado |
| 2 | **Feature Engineering RFM+** | 12 features incluyendo categóricas one-hot | ✅ Implementado |
| 3 | **K-Means** | 6 features de clustering con `pct_fin_semana` y `n_meses_activo` | ✅ Implementado |
| 4 | **Modelado Supervisado** | `pd.get_dummies` real sobre género, franja y categoría edad | ✅ Implementado |
| 5 | **Evaluación** | AUC=0.7537 verificado sobre datos originales | ✅ Verificado |
| 6 | **Smart Pricing** | Umbral 11.92% | ✅ Calculado |
| 7 | **Estacionalidad + Exportación** | Exporta `df_resultados_analisis_parques.csv` | ✅ Implementado |

---

## 2. Plan de Acción — Fases Estratégicas

### Fase 1 — Calidad de Ingesta (Inmediata)
Implementar la función `normalizar_texto` + `colapsar_corrupcion_encoding` como paso obligatorio
del ETL antes de cualquier análisis. Esto previene la acumulación de variantes sucias en
producción y garantiza que los dashboards de estacionalidad reflejen categorías reales.

**Validación:** Confirmar que `tarifa` tiene 5 categorías y `tipo_venta` tiene 2 en cada ejecución.

### Fase 2 — Despliegue del Clasificador
- Usar **Random Forest con 12 features** (archivo `analisis_parques.ipynb`, Tarjeta 4).
- Generar lista de usuarios con probabilidad de churn >0.5 para intervención mensual.
- Priorizar segmento "Usuarios en Riesgo de Deserción" (alta recencia, baja frecuencia).

### Fase 3 — Activación Smart Pricing
- **Segmento VIP:** Priorizar servicios premium, sin descuentos.
- **Segmento Ocasional/Riesgo:** Descuentos ≤ **11.92%** (umbral técnico).
- **Segmento Inactivo:** Campañas de reactivación de bajo costo (email/push).

### Fase 4 — Monitoreo y Mejora de Datos
Implementar un identificador de sesión en el sistema de punto de venta para distinguir
visitas reales del mismo usuario en el mismo día (actualmente marcadas como duplicados).
Esto permitirá reducir el 89.8% de usuarios con 1 sola visita y mejorar estructuralmente
la señal del modelo de churn.

---

## 3. Indicadores de Éxito (KPIs)

| KPI | Valor Actual | Meta Fase 3 |
|:----|:---------------:|:-----------:|
| ROC-AUC del modelo | 0.7537 | ≥ 0.80 (con datos más densos) |
| Cardinalidad `tarifa` | 5 | 5 (estable en producción) |
| Umbral de descuento | 11.92% | Monitoreo mensual |
| % usuarios con ≥3 visitas | 2.4% | ≥ 5% (meta retención) |

---

## 4. Deuda Técnica Documentada

| Item | Prioridad | Descripción |
|:-----|:---------:|:------------|
| Definición de churn basada en recencia pura | Media | Para el 89.8% de usuarios con 1 visita, churn = función de la fecha de esa visita. Mejorar con datos más densos. |
| `grupo` normalizado pero no usado | Baja | 95 categorías post-limpieza. Requiere análisis de negocio para determinar si aporta señal. |

---
*Pipeline validado reproduciblemente contra los datos originales.*
