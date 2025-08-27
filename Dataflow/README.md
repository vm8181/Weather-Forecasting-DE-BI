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
// Lakehouse navigation (keep your IDs)
Source = Lakehouse.Contents(null),
Navigation = Source{[workspaceId = "abec62f6-1fc0-46ba-a00e-3d9b73229de3"]}[Data],
#"Navigation 1" = Navigation{[lakehouseId = "78471b36-06e6-4207-8772-243722b76e5b"]}[Data],

// Files root → expand only the minimal subcolumns we need
FilesRoot = #"Navigation 1"{[Id = "Files", ItemKind = "Folder"]}[Data],
#"Expanded Content" =
    Table.ExpandTableColumn(
        FilesRoot,
        "Content",
        {"Content", "Name", "Extension", "Folder Path"},
        {"Content.1", "Name.1", "Extension.1", "Folder Path.1"}
    ),

// Filter only weather CSV files in RawDataset
#"Filtered to RawDataset" =
    Table.SelectRows(
        #"Expanded Content",
        each Text.Contains([Folder Path.1], "/Files/weather_data_bronze/", Comparer.OrdinalIgnoreCase)
             and Text.EndsWith([Extension.1], ".csv", Comparer.OrdinalIgnoreCase)
             and Text.StartsWith([Name.1], "weather_dataset ")
    ),

// Keep minimal file metadata
#"Kept Minimal File Columns" = Table.SelectColumns(#"Filtered to RawDataset", {"Content.1", "Name.1"}),

// Apply transform function to all files
#"Invoke custom function" = Table.AddColumn(#"Kept Minimal File Columns", "Transform file", each #"Transform file"([Content.1])),

// Expand combined data
#"Expanded table column" =
    Table.ExpandTableColumn(
        #"Invoke custom function",
        "Transform file",
        Table.ColumnNames(#"Transform file"(#"Sample file"))
    ),

// Select relevant columns
#"Selected Data Columns" = Table.SelectColumns(
    #"Expanded table column",
    {"date_time","city","temperature(°C)","feels_like(°C)","temp_min(°C)","temp_max(°C)","pressure(hPa)","humidity(%)","visibility(m)","wind_speed(m/s)","wind_deg(°)","cloudiness(%)","weather_main","weather_description","Name.1"}
),

// Add lineage & crawl time
#"Added source_file" = Table.RenameColumns(#"Selected Data Columns", {{"Name.1", "source_file"}}),
#"Added file_crawl_time" =
    Table.AddColumn(
        #"Added source_file",
        "file_crawl_time",
        each try DateTime.FromText( Text.BetweenDelimiters([source_file], "weather_dataset ", ".csv") ) otherwise null,
        type datetime
    ),

// Final types
#"Changed column type" = Table.TransformColumnTypes(#"Added file_crawl_time", {
    {"date_time", type datetime},
    {"city", type text},
    {"temperature(°C)", type number},
    {"feels_like(°C)", type number},
    {"temp_min(°C)", type number},
    {"temp_max(°C)", type number},
    {"pressure(hPa)", Int64.Type},
    {"humidity(%)", Int64.Type},
    {"visibility(m)", Int64.Type},
    {"wind_speed(m/s)", type number},
    {"wind_deg(°)", Int64.Type},
    {"cloudiness(%)", Int64.Type},
    {"weather_main", type text},
    {"weather_description", type text},
    {"source_file", type text},
    {"file_crawl_time", type datetime}
})
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
