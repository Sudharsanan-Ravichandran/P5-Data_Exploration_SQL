# P5-Data_Exploration_SQL

To include the 10 advanced SQL questions and their solutions in a GitHub project, you can format them in a `README.md` file using **GitHub Flavored Markdown**. Below is an example of how to structure the content:

---

# Advanced SQL Project: Sustainability Dataset Analysis

This repository contains advanced SQL queries for analyzing a sustainability dataset. The project focuses on solving complex problems related to renewable energy usage, CO2 emissions, and other sustainability metrics.

---

## Table of Contents
1. [Introduction](#introduction)
2. [Dataset Description](#dataset-description)
3. [Advanced SQL Questions](#advanced-sql-questions)
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

## Advanced SQL Questions

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


To include the results of the SQL queries in your GitHub repository, you can use **Markdown tables** or **code blocks** within the `README.md` file. Here's how you can format the results:

---

### **1. Use Markdown Tables for Results**

Markdown tables are an excellent way to display query results. Here's an example:

#### Query: Hierarchical Material Efficiency
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

**Result:**
| ID   | Product_Type   | Raw_Material_Usage_kg | Moving_Avg_CO2 |
|------|----------------|------------------------|----------------|
| 101  | Automotive     | 120.5                 | 75.3           |
| 102  | Automotive     | 130.7                 | 80.1           |
| 201  | Pharmaceutical | 90.2                  | 50.6           |

---

### **2. Use Code Blocks for Large Results**

For larger result sets, you can include them as code blocks to maintain readability:

#### Query: Pivoted Renewable Energy Analysis
```sql
SELECT 
    Product_Type,
    COUNT(CASE WHEN Renewable_Energy_Percentage  60 THEN 1 END) AS High_Renewables
FROM green_supply_chain
GROUP BY Product_Type;
```

**Result:**
```plaintext
Product_Type       Low_Renewables   Medium_Renewables   High_Renewables
Automotive         15               25                 10
Pharmaceutical     5                10                 20
Food               8                12                 15
```

---

### **3. Include CSV/JSON Files for Full Results**

If the query results are too large to include directly in the `README.md`, you can:

1. Save the results as a CSV or JSON file.
2. Add these files to your repository in a `results/` directory.
3. Reference them in your `README.md` file with links.

Example:
- Save the result of "Recursive Supply Chain Impact" as `results/recursive_supply_chain.csv`.
- Reference it in the README:
   ```markdown
   **Query: Recursive Supply Chain Impact**

   The full result of this query is available [here](results/recursive_supply_chain.csv).
   ```

---

### **4. Combine Queries and Results in Separate Files**

For a cleaner structure, you can create a `queries/` folder for SQL scripts and a `results/` folder for their outputs.

Example directory structure:
```
sustainability-sql-project/
├── README.md
├── queries/
│   ├── hierarchical_efficiency.sql
│   ├── pivoted_analysis.sql
│   └── recursive_supply_chain.sql
├── results/
│   ├── hierarchical_efficiency.csv
│   ├── pivoted_analysis.csv
│   └── recursive_supply_chain.csv
```

In the README, link to these files:
```markdown
### Query: Hierarchical Material Efficiency

SQL Script: [hierarchical_efficiency.sql](queries/hierarchical_efficiency.sql)  
Result: [hierarchical_efficiency.csv](results/hierarchical_efficiency.csv)
```

---

### **5. Automate Result Generation**

If you're running these queries frequently, consider automating result generation using a script (e.g., Python or Bash). For example:

- Use Python with `pymysql` or `sqlalchemy` to execute queries and save results as CSVs.
- Include the script in your GitHub repository:
   ```
   python generate_results.py
   ```

---

By combining these approaches, you can make your GitHub project professional and easy to navigate while keeping it scalable for large datasets or complex queries!

---
Antwort von Perplexity: pplx.ai/share
