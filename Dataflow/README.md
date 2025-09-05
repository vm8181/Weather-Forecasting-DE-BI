## Microsoft Fabric (Dataflow Gen2)

This demonstrates a bronze–silver–gold dataflow architecture in Microsoft Fabric using Dataflow Gen2 (Power Query M).

I use raw weather datasets (forecast_data, hourly_data & weather_data *.csv) stored in Lakehouse Files and transform them step by step into clean, structured tables.

<img width="1765" height="794" alt="image" src="https://github.com/user-attachments/assets/5786b83c-6548-4066-9948-156ccc2d69bc" />

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
**Purpose**:  
- Load raw CSV files  
- Filter only weather dataset files  
- Combine them into one table  
- Extract crawl timestamp from file name  
- Apply strong data types  

### 1. Silver Layer – `forecast_weather_silver`
```m
let
  Source = Lakehouse.Contents(null),
  Navigation = Source{[workspaceId = "e406e233-b34a-489d-8cd5-8851d4e2a74a"]}[Data],
  #"Navigation 1" = Navigation{[lakehouseId = "5bf8e2ae-a8c7-41ec-ad55-1c807526ffa9"]}[Data],
  #"Navigation 2" = #"Navigation 1"{[Id = "Files", ItemKind = "Folder"]}[Data],
  #"Filtered rows" = Table.SelectRows(#"Navigation 2", each ([Name] = "forecast_data_bronze")),
  #"Expanded Content" = Table.ExpandTableColumn(#"Filtered rows", "Content", {"Content", "Name"}, {"Content.1", "Name.1"}),
  #"Filtered hidden files" = Table.SelectRows(#"Expanded Content", each [Attributes]?[Hidden]? <> true),
  #"Invoke custom function" = Table.AddColumn(#"Filtered hidden files", "Transform file", each #"Transform file"([Content.1])),
  #"Removed other columns" = Table.SelectColumns(#"Invoke custom function", {"Name.1", "Folder Path", "Transform file"}),
  #"Expanded table column" = Table.ExpandTableColumn(#"Removed other columns", "Transform file", Table.ColumnNames(#"Transform file"(#"Sample file"))),
  #"Renamed columns" = Table.RenameColumns(#"Expanded table column", {{"Name.1", "source_path"}, {"Folder Path", "folder_path"}}),
  #"Changed column type" = Table.TransformColumnTypes(#"Renamed columns", {{"crawl_date", type date}, {"forecast_date", type date}, {"source_path", type text}, {"crawl_time", type text}, {"city", type text}, {"sunrise", type text}, {"sunset", type text}, {"weather_main", type text}, {"weather_desc", type text}, {"temp_min(°C)", type number}, {"temp_max(°C)", type number}, {"morning_temp(°C)", type number}, {"day_temp(°C)", type number}, {"evening_temp(°C)", type number}, {"night_temp(°C)", type number}, {"pressure(hPa)", type number}, {"humidity(%)", type number}, {"wind_speed(m/s)", type number}, {"wind_deg(°)", type number}, {"cloudiness(%)", type number}, {"pop(%)", type number}, {"rain(mm)", type number}})
in
  #"Changed column type"
```

