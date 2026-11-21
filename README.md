# EDA y Pronóstico de Afluencia de Pasajeros del Metro (CDMX)

**Trabajo Final — Asignatura: Big Data**

Análisis exploratorio de datos (EDA) y pronóstico de demanda a 30 días de la afluencia de pasajeros del Metro (Líneas A y B), construido sobre una arquitectura **Medallion** (Bronze → Silver → Gold) en Databricks, con registro de modelos en **MLflow**.

## 📊 Datos

- **Fuente:** `Data metro.xlsx` (hoja `Hoja1`) — matriz ancha con horas en filas y fechas + líneas en columnas.
- **Alcance:** 212 días (1 enero – 31 julio 2026), Líneas A y B, horas de operación 04:00–23:00.
- **Volumen:** 8,480 registros validados (sin nulos ni negativos, coincidencia 100% con el origen).

## 🏗️ Arquitectura Medallion

| Capa | Tabla | Contenido |
|---|---|---|
| **Bronze** | `bronze_metro_pasajeros_excel` | Copia cruda del Excel: matriz ancha original, todo como texto, con metadatos de ingesta |
| **Silver** | `silver_metro_pasajeros` | Datos limpios y *tidy*: `fecha`, `linea`, `hora`, `pasajeros` |
| **Gold** | `gold_metro_pasajeros_diarios` | Total diario por línea + hora pico, día de la semana, fin de semana, mes, promedio móvil 7 días y total del sistema |
| **Gold** | `gold_metro_pasajeros_hora` | Perfil típico por hora y línea (promedio, total y participación de cada hora) |
| **Gold** | `gold_metro_tendencia` | Regresión lineal de tendencia por línea con clasificación: AUMENTA / DISMINUYE / ESTABLE |
| **Gold** | `gold_metro_pronostico_30d` | Pronóstico a 30 días: tendencia × factor estacional del día de la semana |

## 🔍 Hallazgos principales

1. **Calidad de datos**: 8,480 registros validados sin nulos ni negativos; la capa Silver coincide 100% con el origen.
2. **Perfil temporal**: picos dobles de mañana (06:00–08:00) y tarde, con **hora pico a las 17:00**; martes y miércoles son los días más cargados y el domingo el de menor afluencia.
3. **Estructura de la demanda**: la **Línea A concentra ~87.4%** del total del sistema (~6.9 veces la Línea B).
4. **Tendencia y pronóstico**: ambas líneas se clasifican como **ESTABLE** a 30 días (variación < ±2%); agosto 2026 pronostica ~18.7M de pasajeros, consistente con el nivel histórico.

## 🔮 Metodología del pronóstico

- Por cada línea se ajusta una **regresión lineal** sobre los totales diarios (parámetros y modelos registrados en MLflow).
- `proyeccion_30d_pct` = variación esperada en 30 días respecto al promedio diario.
- Clasificación: **AUMENTA** (> +2%), **DISMINUYE** (< −2%) o **ESTABLE** (entre ±2%).
- El pronóstico a 30 días combina la **tendencia** con el **factor estacional de cada día de la semana**.
- *Limitación*: R² ≈ 0 — la variación diaria la domina el patrón semanal, no la tendencia lineal; el pronóstico es confiable a 30 días, no a largo plazo.

## 💡 Implicaciones para la toma de decisiones

- **Operación**: concentrar refuerzos de servicio entre semana en la tarde (16:00–19:00, pico 17:00); usar domingo y horas nocturnas (>21:00) como ventana de bajo impacto para mantenimiento.
- **Planificación**: la estabilidad de la demanda no justifica ampliación de capacidad a corto plazo; priorizar eficiencia y calidad de servicio.
- **Línea B**: su baja utilización sugiere revisar frecuencias y promocionar su uso.
- **Monitoreo**: re-ejecutar el notebook con datos nuevos; un cambio de clasificación (±2%) debe gatillar replanificación.

## 🚀 Cómo reproducirlo

1. Importar el notebook `EDA y pronostico Metro Medallion (1).ipynb` en **Databricks** (o verlo directamente en este repositorio — las gráficas están incluidas como imágenes estáticas).
2. El notebook construye todas las tablas Medallion y los gráficos automáticamente desde el Excel fuente.
3. Ejecutar todas las celdas de arriba a abajo: al re-ejecutar con datos nuevos, tablas, gráficos, respuestas y el resumen se recalculan.

**Dependencias** (se instalan en la primera celda): `openpyxl`, `plotly` (matplotlib está incluido en el runtime de Databricks).

## 👥 Integrantes

| Nombre | Código |
|---|---|
| Milena Estrada Muñoz | 1.001.035.917 |
| Yuliana Estrada Calle | 1.007.253.661 |
| Isis Valentina Palacio Taborda | 1.000.306.896 |
| Maria Antonia Pino Giraldo | 1.000.760.444 |
