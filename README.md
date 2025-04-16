# P6-Sustainable_Manufacturing_SQL



# Advanced SQL Project: Sustainable Manufacturing Analysis

This repository contains a collection of advanced SQL queries designed to analyze and optimize sustainable manufacturing operations. The queries help track material efficiency, energy and water usage, emissions, productivity, and more, enabling data-driven decisions for improved sustainability.

---

## Table of Contents
1. [Introduction](#introduction)
2. [Dataset Description](#dataset-description)
3. [Understanding business through SQL data exploration](#Understanding-business-through-SQL-data-exploration)
   - [Efficiency Analysis](#1-efficiency-analysis)
   - [Sustainability Metrics](#2-sustainability-metrics)
   - [Time-Based Analysis](#3-time-based-analysis)
   - [Comparative Analysis](#4-comparative-analysis)
   - [Anomaly Detection](#5-anomaly-detection)
   - [Advanced Challenges](#6-advanced-challenges)

4. [How to Run Queries](#how-to-run-queries)

---

## Introduction

Modern manufacturing faces increasing pressure to improve efficiency and reduce environmental impact. This project provides a suite of SQL queries for analyzing key sustainability metrics using a hypothetical sustainable_manufacturing database. The queries are intended for use by data analysts, engineers, and sustainability managers.

---

## Dataset Description



The dataset includes the following columns:

- **Machine_ID**: Unique identifier for each machine.
- **Date**: Date of the recorded data (YYYY-MM-DD).
- **Material_Used_kg**: Total raw material used by the machine on the given day (in kilograms).
- **Material_Waste_kg**: Amount of material wasted by the machine on the given day (in kilograms).
- **Product_Output_Units**: Number of finished product units produced by the machine on the given day.
- **Energy_Consumption_kWh**: Total energy consumed by the machine on the given day (in kilowatt-hours).
- **Water_Consumption_Liters**: Total water consumed by the machine on the given day (in liters).
- **Water_Recycled_Liters**: Amount of water recycled by the machine on the given day (in liters).
- **CO2_Emission_kg**: Total CO₂ emissions generated by the machine on the given day (in kilograms).

**Each row represents one machine’s operational and environmental data for a single day.**



---

## Understanding business through SQL data exploration

### 1. Efficiency Analysis
**a. Material Utilization**
Calculate daily material efficiency (%) for each machine


```sql
SELECT Machine_ID, Date,
       CASE 
           WHEN (Material_Used_kg - Material_Waste_kg) > 0 
           THEN ROUND((Product_Output_Units / (Material_Used_kg - Material_Waste_kg)) * 100, 2)
           ELSE NULL 
       END AS Material_Efficiency
FROM sustainable_manufacturing
WHERE Material_Used_kg > 0;
```
*Sample result*

```       
    Machine_ID        Date Material_Efficiency
0          M001  01.01.2020              100.00
1          M001  01.01.2020              100.00
2          M001  01.01.2020              100.00
3          M001  01.01.2020              100.00
4          M001  01.01.2020              100.00
...         ...         ...                 ...
9995       M010  10.01.2020               99.48
9996       M010  10.01.2020               99.48
9997       M010  10.01.2020               99.48
9998       M010  10.01.2020               99.48
9999       M010  10.01.2020               99.48

[10000 rows x 3 columns]
```
The full result of this query is available [here](Assets/results/material_efficiency.csv).

---

**b. Energy Productivity**
Find machines with energy consumption per output unit > 2kWh/unit.


```sql
WITH energy_stats AS (
    SELECT Machine_ID,
           Energy_Consumption_kWh / NULLIF(Product_Output_Units, 0) AS Energy_Per_Unit
    FROM sustainable_manufacturing
)
SELECT Machine_ID, Energy_Per_Unit
FROM energy_stats 
WHERE Energy_Per_Unit > 0.2;
```
*Sample result*

```     
        Machine_ID Energy_Per_Unit
0          M001          0.7111
1          M001          0.7111
2          M001          0.7111
3          M001          0.7111
4          M001          0.7111
...         ...             ...
9995       M010          0.6579
9996       M010          0.6579
9997       M010          0.6579
9998       M010          0.6579
9999       M010          0.6579

[10000 rows x 2 columns]
```
The full result of this query is available [here](Assets/results/energy_per_unit.csv).

---

**c. Water Conservation**
Calculate weekly water savings percentage


```sql
SELECT Machine_ID, YEAR(Date) AS Year, WEEK(Date) AS Week,
       ROUND((SUM(Water_Recycled_Liters)/SUM(Water_Consumption_Liters)) * 100, 2) AS Water_Savings_Pct
FROM sustainable_manufacturing
GROUP BY Machine_ID, Year, Week;
```
*Sample result*

```  
Machine_ID  Year  Week Water_Savings_Pct
0       M001  2001     2             20.00
1       M002  2002     3             16.67
2       M003  2003     3             20.00
3       M004  2004     3             14.12
4       M005  2005     3             20.00
5       M006  2006     3             19.57
6       M007  2007     2             17.78
7       M008  2008     3             20.00
8       M009  2009     3             20.00
9       M010  2010     3             17.50
```
The full result of this query is available [here](Assets/results/water_savings.csv).

---

### 2. Sustainability Metrics
**a. CO₂ Intensity Ranking**
Rank machines by CO₂ emissions per product unit

```sql
SELECT Machine_ID,
       RANK() OVER (ORDER BY SUM(CO2_Emission_kg)/NULLIF(SUM(Product_Output_Units),0) DESC) AS Emission_Rank
FROM sustainable_manufacturing
GROUP BY Machine_ID;
```

*Sample result*

```
  Machine_ID  Emission_Rank
0       M003              1
1       M006              2
2       M002              3
3       M008              4
4       M005              5
5       M001              6
6       M007              6
7       M009              6
8       M004              9
9       M010             10
```
The full result of this query is available [here](Assets/results/co2_emissions.csv).

---

**b. Waste Contribution**
Identify machines where material waste exceeds 15% of total material used

```sql
WITH waste AS (
				SELECT machine_id, CAST(SUM(material_waste_kg) AS float) AS total_waste,
                0.10 * (SELECT SUM(material_waste_kg) FROM sustainable_manufacturing) AS ten_percent_waste
                FROM sustainable_manufacturing
                GROUP BY machine_id)

SELECT machine_id, total_waste, ten_percent_waste
FROM waste
WHERE total_waste > 0.10 * (SELECT SUM(material_waste_kg) FROM sustainable_manufacturing)
;
```

*Sample result*

```
  machine_id  total_waste ten_percent_waste
0       M001      50000.0          45500.00
1       M005      55000.0          45500.00
2       M008      60000.0          45500.00
3       M009      50000.0          45500.00
```
The full result of this query is available [here](Assets/results/material_waste.csv).

---

### 3. Time-Based Analysis
**a. Weekly Performance Trends**
Compare machines' productivity changes between first and last weeks of January 2020

```sql
WITH first_week_tbl AS (
SELECT machine_id,
		AVG(energy_consumption / product_output) * 100 AS energy_productivity
FROM sustainable_mfg
WHERE WEEK(date, 3) = (SELECT WEEK("2020-01-01", 3) AS firstweek)
GROUP BY machine_id),

	last_week_tbl AS (

SELECT machine_id,
		AVG(energy_consumption / product_output) * 100 AS energy_productivity
FROM sustainable_mfg
WHERE WEEK(date, 3) = (SELECT WEEK("2020-01-07", 3) AS lastweek)
GROUP BY machine_id)

SELECT first_week_tbl.machine_id AS machine_id, first_week_tbl.energy_productivity AS energy_productivity
FROM first_week_tbl

UNION 

SELECT last_week_tbl.machine_id AS machine_id, last_week_tbl.energy_productivity AS energy_productivity
FROM last_week_tbl
;
```

*Sample result*

```
  machine_id energy_productivity
0       M001         71.11000000
1       M002         70.00000000
2       M003         73.26000000
3       M004         66.67000000
4       M005         70.65000000
5       M006         70.73000000
6       M007         70.37000000
7       M008         70.21000000
8       M009         71.11000000
9       M010         65.79000000
```
The full result of this query is available [here](Assets/results/weekly_comparison.csv).

---

**b. Weekend vs. Weekday**
Compare average energy consumption on weekends vs. weekdays (assuming 5-day work week)

```sql
SELECT "Weekday" AS Day, AVG(energy_consumption) AS Energy_consumption
FROM sustainable_mfg
WHERE WEEKDAY(date) IN (0,4)

UNION

SELECT "Weekend", AVG(energy_consumption)
FROM sustainable_mfg
WHERE WEEKDAY(date) IN (5,6)
;
```

*Sample result*

```
       Day Energy_consumption
0  Weekday           285.0000
1  Weekend           292.5000
```
The full result of this query is available [here](Assets/results/weekend_vs_weekday.csv).

---

### 4. Comparative Analysis
**a. Machine Benchmarking**
List machines performing below department average in both energy efficiency and output

```sql
WITH department_metrics AS (
  SELECT 
    AVG(Product_Output_Units) AS dept_avg_output,
    AVG(Product_Output_Units/NULLIF(Energy_Consumption_kWh,0)) AS dept_avg_energy_eff
  FROM sustainable_manufacturing
),
machine_metrics AS (
  SELECT 
    Machine_ID,
    AVG(Product_Output_Units) AS machine_output,
    AVG(Product_Output_Units/NULLIF(Energy_Consumption_kWh,0)) AS machine_energy_eff
  FROM sustainable_manufacturing
  GROUP BY Machine_ID
)
SELECT m.Machine_ID
FROM machine_metrics m
CROSS JOIN department_metrics d
WHERE m.machine_output < d.dept_avg_output
  AND m.machine_energy_eff < d.dept_avg_energy_eff;
```

*Sample result*

```
  Machine_ID
0       M002
1       M006
2       M007
```
The full result of this query is available [here](Assets/results/machine_benchmarking.csv).

---

**b. Trade-off Identification**
Find machines with top 10% productivity but bottom 25% environmental performance

```sql
WITH productivity AS (
  SELECT Machine_ID,
    PERCENT_RANK() OVER (ORDER BY SUM(Product_Output_Units) DESC) AS prod_rank
  FROM sustainable_manufacturing
  GROUP BY Machine_ID
),
environment AS (
  SELECT Machine_ID,
    PERCENT_RANK() OVER (ORDER BY SUM(CO2_Emission_kg) + SUM(Energy_Consumption_kWh) ASC) AS env_rank
  FROM sustainable_manufacturing
  GROUP BY Machine_ID
)
SELECT p.Machine_ID
FROM productivity p
JOIN environment e USING (Machine_ID)
WHERE p.prod_rank <= 0.10
  AND e.env_rank >= 0.75;
```

*Sample result*

```
  Machine_ID
0       M008
```
The full result of this query is available [here](Assets/results/trade_off.csv).

---

### 5. Anomaly Detection

**a. Output Inconsistencies**
Flag days where a machine's output deviates >2σ from its 7-day moving average

```sql
WITH moving_metrics AS (
  SELECT Machine_ID, Date, Product_Output_Units,
    AVG(Product_Output_Units) OVER (
      PARTITION BY Machine_ID 
      ORDER BY Date 
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS ma_7d,
    STDDEV(Product_Output_Units) OVER (
      PARTITION BY Machine_ID 
      ORDER BY Date 
      ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
    ) AS std_7d
  FROM sustainable_manufacturing
)
SELECT Machine_ID, Date, Product_Output_Units,
  ROUND((Product_Output_Units - ma_7d)/NULLIF(std_7d,0), 2) AS z_score
FROM moving_metrics
WHERE ABS((Product_Output_Units - ma_7d)/NULLIF(std_7d,0)) > 2;
```

*Sample result*

```
Empty DataFrame
Columns: [Machine_ID, Date, Product_Output_Units, z_score]
Index: []
```
The full result of this query is available [here](Assets/results/output_inconsistency.csv).

---

**b. Resource Spikes**
Detect days with simultaneous 30%+ spikes in energy/water use and CO₂ emissions

```sql
WITH resource_spikes AS (
  SELECT Machine_ID, Date,
    CO2_Emission_kg / NULLIF(LAG(CO2_Emission_kg, 1) OVER w, 0) - 1 AS co2_increase,
    Energy_Consumption_kWh / NULLIF(LAG(Energy_Consumption_kWh, 1) OVER w, 0) - 1 AS energy_increase,
    Water_Consumption_Liters / NULLIF(LAG(Water_Consumption_Liters, 1) OVER w, 0) - 1 AS water_increase
  FROM sustainable_manufacturing
  WINDOW w AS (PARTITION BY Machine_ID ORDER BY Date)
)
SELECT Machine_ID, Date,
  ROUND(co2_increase*100,2) AS co2_pct,
  ROUND(GREATEST(energy_increase, water_increase)*100,2) AS resource_pct
FROM resource_spikes
WHERE co2_increase >= 0.3
  AND (energy_increase >= 0.3 OR water_increase >= 0.3);
```

*Sample result*

```
Empty DataFrame
Columns: [Machine_ID, Date, co2_pct, resource_pct]
Index: []
```
The full result of this query is available [here](Assets/results/resource_spikes.csv).

---

### 6. Advanced Challenges

**Sustainability Index**
Create composite score weighting: 40% emissions, 30% energy, 20% water, 10% waste

```sql
WITH normalized AS (
    SELECT Machine_ID,
           (CO2_Emission_kg - MIN(CO2_Emission_kg) OVER()) / 
           (MAX(CO2_Emission_kg) OVER() - MIN(CO2_Emission_kg) OVER()) AS norm_co2,
           (Energy_Consumption_kWh - MIN(Energy_Consumption_kWh) OVER()) / 
           (MAX(Energy_Consumption_kWh) OVER() - MIN(Energy_Consumption_kWh) OVER()) AS norm_energy,
           (Water_Consumption_Liters - MIN(Water_Consumption_Liters) OVER()) / 
           (MAX(Water_Consumption_Liters) OVER() - MIN(Water_Consumption_Liters) OVER()) AS norm_water,
           (Material_Waste_kg - MIN(Material_Waste_kg) OVER()) / 
           (MAX(Material_Waste_kg) OVER() - MIN(Material_Waste_kg) OVER()) AS norm_waste
    FROM sustainable_manufacturing
)
SELECT Machine_ID,
       (0.4 * norm_co2 + 0.3 * norm_energy + 0.2 * norm_water + 0.1 * norm_waste) AS Sustainability_Index
FROM normalized;
```
*Sample result*

```
     Machine_ID Sustainability_Index
0          M001              0.77584
1          M001              0.77584
2          M001              0.77584
3          M001              0.77584
4          M001              0.77584
...         ...                  ...
9995       M010              0.01200
9996       M010              0.01200
9997       M010              0.01200
9998       M010              0.01200
9999       M010              0.01200

[10000 rows x 2 columns]
```
The full result of this query is available [here](Assets/results/sustainability_index.csv).

---



## How to Run Queries

1. Clone this repository:
   ```bash
   git clone https://github.com/yourusername/sustainability-sql-project.git
   cd sustainability-sql-project
   ```

2. Import the dataset into your database:
   ```sql
   LOAD DATA INFILE '/path/to/green_supply_chain_dataset.csv'
   INTO TABLE green_supply_chain;
   ```

3. Run the queries using your preferred SQL client.

Feel free to contribute by submitting pull requests or opening issues!

--- 

This structure is clear, professional, and GitHub-friendly! You can copy it into your `README.md` file and push it to your GitHub repository following standard Git commands (`git add .`, `git commit`, `git push`). Let me know if you'd like further assistance!