### 2. Gold Layer – `forecast_weather_gold` 
```m
let
    Source = forecast_weather_silver,
    ChangedType = Table.TransformColumnTypes(Source, {{"crawl_time", type time}}),
    LatestDate = List.Max(ChangedType[crawl_date]),
    EarliestTime = List.Min(Table.SelectRows(ChangedType, each [crawl_date] = LatestDate)[crawl_time]),
    FilteredRows = Table.SelectRows(
        ChangedType,
        each [crawl_date] = LatestDate and [crawl_time] = EarliestTime
    ),
    AddedDays = Table.AddColumn(
        FilteredRows,
        "Days",  each let
        TodayDate = Date.From(DateTime.LocalNow()),
        TomorrowDate = Date.AddDays(TodayDate, 1)
    in
        if [forecast_date] = TodayDate then "Today"
        else if [forecast_date] = TomorrowDate then "Tomorrow"
        else Date.ToText([forecast_date], "ddd"),
    type text
),
    AddedSortColumn = Table.AddColumn(
        AddedDays,
        "DaySort",
        each if [Days] = "Today" then 1
             else if [Days] = "Tomorrow" then 2
             else Date.DayOfWeek([forecast_date], Day.Monday) + 3,
        Int64.Type
    ),
  #"Choose columns" = Table.SelectColumns(AddedSortColumn, {"crawl_date", "city", "forecast_date", "sunrise", "sunset", "temp_min(°C)", "temp_max(°C)", "morning_temp(°C)", "day_temp(°C)", "evening_temp(°C)", "night_temp(°C)", "pressure(hPa)", "humidity(%)", "wind_speed(m/s)", "wind_deg(°)", "cloudiness(%)", "weather_main", "weather_desc", "pop(%)", "rain(mm)", "Days", "DaySort"})
in
#"Choose columns"
```
---
### 3. Silver Layer – `hourly_weather_silver`
```m
let
  Source = Lakehouse.Contents(null),
  Navigation = Source{[workspaceId = "e406e233-b34a-489d-8cd5-8851d4e2a74a"]}[Data],
  #"Navigation 1" = Navigation{[lakehouseId = "5bf8e2ae-a8c7-41ec-ad55-1c807526ffa9"]}[Data],
  #"Navigation 2" = #"Navigation 1"{[Id = "Files", ItemKind = "Folder"]}[Data],
  #"Filtered rows" = Table.SelectRows(#"Navigation 2", each ([Name] = "hourly_data_bronze")),
  #"Expanded Content" = Table.ExpandTableColumn(#"Filtered rows", "Content", {"Content", "Name"}, {"Content.1", "Name.1"}),
  #"Filtered hidden files" = Table.SelectRows(#"Expanded Content", each [Attributes]?[Hidden]? <> true),
  #"Invoke custom function" = Table.AddColumn(#"Filtered hidden files", "Transform file (2)", each #"Transform file (2)"([Content.1])),
  #"Removed other columns" = Table.SelectColumns(#"Invoke custom function", {"Name.1", "Folder Path", "Transform file (2)"}),
  #"Expanded table column" = Table.ExpandTableColumn(#"Removed other columns", "Transform file (2)", Table.ColumnNames(#"Transform file (2)"(#"Sample file (2)"))),
  #"Renamed columns" = Table.RenameColumns(#"Expanded table column", {{"Folder Path", "folder_path"}, {"Name.1", "source_file"}}),
  #"Split column by delimiter" = Table.SplitColumn(Table.TransformColumnTypes(#"Renamed columns", {{"crawl_time", type text}}), "crawl_time", Splitter.SplitTextByDelimiter(" "), {"crawl_date", "crawl_time"}),
  #"Split column by delimiter 1" = Table.SplitColumn(Table.TransformColumnTypes(#"Split column by delimiter", {{"forecast_time", type text}}), "forecast_time", Splitter.SplitTextByDelimiter(" "), {"forecast_date", "forecast_time"}),
  #"Changed column type" = Table.TransformColumnTypes(#"Split column by delimiter 1", {{"crawl_time", type text}, {"city", type text}, {"forecast_time", type text}, {"temp_c", type number}, {"feels_like_c", type number}, {"temp_min_c", type number}, {"temp_max_c", type number}, {"pressure_hpa", Int64.Type}, {"humidity_pct", Int64.Type}, {"wind_speed_ms", type number}, {"wind_deg", Int64.Type}, {"cloudiness_pct", Int64.Type}, {"weather_main", type text}, {"weather_desc", type text}, {"pop", type number}, {"rain_3h_mm", type number}, {"crawl_date", type date}, {"source_file", type text}, {"forecast_date", type date}}),
  #"Filtered rows 1" = Table.SelectRows(#"Changed column type", each [crawl_date] <> null and [crawl_date] <> "")
in
  #"Filtered rows 1"
```

