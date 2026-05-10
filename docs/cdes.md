# Critical Data Elements (CDEs)

Elementos criticos de datos identificados en el dataset de NYC Yellow Taxi Trips.
Estos campos son fundamentales para la integridad de los KPIs del pipeline.

---

## CDE-001: `fare_amount`

| Atributo             | Detalle                                                                 |
|----------------------|-------------------------------------------------------------------------|
| **Tabla/Columna**    | `trusted.yellow_taxi_trips_clean.fare_amount`                           |
| **Tipo**             | `double`                                                                |
| **Definicion de negocio** | Tarifa base calculada por el taximetro segun distancia y tiempo recorrido. No incluye extras, propinas, peajes ni recargos. |
| **Regla de calidad** | Debe ser mayor a 0 y menor o igual a 500 USD. Valores fuera de rango se consideran outliers y se rechazan a la tabla `trusted.yellow_taxi_trips_rejected`. |
| **KPIs afectados**   | KPI 1 (tarifa promedio por franja horaria), KPI 2 (ingreso promedio por milla). |
| **Impacto si falla** | Distorsion directa de metricas financieras. Un `fare_amount` negativo o extremo invalida los promedios de ingreso y el ranking de zonas rentables. |

---

## CDE-002: `trip_distance`

| Atributo             | Detalle                                                                 |
|----------------------|-------------------------------------------------------------------------|
| **Tabla/Columna**    | `trusted.yellow_taxi_trips_clean.trip_distance`                         |
| **Tipo**             | `double`                                                                |
| **Definicion de negocio** | Distancia total del viaje en millas, registrada por el taximetro desde el punto de recogida hasta el destino final. |
| **Regla de calidad** | Debe ser mayor a 0 y menor o igual a 200 millas. Valores cero indican viajes cancelados; valores extremos indican errores de GPS o taximetro. |
| **KPIs afectados**   | KPI 2 (ingreso por milla, velocidad promedio).                          |
| **Impacto si falla** | Un `trip_distance` de 0 causa division por cero en revenue/mile. Valores extremos distorsionan el ranking de eficiencia por zona. |

---

## CDE-003: `tpep_pickup_datetime`

| Atributo             | Detalle                                                                 |
|----------------------|-------------------------------------------------------------------------|
| **Tabla/Columna**    | `trusted.yellow_taxi_trips_clean.tpep_pickup_datetime`                  |
| **Tipo**             | `timestamp`                                                             |
| **Definicion de negocio** | Fecha y hora en que el taximetro se activo al iniciar el viaje (momento de recogida del pasajero). |
| **Regla de calidad** | Debe ser estrictamente anterior a `tpep_dropoff_datetime`. Ademas, debe estar dentro del rango del mes procesado (enero 2023). |
| **KPIs afectados**   | KPI 1 (franja horaria, dia de la semana), calculo de `trip_duration_minutes`. |
| **Impacto si falla** | Si pickup >= dropoff, la duracion del viaje es negativa o cero, invalidando KPI 1 (promedios de duracion) y KPI 2 (velocidad promedio). Si la fecha esta fuera de enero 2023, contamina el analisis temporal. |
