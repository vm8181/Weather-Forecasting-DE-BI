# ⛅ Data Pipeline - Microsoft Fabric
<img width="1817" height="550" alt="image" src="https://github.com/user-attachments/assets/45939741-cf0a-43b0-86a5-7331e7dd0833" />
<img width="1814" height="513" alt="image" src="https://github.com/user-attachments/assets/562170b1-c78b-439b-9997-d6c9197b635b" />

This documents the **weather_datapipeline, hourly_weather_pipeline** built in **Microsoft Fabric Data Pipelines**.  
The pipeline orchestrates the collection, transformation, and storage of real-time weather data using **Notebook + Dataflow + Wait activity**.

---

## 🔄 Pipeline Overview

The pipeline `WeatherHourlyPipeline` automates the following sequence:

1. **Notebook (Data_Crawler)**  
   - Executes Python code to fetch weather data from the OpenWeather API  
   - Saves the results into **Lakehouse (Bronze Layer)** as timestamped CSV files  

2. **Dataflow (Refresh_Weather_Dataflow)**  
   - Cleans and refreshes the Bronze dataset  
   - Prepares the data for downstream reporting (Silver/Gold layers)  

3. **Wait Activity**  
   - Ensures that data refresh is completed before triggering dependent activities  
   - Helps in scheduling periodic (hourly/daily) data pulls  

---

## 📂 Pipeline Structure (Visual)

```
Notebook (Data_Crawler) ---> Dataflow (Refresh_Weather_Dataflow) ---> Wait
```

---

## 🛠 Features

- **Automated Data Collection**: No manual intervention required once scheduled  
- **Orchestration**: Manages execution order between notebook, dataflow, and wait steps  
- **Scalability**: Can be extended to include more cities, different APIs, or multiple transformations  
- **Integration Ready**: Data is stored in Fabric Lakehouse and ready for **Power BI dashboards**  

---

## ⏲ Scheduling

- The pipeline can be triggered **hourly, daily, or weekly**  
- Example: For near real-time dashboards, schedule every **1 hour**  

---

## 📊 Use Case

- Build **weather analysis dashboards** in Power BI  
- Analyze **temperature, humidity, and rainfall patterns** across major Indian cities  
- Enable **decision-making** for logistics, travel, and agriculture industries  