### 4. Gold Layer – `hourly_weather_gold`
```m
let
    Source = hourly_weather_silver,
    ChangedType = Table.TransformColumnTypes(Source, {{"crawl_time", type time}}),
    LatestDate = List.Max(ChangedType[crawl_date]),
    EarliestTime = List.Min(Table.SelectRows(ChangedType, each [crawl_date] = LatestDate)[crawl_time]),
    #"Filtered rows" = Table.SelectRows(
        ChangedType,
        each [crawl_date] = LatestDate and [crawl_time] = EarliestTime
    ),
    #"Added SortCol" = Table.AddColumn(
    #"Filtered rows",
    "hour_sort",
    each Time.Hour(Time.FromText([forecast_time])),
    Int64.Type
),
    Format_time = Table.TransformColumns(
        #"Added SortCol",
        {"forecast_time", each Time.ToText(Time.From(_), "HH:mm"), type text}
    ),

    #"Choose columns" = Table.SelectColumns(
        Format_time,
        {"crawl_date", "city", "forecast_time", "hour_sort", "temp_c", "feels_like_c", "temp_min_c",
         "temp_max_c", "pressure_hpa", "humidity_pct", "wind_speed_ms", "wind_deg",
         "cloudiness_pct", "weather_main", "weather_desc", "pop", "rain_3h_mm"}
    )
in
    #"Choose columns"
```
### 5. Silver Layer – `current_weather_silver`
```m
let
  Source = Lakehouse.Contents(null),
  Navigation = Source{[workspaceId = "e406e233-b34a-489d-8cd5-8851d4e2a74a"]}[Data],
  #"Navigation 1" = Navigation{[lakehouseId = "5bf8e2ae-a8c7-41ec-ad55-1c807526ffa9"]}[Data],
  #"Navigation 2" = #"Navigation 1"{[Id = "Files", ItemKind = "Folder"]}[Data],
  #"Filtered rows" = Table.SelectRows(#"Navigation 2", each ([Name] = "weather_data_bronze")),
  #"Expanded Content" = Table.ExpandTableColumn(#"Filtered rows", "Content", {"Content", "Name"}, {"Content.1", "Name.1"}),
  #"Filtered hidden files" = Table.SelectRows(#"Expanded Content", each [Attributes]?[Hidden]? <> true),
  #"Invoke custom function" = Table.AddColumn(#"Filtered hidden files", "Transform file (3)", each #"Transform file (3)"([Content.1])),
  #"Removed other columns" = Table.SelectColumns(#"Invoke custom function", {"Name.1", "Folder Path", "Transform file (3)"}),
  #"Expanded table column" = Table.ExpandTableColumn(#"Removed other columns", "Transform file (3)", Table.ColumnNames(#"Transform file (3)"(#"Sample file (3)"))),
  #"Renamed columns" = Table.RenameColumns(#"Expanded table column", {{"Folder Path", "folder_path"}, {"Name.1", "source_file"}}),
  #"Split column by delimiter" = Table.SplitColumn(Table.TransformColumnTypes(#"Renamed columns", {{"date_time", type text}}), "date_time", Splitter.SplitTextByDelimiter(" "), {"crawl_date", "crawl_time"}),
  #"Split column by delimiter 1" = Table.SplitColumn(Table.TransformColumnTypes(#"Split column by delimiter", {{"sunrise_ist", type text}}), "sunrise_ist", Splitter.SplitTextByDelimiter(" "), {"sunrise_date", "sunrise_time"}),
  #"Split column by delimiter 2" = Table.SplitColumn(Table.TransformColumnTypes(#"Split column by delimiter 1", {{"sunset_ist", type text}}), "sunset_ist", Splitter.SplitTextByDelimiter(" "), {"sunset_date", "sunset_time"}),
  #"Changed column type" = Table.TransformColumnTypes(#"Split column by delimiter 2", {{"city", type text}, {"temperature(°C)", type number}, {"feels_like(°C)", type number}, {"temp_min(°C)", type number}, {"temp_max(°C)", type number}, {"pressure(hPa)", Int64.Type}, {"humidity(%)", Int64.Type}, {"visibility(m)", Int64.Type}, {"wind_speed(m/s)", type number}, {"wind_deg(°)", Int64.Type}, {"cloudiness(%)", Int64.Type}, {"weather_main", type text}, {"weather_description", type text}, {"crawl_date", type date}, {"crawl_time", type text}, {"sunrise_date", type date}, {"sunrise_time", type text}, {"sunset_date", type date}, {"sunset_time", type text}, {"source_file", type text}})
in
  #"Changed column type"
```

### 2. Gold Layer – `current_weather_gold`
```m
let
  Source = current_weather_silver,
  #"Changed Type" = Table.TransformColumnTypes(Source, {{"crawl_time", type time}}),
  #"Added crawl_datetime" = Table.AddColumn(
        #"Changed Type", 
        "crawl_datetime", 
        each #datetime(
            Date.Year([crawl_date]), 
            Date.Month([crawl_date]), 
            Date.Day([crawl_date]), 
            Time.Hour([crawl_time]), 
            Time.Minute([crawl_time]), 
            Time.Second([crawl_time])
        ),
        type datetime
    ),
  MaxDateTime = List.Max(#"Added crawl_datetime"[crawl_datetime]),
  FinalFiltered = Table.SelectRows(#"Added crawl_datetime", each [crawl_datetime] = MaxDateTime),
  Formatting = Table.TransformColumns(FinalFiltered,
      {{"crawl_time", each Time.ToText(Time.From(_), "HH:mm"), type text},
      {"sunrise_time", each Time.ToText(Time.From(_), "HH:mm"), type text},
        {"sunset_time", each Time.ToText(Time.From(_), "HH:mm"), type text}}),
  #"Inserted day name" = Table.AddColumn(Formatting, "Day name", each Date.DayOfWeekName([crawl_date]), type nullable text),
  #"Capitalized each word" = Table.TransformColumns(#"Inserted day name", {{"weather_description", each Text.Proper(_), type nullable text}}),
  #"Choose columns" = Table.SelectColumns(#"Capitalized each word", {"crawl_date", "crawl_time", "city", "temperature(°C)", "feels_like(°C)", "temp_min(°C)", "temp_max(°C)", "pressure(hPa)", "humidity(%)", "visibility(m)", "wind_speed(m/s)", "wind_deg(°)", "cloudiness(%)", "weather_main", "weather_description", "sunrise_time", "sunset_time", "Day name"})
in
  #"Choose columns"
```

### ✅ Output Tables
- weather_data_silver → Full dataset with lineage & file metadata
- weather_data_gold → Clean analytical dataset ready for BI

### 📊 Usage
- Connect Power BI directly to the Gold table for reporting.
- Use Silver for debugging, lineage, and data quality checks.
- Extend the pipeline by adding a Platinum layer (aggregations, KPIs).
in
#"Changed column type"
