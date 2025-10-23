# ⚡ Regional Energy Analysis — Intel Energy Data Project

This project explores **energy generation patterns across U.S. regions** using SQL.  
It identifies energy surpluses, renewable production leaders, and efficiency insights across regions and time periods.  

---

## SQL Analysis

You can view the **Tableau** visualizations and final recommendation in the separate file: [Tableau Visualizations](tableau-visualizations.md)

---

## Energy Generation  
Identify regions that are net energy producers. Not all regions generate enough energy to meet the local demand. Some regions purchase power from other regions, while others sell their surplus to regions in need. 

---

**Which region has the highest positive total energy?**
```sql
SELECT  
  region,  
  SUM(net_generation - demand) AS total_energy  
FROM  
  intel.energy_data  
GROUP BY  
  region  
ORDER BY  
  total_energy DESC;
```
The Mid-Atlantic region demonstrates the largest energy surplus, generating 31,693,087 units.  


**What are the top two regions for total renewable energy production?**
```sql
SELECT
  region,
  SUM(hydropower_and_pumped_storage + wind + solar) AS renewable_energy_sources
FROM
  intel.energy_data
GROUP BY
  region
ORDER BY
  SUM(hydropower_and_pumped_storage + wind + solar) DESC;
```
The Northwest and Texas regions rank highest in total renewable energy production.


**Which regions differ in ranking when considering total renewable energy versus the percentage of renewable energy?**
```sql
SELECT
  region,
  ROUND(
    (
      SUM(hydropower_and_pumped_storage + wind + solar) / SUM(net_generation)
    ) * 100,
    2
  ) AS renewable_energy_percentage
FROM
  intel.energy_data
GROUP BY
  region
ORDER BY
  renewable_energy_percentage DESC;
```
The top three regions by percentage of renewable energy are Northwest, Central, and California, whereas the top three regions by total renewable energy are Northwest, Texas, and Central. Consequently, California and Texas switch positions between the two rankings: California ranks highly in percentage but not in total energy, while Texas ranks highly in total energy but not in percentage.

---

## Generating New Data by Energy Type
Investigate how renewable energy and fossil fuels trend over time. Create a new column for energy type.
```sql
SELECT
  date,
  region, 
  SUM(hydropower_and_pumped_storage + wind + solar) AS energy_generated_mw,
  'renewable energy' AS energy_type
FROM 
  intel.energy_data
GROUP BY 
  date,
  region;
```

Calculate the fossil fuel generated for each row.
```sql
SELECT
  date,
  region, 
  SUM(all_petroleum_products + coal + natural_gas + nuclear + other_fuel_sources) AS energy_generated_mw
  'fossil fuel' AS energy_type
FROM 
  intel.energy_data
GROUP BY 
  date,
  region;
```

Union the queries together.
```sql
SELECT
  date,
  region, 
  SUM(hydropower_and_pumped_storage + wind + solar) AS energy_generated_mw,
  'renewable energy' AS energy_type
FROM 
  intel.energy_data
GROUP BY 
  date,
  region

UNION

SELECT
  date,
  region, 
  SUM(all_petroleum_products + coal + natural_gas + nuclear + other_fuel_sources) AS energy_generated_mw,
  'fossil fuel' AS energy_type
FROM 
  intel.energy_data
GROUP BY 
  date,
  region;
```

---

## Aggregating Power Plant Data

Join the tables together.
```sql
SELECT
  *
FROM
  intel.power_plants AS p
INNER JOIN
  intel.energy_by_plant AS e
ON
  p.plant_code = e.plant_code;
```

Which region has the most renewable power plants?
```sql
WITH power_plant_table AS (SELECT
  *
FROM
  intel.power_plants AS p
INNER JOIN
  intel.energy_by_plant AS e
ON
  p.plant_code = e.plant_code)

SELECT 
  region,
  COUNT(*) AS n_plants
FROM 
  power_plant_table
WHERE 
  energy_type = 'renewable_energy'
GROUP BY 
  region
ORDER BY 
  n_plants DESC;
```
The Midwest has 234 plants, making it the region with the most renewable power plants.


Return the total number of plants and total energy generated from plants that use solar photovoltaic technology, and group by region. Show regions with 50 or more.
What can we infer about the efficiency of the power plants in the Midwest region compared to the other regions?
```sql
WITH power_plant_table AS (
  SELECT
    *
  FROM
    intel.power_plants AS p
    INNER JOIN intel.energy_by_plant AS e ON p.plant_code = e.plant_code
)
SELECT
  region,
  COUNT(*) AS n_plants,
  SUM(energy_generated_mw) AS total_energy_generated
FROM
  power_plant_table
WHERE
  energy_type = 'renewable_energy'
  AND primary_technology = 'Solar Photovoltaic'
GROUP BY
  region
HAVING
  COUNT(*) >= 50
ORDER BY
  n_plants DESC;
```
The Midwest region has the third-largest number of power plants but produces the least total energy. It generates about 4 million units, averaging roughly 69,000 units per plant, whereas comparable regions generate around 10 million units, or about 150,000 units per plant. This suggests that power plants in the Midwest may be smaller or less efficient than those in other regions. Since we are looking at Solar Photovoltaic as the primary technology, it would be valuable to compare what secondary technologies the plants are using. 

---

## Hourly Trends in Renewable Energy
Investigate how renewable energy generation fluctuates with the time of day.

---

Find how much renewable energy each region produces at each hour of the day.
```sql
SELECT
  DATE_PART('hour', time_at_end_of_hour) AS hour,
  region,
  SUM(hydropower_and_pumped_storage + wind + solar) AS renewable_energy_sources
FROM 
  intel.energy_data
GROUP BY
  region,
  hour
ORDER BY 
  hour 
;
```

Include California and Northwest regions only. 
```sql
SELECT
  DATE_PART('hour', time_at_end_of_hour) AS hour,
  region,
  SUM(hydropower_and_pumped_storage + wind + solar) AS renewable_energy_sources
FROM
  intel.energy_data
WHERE
  region = 'California'
  OR region = 'Northwest'
GROUP BY
  region,
  hour
ORDER BY
  hour;
```
Across all hours of the day, the Northwest consistently generates more renewable energy than California, producing approximately 7–9 million units per hour compared to California’s 2–5 million. Peak generation occurs around hour 18 in the Northwest and hour 14 in California.






