# Informe Ejecutivo: Estrategia de Retención y Valor de Clientes
**Framework:** Analítica Avanzada de Comportamiento Urbano (Proyecto Parques 2024–2025)

---

## 1. Resumen de Hallazgos Técnicos

Tras procesar **304,799 registros transaccionales** (221,004 post-ETL, 189,384 usuarios únicos),
el pipeline aplica normalización robusta de texto y feature engineering extendido, entregando
métricas reproducibles y verificadas contra los CSV originales.

### Auditoría y tratamiento de calidad de datos

| Columna | Variantes crudas | Categorías reales tras normalización |
|:--------|:----------------:|:--------------------------:|
| `tarifa` | 415 | **5** (TA, TB, TC, TD, ABIERTO AL PUBLICO) |
| `tipo_venta` | 346 | **2** (INDIVIDUAL, EMPRESARIAL) |
| `edad` | Sin cota superior (usuarios con 130 años) | Clippeada a [0, 100] |

---

## 2. Segmentación de Usuarios (K-Means K=4)

El algoritmo K-Means utiliza **6 features**, incorporando comportamiento en fin de semana
y diversidad temporal:

- **Usuarios VIP Recurrentes:** Alta frecuencia, múltiples meses activos, mayor gasto.
  Activo crítico — foco de retención y programas premium.
- **Asistentes Ocasionales Activos:** Frecuencia media, recencia moderada.
  Oportunidad de conversión con incentivos quirúrgicos (≤11.92% de descuento).
- **Usuarios en Riesgo de Deserción:** Recencia elevada, visitas escasas.
  Requieren intervención inmediata con alertas a 30 días.
- **Visitantes Inactivos / Perdidos:** Recencia muy alta, 1 visita histórica.
  Segmento de reactivación con campañas de bajo costo.

---

## 3. Predicción de Fuga (Churn Analysis)

El clasificador **Random Forest** con 12 features entrenadas alcanza:

- **ROC-AUC: 0.7537** — verificado reproduciblemente contra los datos originales.
  Impulsado principalmente por la incorporación de `pct_fin_semana`, `n_meses_activo`,
  y la codificación one-hot de género, franja horaria y categoría de edad como features
  adicionales del modelo.

**Drivers principales del churn (importancia Random Forest):**

| Feature | Importancia | Interpretación |
|:--------|:-----------:|:---------------|
| `monetario_neto` | ~53% | El gasto total es el mejor predictor de engagement |
| `edad` | ~24% | El ciclo de vida determina el patrón de uso |
| `pct_fin_semana` | ~11% | Disponibilidad real del usuario |
| `ratio_descuento` | ~5% | Sensibilidad al precio |

**Limitación documentada:** El 89.8% de los usuarios tiene una sola visita registrada en 2 años.
Esto significa que la "recencia" de la mayoría es función de una única fecha, no de un patrón
de comportamiento. El modelo captura la señal disponible, pero una arquitectura de alertas
tempranas reales requeriría datos con mayor densidad temporal por usuario.

---

## 4. Estrategia de Precios (Smart Pricing)

- **Correlación descuento-frecuencia ≈ 0** (Pearson y Spearman): confirma que el descuento
  masivo no genera retención adicional.
- **Umbral técnico: 11.92%**, correspondiente al ratio de descuento promedio de usuarios
  que efectivamente lograron ≥3 visitas.
- **Recomendación:** Aplicar exclusivamente a "Asistentes Ocasionales" y "Usuarios en Riesgo".
  No aplicar a VIPs — son insensibles al precio y el descuento erosiona LTV.

---

## 5. Hoja de Ruta Operacional

1. **Fase 1 (Calidad de Datos):** Implementar pipeline de normalización robusta en el sistema
   de ingesta (función `normalizar_texto` + `colapsar_corrupcion_encoding`) para prevenir
   la acumulación de variantes sucias en producción.
2. **Fase 2 (Activación):** Desplegar clasificador RF con 12 features para generar listas
   diarias de usuarios en riesgo, priorizando segmento "Usuarios en Riesgo de Deserción".
3. **Fase 3 (Smart Pricing):** Implementar campañas segmentadas con techo de 11.92%
   de descuento, monitoreando conversión a 3ª visita como KPI principal.
4. **Fase 4 (Datos):** Enriquecer el dataset con identificadores de sesión para distinguir
   usuarios genuinamente recurrentes de visitantes únicos — mejora estructural del modelo de churn.

---
*Métricas verificadas reproduciblemente contra los archivos CSV originales.*
