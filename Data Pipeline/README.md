# ⛅ Weather Data Pipeline - Microsoft Fabric
<img width="1833" height="500" alt="image" src="https://github.com/user-attachments/assets/c66027f7-b2c0-4242-a94c-4a5011aa5ad9" />

This documents the **WeatherHourlyPipeline** built in **Microsoft Fabric Data Pipelines**.  
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

- Build **weather trend dashboards** in Power BI  
- Analyze **temperature, humidity, and rainfall patterns** across major Indian cities  
- Enable **decision-making** for logistics, travel, and agriculture industries  
