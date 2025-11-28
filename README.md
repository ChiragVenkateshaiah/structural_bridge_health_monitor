# structural_bridge_health_monitor
This project demonstrates an end-to-end real-time data engineering pipeline built using Databricks Lakeflow declarative pipelines (formerly Delta Live Tables) and the Medallion Architecture on Unity Catalog. The scenario simulates live IoT sensor data from five major European bridges, each equipped with three sensors continuously transmitting three key metrics every minute:

- Ambient temperature
- Deck vibration
- Tilt angle
The goal of the pipeline is to detect abnormal patterns such as unusual vibration spikes or slow-moving structural tilt in near real time.

# Data Simulation & Ingestion (Landing → Bronze)

To simulate realistic IoT streaming, I created a Python generator that emits synthetic readings per device every minute.
Each event’s timestamp is intentionally offset by a random 0–60 seconds to mimic real-world network latency and out-of-order arrival—teaching the pipeline to handle slightly late data.
These raw streaming batches are appended into three independent Delta tables in the Bronze layer, forming continuously updating sources for downstream processing.

# Silver Layer – Enrichment, Metadata & Data Quality with DLT

A static bridge metadata table is materialized to map each device ID to its bridge name, location, and structural attributes.
For each raw Bronze stream (temperature, vibration, tilt), I built a streaming Silver table that:
- Applies DLT expectations to enforce data quality
- Performs a stream–static join with the bridge metadata
- Adds contextual fields such as bridge name and location

This produces three clean, enriched, self-describing streams. Invalid rows are tracked through quality rules and optionally quarantined.

# Gold Layer – Real-Time Aggregations & Unified Monitoring View
The Gold layer combines all streams into a consolidated real-time monitoring dataset.
For each bridge and every incoming stream:
  - A 10-minute sliding window is used to compute:
    - Average temperature
    - Maximum vibration
    - Maximum tilt angle

  - A two-minute watermark accommodates the intentional out-of-order timestamps while still bounding state.
Next, the three aggregated streams are joined on:
- Bridge ID
- Window start
- Window end

This stream-to-stream join ensures that each row corresponds to a single bridge and a single 10-minute interval, containing all three sensor metrics.

The final output is an append-only, production-ready Gold table delivering a unified operational view for real-time structural health monitoring.

# Pipeline Characteristics

- All tables (except bridge metadata) are streaming tables.
- The entire Lakeflow DLT pipeline is declarative, incremental, and processes only new or changed data.
- Implements full Medallion Architecture on Unity Catalog:
  Landing → Bronze → Silver → Gold


This project highlights core data engineering capabilities, including streaming ingestion, declarative pipeline design, data quality management, incremental processing, and advanced time-series aggregations—all implemented using Databricks Lakeflow on a governed Unity Catalog environment.
