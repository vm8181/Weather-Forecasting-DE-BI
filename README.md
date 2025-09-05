# 🌈Weather Forecast Analysis

## 📖 Project Overview
This project demonstrates an end-to-end data pipeline and analytics solution for live and forecasted weather data using **Microsoft Fabric**. A custom **Python**. crawler **ingests real-time and forecast data from the OpenWeather API**, enriched with metadata such as sunrise/sunset times, etc.

On the **Data Engineering** side, the pipeline handles ingestion, lineage, and incremental refreshes. On the **Business Intelligence** side, **Power BI (DirectLake)** delivers **interactive dashboard**, including city-level snapshots, historical trends, and forecast insights.

The solution follows a **Medallion Architecture Bronze → Silver → Gold** design pattern, separating raw storage, cleaned append logs, and deduplicated reporting data.

This project highlights effective collaboration between DE and BI practices—scalable ingestion, clean modeling, and actionable insights for decision-making. 

---
## 🛠️ Tools & Technologies

This project integrates Data Engineering + Business Intelligence capabilities using modern tools:

- **Microsoft Fabric**
   - Unified Data & Analytics SaaS platform.
   - Provides Lakehouse, Dataflow Gen2, Pipelines, and DirectLake for seamless data movement and reporting.
   - Enables Medallion Architecture (Bronze → Silver → Gold).

- **Python (with Pandas & Requests)**
   - Crawler Notebook written in Python scrapes live weather APIs/websites.
   - Pandas used for data wrangling (cleaning, transforming CSV files).
   - Handles timestamps, city-level records, and saves CSVs to OneLake.

- **SQL (Fabric SQL Endpoint)**
   - Used for validation and monitoring queries.
   - Ensures deduplication, data coverage by city, and latest timestamps.
   - Helps in data quality checks on Gold tables.

- **Power BI + Power Automate**
   - Visualizes trends and latest weather snapshots.
   - Connects directly to Gold table via Warehouse → On-demand dataset refresh available.
   - Power Automate to automate the on-demand refresh to get the update data in the report.

- **Power Query / Dataflow Gen2**
   - Transforms raw Bronze files into Silver (append log) and Gold (deduplicated) Delta tables.
   - Provides lineage columns (source_file, crawl_time, ingestion_time).
   - Supports incremental and automated transformations.

- **Data Pipelines (Fabric Pipeline)**
   - Orchestrates workflow: Crawler Notebook → Wait → Dataflow refresh.
   - Handles retries, timeouts, and monitoring.

- **Business Intelligence & Data Engineering Integration**
   - BI layer (Power BI) provides insights, trends, and live snapshots.
   - DE layer (Fabric Lakehouse, Pipelines, Dataflow) ensures scalable ingestion, transformation, and storage.

Together, they deliver an end-to-end robust BI + DE solution.

## 🏗️ Architecture

### Components
1. **Crawler Notebook (Fabric Notebook)**  
   - Scrapes weather data for multiple cities.  
   - Saves raw CSV files into:  
     ```
     Files/forecast_data_broze/forecast_data {yyyy-MM-dd HH:mm:ss}.csv
     ```
   - Example: `forecast_data_bronze 2025-08-27 10:15:30.csv`.

2. **Lakehouse (WeatherLakehouse)**  
   - Stores raw files in `Files/weather_data_broze`.  
   - Contains **Silver** and **Gold** Delta tables.

3. **Dataflow Gen2 (Weather Dataflow)**  
   - **Silver Table (`forecast_data_silver`)**  
     - Appends all crawled datasets with lineage columns:  
       - `source_file`, `file_crawl_time`, `ingestion_time`.  
   - **Gold Table (`forecast_data_gold`)**  
     - Deduplicated by `(city, date_time)`.  
     - Columns renamed for clarity (e.g. `temperature_c`, `humidity_pct`).  
     - Used directly by Power BI.

