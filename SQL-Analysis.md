## Energy Generation  
Identify regions that are net energy producers. Not all regions generate enough energy to meet the local demand. Some regions purchase power from other regions, while others sell their surplus to regions in need. 

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





