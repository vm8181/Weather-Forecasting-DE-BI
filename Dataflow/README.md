## Microsoft Fabric (Dataflow Gen2)

This demonstrates a bronze–silver–gold dataflow architecture in Microsoft Fabric using Dataflow Gen2 (Power Query M).

I use raw weather datasets (weather_dataset *.csv) stored in Lakehouse Files and transform them step by step into clean, structured tables.

### 🏗️ Architecture
```Raw CSV (Bronze) → Dataflow Gen2 → Silver (Clean, Typed) → Gold (Curated Subset) → Power BI / Analytics```

- **Bronze**: Raw files stored in Lakehouse /Files/weather_data_bronze/.
- **Silver**: Cleaned, typed, and enriched table (weather_data_silver).
- **Gold**: Analytical subset for reporting (weather_data_gold).

Each file contains weather readings (temperature, humidity, pressure, etc.) for multiple cities.

### 📂 Data Sources
Input: Multiple CSV files named like

```weather_dataset 2025-08-20 10-00-00.csv```
```weather_dataset 2025-08-20 11-00-00.csv```

**Location**: /Files/weather_data_bronze/ inside your Lakehouse.

Each file contains weather readings (temperature, humidity, pressure, etc.) for multiple cities.

---

## ⚙️ Dataflow Queries

### 1. Silver Layer – `weather_data_silver`

**Purpose**:  
- Load raw CSV files  
- Filter only weather dataset files  
- Combine them into one table  
- Extract crawl timestamp from file name  
- Apply strong data types  

```m
let
  // ========= Lakehouse navigation =========
  Source          = Lakehouse.Contents(null),
  Navigation      = Source{[workspaceId = "abec62f6-1fc0-46ba-a00e-3d9b73229de3"]}[Data],
  Lh              = Navigation{[lakehouseId  = "78471b36-06e6-4207-8772-243722b76e5b"]}[Data],

  // ========= Files (expand minimal columns only) =========
  FilesRoot       = Lh{[Id = "Files", ItemKind = "Folder"]}[Data],
  ExpandedFiles   = Table.ExpandTableColumn(
                      FilesRoot, "Content",
                      {"Content", "Name", "Extension", "Folder Path"},
                      {"FileBinary", "Name.1", "Extension.1", "FolderPath"}
                    ),

  // ========= Early filter: folder + csv + filename prefix =========
  Filtered        = Table.SelectRows(
                      ExpandedFiles,
                      each Text.Contains([FolderPath], "/Files/forecast_data_bronze/", Comparer.OrdinalIgnoreCase)
                        and Text.EndsWith([Extension.1], ".csv", Comparer.OrdinalIgnoreCase)
                        and Text.StartsWith([Name.1], "weather_forecast ")
                    ),

  // Keep only the two columns we really need from the file list
  PrunedFiles     = Table.SelectColumns(Filtered, {"FileBinary", "Name.1"}),

  // ========= Parse CSV (self-contained, no helper queries) =========
  Parsed          = Table.AddColumn(
                      PrunedFiles, "Data",
                      each
                        let
                          Csv  = Csv.Document([FileBinary],[Delimiter=",", Encoding=65001, QuoteStyle=QuoteStyle.Csv]),
                          Tbl  = Table.PromoteHeaders(Csv, [PromoteAllScalars=true])
                        in
                          Tbl
                    ),
  #"Expanded Data" = Table.ExpandTableColumn(Parsed, "Data", {"crawl_time", "city", "city_lat", "city_lon", "forecast_time", "temp_c", "feels_like_c", "temp_min_c", "temp_max_c", "pressure_hpa", "humidity_pct", "visibility_m", "wind_speed_ms", "wind_deg", "cloudiness_pct", "weather_main", "weather_desc", "pop", "rain_3h_mm", "snow_3h_mm"}, {"crawl_time", "city", "city_lat", "city_lon", "forecast_time", "temp_c", "feels_like_c", "temp_min_c", "temp_max_c", "pressure_hpa", "humidity_pct", "visibility_m", "wind_speed_ms", "wind_deg", "cloudiness_pct", "weather_main", "weather_desc", "pop", "rain_3h_mm", "snow_3h_mm"}),

  // ========= Strong types (defensive) =========
  Typed           = Table.TransformColumnTypes(
                      #"Expanded Data",
                      {
                        {"crawl_time",      type datetime},
                        {"forecast_time",   type datetime},
                        {"city",            type text},
                        {"city_lat",        type number},
                        {"city_lon",        type number},
                        {"temp_c",          type number},
                        {"feels_like_c",    type number},
                        {"temp_min_c",      type number},
                        {"temp_max_c",      type number},
                        {"pressure_hpa",    type number},
                        {"humidity_pct",    type number},
                        {"visibility_m",    type number},
                        {"wind_speed_ms",   type number},
                        {"wind_deg",        type number},
                        {"cloudiness_pct",  type number},
                        {"weather_main",    type text},
                        {"weather_desc",    type text},
                        {"pop",             type number},
                        {"rain_3h_mm",      type number},
                        {"snow_3h_mm",      type number}
                      }
                    ),
  #"Renamed columns" = Table.RenameColumns(Typed, {{"Name.1", "source_file"}}),
  #"Removed columns" = Table.RemoveColumns(#"Renamed columns", {"FileBinary"})
in
  #"Removed columns"
```

### 2. Gold Layer – `weather_data_gold`
**Purpose**:  
Provide a curated dataset with only analytical columns (drop metadata).  
```m
let
  Source = weather_data_silver,
  #"Removed other columns" = Table.SelectColumns(Source, {"date_time", "city", "temperature(°C)", "feels_like(°C)", "temp_min(°C)", "temp_max(°C)", "pressure(hPa)", "humidity(%)", "visibility(m)", "wind_speed(m/s)", "wind_deg(°)", "cloudiness(%)", "weather_main", "weather_description"})
in
  #"Removed other columns"
```
---

### ✅ Output Tables
- weather_data_silver → Full dataset with lineage & file metadata
- weather_data_gold → Clean analytical dataset ready for BI

### 📊 Usage
- Connect Power BI directly to the Gold table for reporting.
- Use Silver for debugging, lineage, and data quality checks.
- Extend the pipeline by adding a Platinum layer (aggregations, KPIs).
in
#"Changed column type"