4. **Pipeline (WeatherHourlyPipeline)**  
   - Orchestrates:
     1. Run **Notebook (Data_Crawler)**.  
     2. **Wait 20–30 seconds** (file commit buffer).  
     3. Refresh **Dataflow Gen2**.  
   - Runs **every 4 hour (schedule)**.

5. **Power BI Dashboard**  
   - Connects directly to `forecast_weather_gold` via **Warehouse**.  
   - Pages:
     - **Overview**: Pending.  
     - **Snapshot**: latest temperature/humidity by city.
---

## 🔄 Data Flow

```
Notebook (Crawler)
    ↓
Raw CSV files (Files/forecast_data_broze, hourly_data_broze, weather_data_broze in WeatherLakehouse)
    ↓
Dataflow Gen2
    ├─ Silver Table: forecast_weather_silver, hourly_weather_silver, current_weather_silver (append log)
    └─ Gold Table: forecast_weather_gold, hourly_weather_gold, current_weather_gold (deduped, reporting)
    ↓
Pipeline (orchestrates Notebook → Wait → Dataflow refresh)
    ↓
Power BI (Warehouse → Gold table)
    ↓
Users (insights + On-Demand refresh)
```

## ⚙️ Pipeline Configuration

**Activities:**
1. **Data_Crawler (Notebook)**  
   - Timeout: `0:10:00`  
   - Retry: `2`  
   - Retry interval: `120 sec`

2. **Wait**  
   - Duration: `20–30 sec` (ensures files are committed in OneLake)

3. **Refresh_Weather_Dataflow (Dataflow Gen2)**  
   - Timeout: `0:20:00`  
   - Retry: `2`  
   - Retry interval: `120 sec`

**Schedule:** Every **4 hour**, no concurrent runs.

---

## 📊 Power BI Setup

### Connection
- Dataset: **forecast_weather_gold, hourly_weather_gold, current_weather_gold** (Warehouse from Lakehouse).
- **On-Demand dataset refresh available**, will help to get updated data in the report. 

### Weather Report:  
 <img width="1492" height="862" alt="image" src="https://github.com/user-attachments/assets/b9144fd3-bfdf-4524-8980-5db9060a1390" />

---

## ✅ Validation & Monitoring

### SQL Checks (Lakehouse SQL Endpoint)
```sql
-- Latest record timestamp
SELECT MAX(date_time) AS latest_ts FROM forecast_data_gold;

-- Coverage by city
SELECT city, COUNT(*) AS rows_, MIN(date_time) AS first_dt, MAX(date_time) AS last_dt
FROM forecast_data_gold
GROUP BY city;

-- Check duplicates
SELECT city, date_time, COUNT(*) c
FROM forecast_data_gold
GROUP BY city, date_time
HAVING COUNT(*) > 1;
```

### Monitoring
- **Pipeline run history** → check success/failures.  
- **Dataflow refresh history** → row counts.  
- **Power BI card** → shows `Latest Timestamp` for confidence.

---

## 🛠️ Best Practices

- **Keep raw files untouched** → always stored in `Files/forecast_weather_bronze`.  
- **Use Silver for append logs** → preserves lineage.  
- **Use Gold for reporting** → deduped, clean, optimized.  
- **On-Demand Refresh in Power BI** →  One click refresh button is available.  
- **Retry & Timeout settings** → handle transient failures gracefully.  
- **Single source of refresh (Pipeline)** → avoid dedup errors. 

---

## 🗂️ Repository Structure
```
/WeatherProject
│
├─ /notebooks
│   └─ crawler_notebook.ipynb
│
├─ /dataflows
│   └─ weather_data_ETL.json
│
├─ /pipelines
│   └─ weather_datapipeline.json
    └─ hourly_weather_pipeline.json
│
├─ /powerbi
│   └─ WeatherReport.pbix
│
└─ README.md   ← (this file)
```

---

💡 With this setup, you get **a robust end-to-end BI solution**:  
✔ Continuous history  
✔ Live updates on demand  
✔ Scalable Lakehouse storage  
✔ Seamless DirectLake reporting  
