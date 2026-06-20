# Informe Ejecutivo: Estrategia de Retención y Valor de Clientes
**Framework:** Analítica Avanzada de Comportamiento Urbano (Proyecto Parques 2024–2025)

---

## 1. Resumen de Hallazgos Técnicos

Se procesaron **304,799 registros transaccionales**, que tras limpieza y deduplicación quedan en
**221,004 registros válidos**, correspondientes a **189,384 usuarios únicos**. El pipeline aplica
normalización de texto robusta y feature engineering extendido, con métricas verificadas por
ejecución directa contra los archivos de origen.

| Indicador | Valor |
|:--|:--:|
| Registros brutos | 304,799 |
| Registros tras ETL | 221,004 |
| Usuarios únicos | 189,384 |
| Categorías reales en `tarifa` (tras normalizar) | 5 |
| Categorías reales en `tipo_venta` (tras normalizar) | 2 |
| Features del modelo de churn | 12 |

---

## 2. Segmentación de Usuarios (K-Means, K=4)

El modelo agrupa a los usuarios en 4 segmentos a partir de 6 variables (recencia, frecuencia, gasto
neto, edad, proporción de visitas en fin de semana y diversidad de meses activos):

| Segmento | Usuarios | Perfil |
|:--|:--:|:--|
| **Usuarios VIP Recurrentes** | 3,598 | Mayor frecuencia y gasto, visitas distribuidas en varios meses. Activo crítico de retención. |
| **Asistentes Ocasionales Activos** | 59,562 | Recencia relativamente buena, visita poco frecuente. Oportunidad de conversión a recurrencia. |
| **Usuarios en Riesgo de Deserción** | 74,580 | Recencia intermedia-alta, baja frecuencia. Candidatos a intervención preventiva. |
| **Visitantes Inactivos / Perdidos** | 51,644 | Recencia más alta del sistema. Requieren campañas de reactivación de bajo costo. |

---

## 3. Predicción de Fuga (Churn Analysis)

El clasificador **Random Forest** con 12 features alcanza:

- **ROC-AUC: 0.7537**, superando a la Regresión Logística (0.5593) por 19.4 puntos porcentuales.
- El resultado está **impulsado casi en su totalidad por `monetario_neto` (53%) y `edad` (24%)** —
  juntas explican el 77% de la importancia del modelo. Las variables adicionales incorporadas en
  esta iteración (comportamiento de fin de semana, diversidad de meses activos, género, franja
  horaria, categoría de edad) aportan en conjunto un 5% adicional: contribución real pero marginal,
  no el motor principal del resultado.

**Limitación que debe leerse junto con el AUC:** el 89.8% de los usuarios tiene una sola visita
registrada en los dos años de cobertura. Para ese grupo, la "recencia" usada para definir el churn
equivale a la fecha de esa única visita, no a un patrón de abandono observado. El modelo es válido
sobre los datos disponibles, pero una estrategia de alertas tempranas con mayor confianza requeriría
datos de mayor densidad temporal por usuario (ver recomendación en sección 5, Fase 4).

---

## 4. Estrategia de Precios (Smart Pricing)

- **Correlación descuento–frecuencia ≈ 0** (Pearson -0.03, Spearman 0.02): el descuento masivo no
  está asociado a mayor recurrencia.
- **Umbral técnico: 11.92%**, calculado sobre el ratio de descuento promedio de usuarios que
  alcanzaron 3 o más visitas.
- **Recomendación:** aplicar el descuento exclusivamente a "Asistentes Ocasionales Activos" y
  "Usuarios en Riesgo de Deserción". No aplicar a "VIP Recurrentes" — su comportamiento de gasto no
  depende del descuento, y subsidiarlos erosiona margen sin ganancia de retención.

---

## 5. Hoja de Ruta Operacional

1. **Fase 1 — Calidad de datos (inmediata):** incorporar la normalización de texto robusta al
   proceso de ingesta del sistema de punto de venta, antes de que las variantes sucias se acumulen
   en producción.
2. **Fase 2 — Activación del modelo:** desplegar el clasificador para generar listas mensuales de
   usuarios priorizando "Usuarios en Riesgo de Deserción".
3. **Fase 3 — Smart Pricing:** activar campañas segmentadas con techo de 11.92% de descuento,
   monitoreando como KPI principal la conversión a una tercera visita.
4. **Fase 4 — Mejora estructural de datos:** evaluar la incorporación de un identificador de
   sesión/visita en el sistema de punto de venta. Esto permitiría distinguir visitantes
   genuinamente recurrentes de visitantes únicos con mayor precisión, y sustentar una definición de
   churn con base conductual más sólida que la actual.

---
*Métricas verificadas por ejecución directa y reproducible contra los archivos CSV originales.
Detalle técnico completo disponible en `AUDITORIA_TECNICA.md`.*
