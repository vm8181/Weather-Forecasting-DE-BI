# 🌦 Weather Data Pipeline with Microsoft Fabric Notebook

This project demonstrates how to build a **real-time weather data pipeline** using **Microsoft Fabric Notebook (Dataflow Gen2)**.  
The pipeline fetches weather data for major Indian cities using the **OpenWeather API**, processes it with **Python**, and stores the results into **OneLake (Lakehouse Files)** in CSV format.

---

## 🚀 Project Overview

- **Source**: OpenWeather API (`https://api.openweathermap.org/data/2.5/weather`)  
- **Technology**: Microsoft Fabric (Notebook + Dataflow Gen2 + OneLake)  
- **Data Format**: CSV (Bronze Layer)  
- **Use Case**: Enables downstream analytics in **Power BI / Fabric Warehouse** for real-time weather dashboards.  

---

## 📂 Pipeline Flow

1. **Notebook Execution (Python in Fabric)**  
   - Connects to the OpenWeather API using a personal API key.  
   - Fetches weather data for a predefined list of cities.  
   - Extracts and structures the response into a Pandas DataFrame.  

2. **Data Lakehouse Storage**  
   - Saves processed weather data into **OneLake** under the Bronze folder.  
   - Each file is timestamped for historical tracking.  

3. **Integration with Power BI**  
   - Weather data can be directly queried from Fabric Lakehouse or connected to Power BI for live dashboards.  

---

## 📊 Example Dataset

Sample output for a single run:

| date_time           | city     | temperature(°C) | humidity(%) | wind_speed(m/s) | weather_main | weather_description |
|---------------------|----------|-----------------|-------------|-----------------|--------------|---------------------|
| 2025-08-26 18:30:21 | Delhi    | 32.5            | 65          | 3.5             | Clouds       | scattered clouds    |
| 2025-08-26 18:30:21 | Mumbai   | 29.2            | 78          | 5.2             | Rain         | light rain          |
| 2025-08-26 18:30:21 | Bangalore| 27.0            | 70          | 2.1             | Clear        | clear sky           |

---

## 🛠 Setup Instructions

### 1. Prerequisites
- Microsoft Fabric Workspace (with **Lakehouse** enabled)  
- Python environment inside Fabric Notebook  
- API key from [OpenWeather](https://home.openweathermap.org/users/sign_up)  

### 2. Update API Key
In the notebook, replace the placeholder with your API key:

```python
API_KEY = "your_api_key_here"
```

### 3. Define City List
You can customize the city list in the script:

```python
city_list = ["Delhi", "Mumbai", "Bangalore", "Chennai", "Kolkata"]
```

### 4. Save to Lakehouse
Data is automatically saved to:

```
/lakehouse/default/Files/weather_data_bronze/weather_dataset <timestamp>.csv
```

---

## 📂 Project Structure

```
📁 ms-fabric-weather-pipeline
 ┣ 📄 notebook_weather_pipeline.ipynb     # Fabric Notebook script
 ┣ 📄 README.md                           # Project documentation
 ┗ 📂 /Files/weather_data_bronze/         # Bronze layer CSV output
```

---

## 🔮 Next Steps (Silver / Gold)

- **Silver Layer**: Clean and standardize data (city normalization, missing values, etc.)  
- **Gold Layer**: Build KPIs (average temperature, AQI trends, seasonal analysis)  
- **Visualization**: Create Power BI dashboards for real-time monitoring  

---

## 📜 License

This project is licensed under the MIT License - see the [LICENSE](LICENSE) file for details.

---
