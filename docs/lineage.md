# Linaje de Datos — NYC Taxi Pipeline

Diagrama del flujo de datos desde las fuentes externas hasta la capa Refined.

## Diagrama de Linaje (Mermaid)

```mermaid
graph TD
    subgraph FUENTES["Fuentes Externas"]
        SRC1["Parquet: yellow_tripdata_2023-01<br/>(NYC TLC - ~3M registros)"]
        SRC2["CSV: taxi_zone_lookup<br/>(263 zonas)"]
    end

    subgraph RAW["Capa Raw (Sin transformacion)"]
        R1["raw.yellow_taxi_trips<br/>(Delta Table)"]
        R2["raw.taxi_zone_lookup<br/>(Delta Table)"]
    end

    subgraph TRUSTED["Capa Trusted (Limpieza + Enriquecimiento)"]
        T1["trusted.yellow_taxi_trips_clean<br/>(Registros validados + join zonas)"]
        T2["trusted.yellow_taxi_trips_rejected<br/>(Registros descartados + motivo)"]
    end

    subgraph REFINED["Capa Refined (KPIs + Calidad)"]
        KPI1["refined.kpi_demand_pattern<br/>(Demanda por franja y dia)"]
        KPI2["refined.kpi_economic_efficiency<br/>(Top 10 zonas rentables)"]
        KPI3["refined.kpi_data_quality_impact<br/>(Impacto de registros descartados)"]
        DQ["refined.data_quality_report<br/>(Expectations de calidad)"]
        EXEC["refined.pipeline_execution_report<br/>(Metricas de ejecucion JSON)"]
    end

    SRC1 -->|"Ingesta directa"| R1
    SRC2 -->|"Ingesta directa"| R2

    R1 -->|"Validacion + Filtrado"| T1
    R1 -->|"Registros invalidos"| T2
    R2 -->|"LEFT JOIN por pickup_location_id"| T1

    T1 -->|"Agregacion por franja/dia"| KPI1
    T1 -->|"Revenue/mile por zona"| KPI2
    T2 -->|"Conteo por regla de rechazo"| KPI3
    T1 -->|"Expectations de calidad"| DQ
    T1 -->|"Metricas globales"| EXEC
    T2 -->|"Metricas de descarte"| EXEC

    style RAW fill:#4A90D9,color:#fff
    style TRUSTED fill:#F5A623,color:#fff
    style REFINED fill:#7ED321,color:#fff
    style FUENTES fill:#9B9B9B,color:#fff
```

## Transformaciones por Capa

| Capa | Entrada | Transformacion | Salida |
|------|---------|----------------|--------|
| **Raw** | Parquet (URL publica) | Ninguna (tipado basico) | `raw.yellow_taxi_trips` |
| **Raw** | CSV (URL publica) | Ninguna (inferSchema) | `raw.taxi_zone_lookup` |
| **Trusted** | raw.yellow_taxi_trips + raw.taxi_zone_lookup | Validacion, limpieza nulos, filtrado outliers, JOIN, renombrado columnas, calculo duracion | `trusted.yellow_taxi_trips_clean` |
| **Trusted** | raw.yellow_taxi_trips (rechazados) | Etiquetado con `rejection_reason` | `trusted.yellow_taxi_trips_rejected` |
| **Refined** | trusted.yellow_taxi_trips_clean | Agregacion por franja horaria y dia | `refined.kpi_demand_pattern` |
| **Refined** | trusted.yellow_taxi_trips_clean | Revenue/mile y velocidad por zona | `refined.kpi_economic_efficiency` |
| **Refined** | trusted.yellow_taxi_trips_rejected | Conteo e impacto por regla | `refined.kpi_data_quality_impact` |
| **Refined** | trusted.yellow_taxi_trips_clean | 2 expectations de calidad | `refined.data_quality_report` |
