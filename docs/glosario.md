# Glosario de Negocio

Definiciones de los terminos clave utilizados en el pipeline de datos de NYC Yellow Taxi.

---

## 1. Viaje Valido

Un registro de viaje que ha pasado todas las reglas de validacion de la capa Trusted:
- La fecha/hora de recogida (`pickup`) es anterior a la de destino (`dropoff`).
- La distancia (`trip_distance`) es mayor a 0 y menor o igual a 200 millas.
- La tarifa (`fare_amount`) es mayor a 0 y menor o igual a 500 USD.
- El monto total (`total_amount`) es mayor o igual a 0.
- El numero de pasajeros es entre 1 y 6.
- La duracion esta entre 1 y 300 minutos.

Los registros que no cumplen alguna regla se mueven a la tabla de rechazados con su motivo.

---

## 2. Hora Pico (Peak Hour)

Franja horaria y dia de la semana cuyo volumen de viajes se encuentra en el **percentil 80 o superior** del total de combinaciones franja-dia. En el KPI de demanda temporal (`refined.kpi_demand_pattern`), estas combinaciones se marcan con `is_peak = true`. Tipicamente corresponden a las horas de la manana (08-12h) y tarde-noche (16-20h) en dias laborales.

---

## 3. Borough

Division administrativa de la ciudad de Nueva York. Existen cinco boroughs:
- **Manhattan**
- **Brooklyn**
- **Queens**
- **Bronx**
- **Staten Island**

Ademas, el dataset incluye las categorias **EWR** (Newark Airport) y **Unknown** para zonas no mapeadas. El borough se obtiene del cruce con la tabla `taxi_zone_lookup` usando el `PULocationID` (zona de recogida).

---

## 4. Tarifa Base (Fare Amount)

Monto en dolares calculado automaticamente por el taximetro en funcion de la distancia recorrida y el tiempo transcurrido durante el viaje. **No incluye**:
- Propinas (`tip_amount`)
- Peajes (`tolls_amount`)
- Recargo por congestion (`congestion_surcharge`)
- Tarifa de aeropuerto (`airport_fee`)
- Impuesto MTA (`mta_tax`)
- Recargos adicionales (`extra`, `improvement_surcharge`)

La tarifa base inicial en 2023 es de **$3.00** (tarifa de bajada de bandera).

---

## 5. Eficiencia por Milla (Revenue per Mile)

Metrica calculada como `total_amount / trip_distance`, expresada en **dolares por milla**. Representa cuanto ingreso genera cada milla recorrida en un viaje. Se utiliza en el KPI 2 (`refined.kpi_economic_efficiency`) para identificar las zonas y boroughs mas rentables. Viajes cortos en zonas de alta demanda tienden a tener mayor eficiencia por milla que viajes largos a las afueras.
