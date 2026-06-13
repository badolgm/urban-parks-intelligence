# Propuesta de Ejecución: Roadmap Operacional
**Enfoque:** Implementación del Modelo Predictivo en el Pipeline del Parque.

---

## 1. Metodología Técnica (Estandarización 7 Tarjetas)
Se ha seguido el estándar de flujo operativo definido en el Notebook:
1. **Arquitectura:** Ingesta robusta y normalización de datos.
2. **Feature Engineering:** Cálculo de métricas RFM (Recencia, Frecuencia, Monetario).
3. **Clustering:** Segmentación comportamental con validación WCSS.
4. **Modelado Supervisado:** Comparativa entre Regresión Logística y Random Forest.
5. **Evaluación:** Validación técnica mediante curvas ROC-AUC.
6. **Optimización:** Modelo de elasticidad precio-frecuencia.
7. **Interfaz:** Reportes y dashboards estratégicos.

## 2. Plan de Acción — Fases Estratégicas
- **Despliegue (Pipeline):** Integrar el clasificador *Random Forest* para generar listas diarias de usuarios en riesgo.
- **Activación (Smart Pricing):** - Segmento VIP: Priorizar servicios premium (no descuentos).
    - Segmento Riesgo/Ocasional: Aplicar umbral de descuento del **18.71%**.
- **Monitoreo de Saturación:** Ajustar mantenimientos en "meses de valle" basados en el análisis estacional 2024-2025.

## 3. Indicadores de Éxito (KPIs)
- **Precisión del Modelo:** ROC-AUC de 0.7338.
- **Eficiencia Financiera:** Reducción de subsidios ineficientes mediante el tope de descuento definido.
- **Retención:** Reducción de la tasa de abandono mediante alertas tempranas.

---
*Este documento constituye la base operativa para la ejecución del modelo.*
