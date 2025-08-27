# 🌦 Weather Data Collection with Microsoft Fabric Notebook

This project uses a **Microsoft Fabric Notebook** with Python code to collect real-time weather data from the **OpenWeather API** for major Indian cities, and saves the results into **OneLake (Lakehouse Files)** as CSV files.

---

## 📜 What the Code Does

1. **Import Libraries**  
   - Uses `requests` to call the OpenWeather API  
   - Uses `pandas` to process JSON response into tabular format  
   - Uses `datetime` to add a timestamp for each run  

2. **API Configuration**  
   - Connects to the OpenWeather API using an API key  
   - Queries weather data for a predefined list of Indian cities  

3. **Data Extraction**  
   - Extracts temperature, humidity, wind speed, cloudiness, and weather description from the API response  
   - Stores all values in a structured Python dictionary  

4. **DataFrame Creation**  
   - Converts the list of dictionaries into a Pandas DataFrame for easier handling  

5. **Saving to OneLake**  
   - Saves the DataFrame as a CSV file into:  

```
/lakehouse/default/Files/weather_data_bronze/
```

   - File name includes the current date & time for uniqueness  

---

## 📊 Example Output

| date_time           | city     | temperature(°C) | humidity(%) | wind_speed(m/s) | weather_main | weather_description |
|---------------------|----------|-----------------|-------------|-----------------|--------------|---------------------|
| 2025-08-26 18:30:21 | Delhi    | 32.5            | 65          | 3.5             | Clouds       | scattered clouds    |
| 2025-08-26 18:30:21 | Mumbai   | 29.2            | 78          | 5.2             | Rain         | light rain          |

---

## ⚙️ How to Run

1. Open the notebook in Microsoft Fabric  
2. Replace the placeholder API key with your **OpenWeather API key**  
3. Run all cells to fetch data and save it to OneLake  

---
