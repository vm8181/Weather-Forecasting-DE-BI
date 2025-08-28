## Microsoft Fabric (Dataflow Gen2)

This demonstrates a bronze–silver–gold dataflow architecture in Microsoft Fabric using Dataflow Gen2 (Power Query M).

I use raw weather datasets (forecast_data *.csv) stored in Lakehouse Files and transform them step by step into clean, structured tables.

<img width="1813" height="432" alt="image" src="https://github.com/user-attachments/assets/f4e49938-9ba1-4ca9-afc1-8e59738832b7" />

### 🏗️ Architecture
```Raw CSV (Bronze) → Dataflow Gen2 → Silver (Clean, Typed) → Gold (Curated Subset) → Power BI / Analytics```

- **Bronze**: Raw files stored in Lakehouse /Files/forecast_data_bronze/.
- **Silver**: Cleaned, typed, and enriched table (weather_data_silver).
- **Gold**: Analytical subset for reporting (weather_data_gold).

Each file contains weather readings (temperature, humidity, pressure, etc.) for multiple cities.

### 📂 Data Sources
Input: Multiple CSV files named like

```forecast_data 2025-08-20 10-00-00.csv```
```forecast_data 2025-08-20 11-00-00.csv```

**Location**: /Files/forecast_data_bronze/ inside your Lakehouse.

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
  #"Expanded Content" = Table.ExpandTableColumn(FilesRoot, "Content", {"Content", "Name", "Extension", "Date accessed", "Date modified", "Date created", "Attributes", "Folder Path"}, {"Content.1", "Name.1", "Extension.1", "Date accessed.1", "Date modified.1", "Date created.1", "Attributes.1", "Folder Path.1"}),
  #"Filtered hidden files" = Table.SelectRows(#"Expanded Content", each [Attributes]?[Hidden]? <> true),
  #"Invoke custom function" = Table.AddColumn(#"Filtered hidden files", "Transform file", each #"Transform file"([Content.1])),
  #"Removed other columns" = Table.SelectColumns(#"Invoke custom function", {"Name.1", "Folder Path.1", "Transform file"}),
  #"Expanded table column" = Table.ExpandTableColumn(#"Removed other columns", "Transform file", Table.ColumnNames(#"Transform file"(#"Sample file"))),
  #"Renamed columns" = Table.RenameColumns(#"Expanded table column", {{"Name.1", "source_file"}, {"Folder Path.1", "folder_path"}})
  in
    #"Renamed columns"
```

### 2. Gold Layer – `weather_data_gold`
**Purpose**:  
Provide a curated dataset with only analytical columns (drop metadata).  
```m
let
  Source = weather_data_silver,
  #"Removed columns" = Table.RemoveColumns(Source, {"source_file", "folder_path", "timezone_offset_s"}),
  #"Capitalized each word" = Table.TransformColumns(#"Removed columns", {{"weather_desc", each Text.Proper(Text.From(_)), type nullable text}}),
  #"Added custom" = Table.AddColumn(#"Capitalized each word", "visibility_km", each Number.FromText([visibility_m]) / 1000),
  #"Changed column type" = Table.TransformColumnTypes(#"Added custom", {{"crawl_time", type datetime}, {"forecast_time", type datetime}, {"sunrise_ist", type datetime}, {"sunset_ist", type datetime}, {"weather_desc", type text}, {"weather_main", type text}, {"city", type text}, {"snow_3h_mm", type number}, {"rain_3h_mm", type number}, {"pop", type number}, {"wind_speed_ms", type number}, {"cloudiness_pct", type number}, {"wind_deg", type number}, {"humidity_pct", type number}, {"visibility_m", type number}, {"pressure_hpa", type number}, {"temp_max_c", type number}, {"temp_c", type number}, {"feels_like_c", type number}, {"temp_min_c", type number}, {"city_lat", type number}, {"city_lon", type number}, {"visibility_km", type number}})
in
  #"Changed column type"
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
