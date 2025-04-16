# P5-Data_Exploration_SQL



# Advanced SQL Project: Green Supply Chain Analysis

This repository contains advanced SQL queries for analyzing a sustainability dataset. The project focuses on solving complex problems related to renewable energy usage, CO2 emissions, and other sustainability metrics.

---

## Table of Contents
1. [Introduction](#introduction)
2. [Dataset Description](#dataset-description)
3. [Understanding business through SQL data exploration](#Understanding-business-through-SQL-data-exploration)
   - [1. Hierarchical Material Efficiency](#1-hierarchical-material-efficiency)
   - [2. Nested Sustainability Tiers](#2-nested-sustainability-tiers)
   - [3. Recursive Supply Chain Impact](#3-recursive-supply-chain-impact)
   - [4. Pivoted Renewable Energy Analysis](#4-pivoted-renewable-energy-analysis)
   - [5. Optimized Pagination for Large Datasets](#5-optimized-pagination-for-large-datasets)
   - [6. Temporal Sustainability Trends](#6-temporal-sustainability-trends)
   - [7. Anti-Join for Anomaly Detection](#7-anti-join-for-anomaly-detection)
   - [8. Dynamic Threshold Optimization](#8-dynamic-threshold-optimization)
   - [9. Materialized View for Frequent Aggregates](#9-materialized-view-for-frequent-aggregates)
   - [10. Multi-Criteria Indexing Challenge](#10-multi-criteria-indexing-challenge)
4. [How to Run Queries](#how-to-run-queries)

---

## Introduction

This project demonstrates the use of advanced SQL techniques to analyze a sustainability dataset. The queries address real-world challenges such as anomaly detection, recursive calculations, and dynamic optimizations.

---

## Dataset Description

The dataset includes the following columns:
- **ID**: Unique identifier for each product.
- **Product_Type**: Type of product (e.g., Pharmaceutical, Automotive).
- **Raw_Material_Usage_kg**: Raw material usage in kilograms.
- **Energy_Consumption_kWh**: Energy consumption in kilowatt-hours.
- **Waste_Generated_kg**: Waste generated in kilograms.
- **Transport_Distance_km**: Transport distance in kilometers.
- **CO2_Emissions_kg**: CO2 emissions in kilograms.
- **Renewable_Energy_Percentage**: Percentage of energy from renewable sources.
- **Cost_USD**: Manufacturing cost in USD.
- **Delivery_Time_days**: Delivery time in days.
- **Sustainability_Score**: Sustainability score (0–100).

---

## Understanding business through SQL data exploration

### 1. Hierarchical Material Efficiency
Identify products where raw material usage exceeds the average of their product type, and rank them within their industry using a moving average of CO2 emissions.

```sql
WITH ProductAverages AS (
    SELECT 
        Product_Type,
        AVG(Raw_Material_Usage_kg) AS Avg_Material_Usage
    FROM green_supply_chain
    GROUP BY Product_Type
)
SELECT 
    g.ID,
    g.Product_Type,
    g.Raw_Material_Usage_kg,
    AVG(g.CO2_Emissions_kg) OVER (
        PARTITION BY g.Product_Type 
        ORDER BY g.ID 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS Moving_Avg_CO2
FROM green_supply_chain g
JOIN ProductAverages pa ON g.Product_Type = pa.Product_Type
WHERE g.Raw_Material_Usage_kg > pa.Avg_Material_Usage;
```
*Sample result*

```      ID    Product_Type  Raw_Material_Usage_kg  Moving_Avg_CO2
0     16         Apparel             169.390636      861.245738
1     21         Apparel             122.342333      887.493884
2     44         Apparel             130.130215      695.092154
3     58         Apparel             180.630315      533.164084
4     65         Apparel             157.350247      443.304000
..   ...             ...                    ...             ...
500  953  Pharmaceutical             154.455066      741.284390
501  961  Pharmaceutical             192.319562      657.310585
502  971  Pharmaceutical             150.097845      641.873253
503  976  Pharmaceutical             159.153143      558.100330
504  988  Pharmaceutical             131.068281      392.425161

[505 rows x 4 columns]
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### 2. Nested Sustainability Tiers
Classify products into tiers (Gold, Silver, Bronze) based on renewable energy usage percentiles and calculate the median cost for each tier.

```sql
WITH EnergyTiers AS (
    SELECT 
        *,
        NTILE(3) OVER (ORDER BY Renewable_Energy_Percentage DESC) AS Tier
    FROM green_supply_chain
)
SELECT 
    CASE Tier 
        WHEN 1 THEN 'Gold' 
        WHEN 2 THEN 'Silver' 
        ELSE 'Bronze' 
    END AS Tier_Group,
    PERCENTILE_CONT(0.5) WITHIN GROUP (ORDER BY Cost_USD) AS Median_Cost
FROM EnergyTiers
GROUP BY Tier;
```

*Sample result*

```
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### 3. Recursive Supply Chain Impact
Calculate cumulative CO2 emissions for products with transport distances forming a chain (each product’s transport distance ≥ previous product’s distance).

```sql
WITH RECURSIVE EmissionChain AS (
    SELECT 
        ID,
        Transport_Distance_km,
        CO2_Emissions_kg AS Cumulative_CO2
    FROM green_supply_chain
    UNION ALL
    SELECT 
        g.ID,
        g.Transport_Distance_km,
        ec.Cumulative_CO2 + g.CO2_Emissions_kg
    FROM green_supply_chain g
    JOIN EmissionChain ec ON g.Transport_Distance_km >= ec.Transport_Distance_km
)
SELECT MAX(Cumulative_CO2) AS Max_Chain_Impact FROM EmissionChain;
```

*Sample result*

```
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### 4. Pivoted Renewable Energy Analysis
Display renewable energy percentages across industries in a cross-tab format.

```sql
SELECT 
    Product_Type,
    COUNT(CASE WHEN Renewable_Energy_Percentage  60 THEN 1 END) AS High_Renewables
FROM green_supply_chain
GROUP BY Product_Type;
```

*Sample result*

```
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### 5. Optimized Pagination for Large Datasets

```sql
SELECT *
FROM green_supply_chain
WHERE Waste_Generated_kg > (SELECT AVG(Waste_Generated_kg) FROM green_supply_chain)
AND ID > (SELECT ID FROM green_supply_chain ORDER BY ID LIMIT 1 OFFSET 99)
ORDER BY ID
LIMIT 10;
```

*Sample result*

```
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### **6. Temporal Sustainability Trends**
*Compare quarterly average sustainability scores using generated dates (if no date column, simulate quarters using ID ranges).*  
```sql
WITH SimulatedQuarters AS (
    SELECT *,
        CASE 
            WHEN ID % 4 = 0 THEN 'Q4' 
            WHEN ID % 3 = 0 THEN 'Q3' 
            WHEN ID % 2 = 0 THEN 'Q2' 
            ELSE 'Q1' 
        END AS Quarter
    FROM green_supply_chain
)
SELECT 
    Quarter,
    AVG(Sustainability_Score) AS Avg_Score,
    LAG(AVG(Sustainability_Score)) OVER (ORDER BY MIN(ID)) AS Prev_Quarter_Score
FROM SimulatedQuarters
GROUP BY Quarter;
```
*Sample result*

```
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### **7. Anti-Join for Anomaly Detection**
*Find products with below-average sustainability scores but above-average renewable energy usage.*  
```sql
SELECT *
FROM green_supply_chain g
WHERE g.Sustainability_Score  40 OR Cost_USD < 5000)
ORDER BY Sustainability_Score DESC;
```
  
```sql
CREATE INDEX idx_product_renewable_cost ON green_supply_chain 
(Product_Type, Renewable_Energy_Percentage, Cost_USD, Sustainability_Score);
```

*Sample result*

```
```
The full result of this query is available [here](results/recursive_supply_chain.csv).

---

### 8. Dynamic Threshold Optimization
*Create a function to calculate energy efficiency benchmarks that adjust based on product type.*

```sql
DELIMITER //
CREATE FUNCTION GetEnergyBenchmark(product_type VARCHAR(50)) 
RETURNS DECIMAL(10,2)
DETERMINISTIC
BEGIN
    DECLARE benchmark DECIMAL(10,2);
    SELECT AVG(Energy_Consumption_kWh) * 0.85 INTO benchmark
    FROM green_supply_chain
    WHERE Product_Type = product_type;
    RETURN benchmark;
END //
DELIMITER ;

SELECT ID, Energy_Consumption_kWh, GetEnergyBenchmark(Product_Type) AS Benchmark
FROM green_supply_chain;
```

*Sample result*
```
```
The full result of this query is available [here](results/energy_efficiency_benchmarks.csv).

---

### 9. Materialized View for Frequent Aggregates
*Precompute and refresh daily stats for high-traffic sustainability dashboards.*

```sql
CREATE MATERIALIZED VIEW DailySustainability AS
SELECT 
    Product_Type,
    COUNT(*) AS Product_Count,
    AVG(CO2_Emissions_kg) AS Avg_CO2,
    SUM(Renewable_Energy_Percentage) AS Total_Renewables
FROM green_supply_chain
GROUP BY Product_Type;
```

*Sample result*
```
```
The full result of this query is available [here](results/daily_sustainability.csv).

---

### 10. Multi-Criteria Indexing Challenge
*Optimize this slow-running query without changing its logic:*

**Problem Query**
```sql
SELECT *
FROM green_supply_chain
WHERE Product_Type = 'Pharmaceutical'
AND (Renewable_Energy_Percentage > 40 OR Cost_USD < 5000)
ORDER BY Sustainability_Score DESC;
```

**Solution**
```sql
CREATE INDEX idx_product_renewable_cost ON green_supply_chain 
(Product_Type, Renewable_Energy_Percentage, Cost_USD, Sustainability_Score);
```

*Sample result*
```
```
The full result of this query is available [here](results/multi_criteria_indexing.csv).

---

---
Antwort von Perplexity: pplx.ai/share


---

## How to Run Queries

1. Clone this repository:
   ```bash
   git clone https://github.com/Sudharsanan-Ravichandran/P5-Green_Supply_Chain_SQL.git
   cd P5-Green_Supply_Chain_SQL
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


