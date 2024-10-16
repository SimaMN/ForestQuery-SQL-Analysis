-- APPENDIX: SQL queries used

-- Total forest area in the world for 1990

-- Create View
```sql

CREATE VIEW forestation AS
SELECT 
    fra.country_code,
    fra.country_name,
    fra.year,
    fra.forest_area_sqkm,
    lda.total_area_sq_mi,
    lda.total_area_sq_mi * 2.59 AS total_area_sqkm,
    reg.region,
    reg.income_group,
    (fra.forest_area_sqkm / (lda.total_area_sq_mi * 2.59)) * 100 AS forest_percent
FROM 
    forest_area AS fra
JOIN 
    land_area AS lda ON fra.country_code = lda.country_code 
                     AND fra.year = lda.year
JOIN 
    regions reg ON reg.country_code = lda.country_code
GROUP BY 
    fra.country_code,
    fra.country_name,
    fra.year,
    reg.country_name, 
    reg.year, 
    reg.income_group, 
    reg.region, 
    lda.total_area_sq_mi, 
    fra.forest_area_sqkm;

-- GLOBAL SITUATION

-- What was the total forest area (in sq km) of the world in 1990?
SELECT SUM(forest_area_sqkm) AS total_area_of_forest
FROM forestation
WHERE YEAR = 1990
AND country_name = 'World';

-- What was the total forest area (in sq km) of the world in 2016?
SELECT SUM(forest_area_sqkm) AS total_area_of_forest
FROM forestation
WHERE YEAR = 2016
AND country_name = 'World';

-- What was the change (in sq km) in the forest area of the world from 1990 to 2016?
SELECT ( 
    (SELECT SUM(forest_area_sqkm) AS total_forest_area 
     FROM forestation 
     WHERE YEAR = 1990 
     AND country_name = 'World') -
    (SELECT SUM(forest_area_sqkm) AS total_forest_area 
     FROM forestation 
     WHERE YEAR = 2016 
     AND country_name = 'World')
) AS Change_in_Forest_Area
FROM forestation
LIMIT 1;

-- What was the percent change in forest area of the world between 1990 and 2016?
SELECT (
    (( 
        (SELECT SUM(forest_area_sqkm) AS total_forest_area 
         FROM forestation 
         WHERE YEAR = 1990 
         AND country_name = 'World') -
        (SELECT SUM(forest_area_sqkm) AS total_forest_area 
         FROM forestation 
         WHERE YEAR = 2016 
         AND country_name = 'World')
    ) / (
        (SELECT SUM(forest_area_sqkm) AS total_forest_area 
         FROM forestation 
         WHERE YEAR = 1990 
         AND country_name = 'World')
    )) * 100
) AS Percent_Change_in_Forest_Area
FROM forestation 
LIMIT 1;

-- If you compare the amount of forest area lost between 1990 and 2016, to which country's total area in 2016 is it closest to?
SELECT country_name, 
    SUM(total_area_sq_mi * 2.59) AS total_area_of_land
FROM forestation 
WHERE YEAR = 2016 
AND total_area_sq_mi * 2.59 <= 1324449
GROUP BY country_name
ORDER BY total_area_of_land DESC
LIMIT 1;

-- REGIONAL OUTLOOK

-- What was the percent forest of the entire world in 2016? 
SELECT country_name, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area2016
FROM forestation 
WHERE YEAR = 2016 
AND country_name = 'World' 
GROUP BY country_name;

-- Which region had the HIGHEST percent forest in 2016, and which had the LOWEST, to 2 decimal places?
(SELECT region, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area2016
 FROM forestation 
 WHERE YEAR = 2016
 GROUP BY region 
 ORDER BY Percent_Forest_Area2016 DESC
 LIMIT 1)
UNION ALL
(SELECT region, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area2016
 FROM forestation 
 WHERE YEAR = 2016
 GROUP BY region
 ORDER BY Percent_Forest_Area2016 ASC
 LIMIT 1);

-- What was the percent forest of the entire world in 1990? 
SELECT country_name, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area1990
FROM forestation 
WHERE YEAR = 1990 
AND country_name = 'World' 
GROUP BY country_name;

-- Which region had the HIGHEST percent forest in 1990, and which had the LOWEST, to 2 decimal places?
(SELECT region, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area1990
 FROM forestation 
 WHERE YEAR = 1990
 GROUP BY region 
 ORDER BY Percent_Forest_Area1990 DESC
 LIMIT 1)
UNION ALL
(SELECT region, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area1990
 FROM forestation 
 WHERE YEAR = 1990
 GROUP BY region
 ORDER BY Percent_Forest_Area1990 ASC
 LIMIT 1);

-- Based on the table you created, which regions of the world DECREASED in forest area from 1990 to 2016?
WITH T1 AS
(SELECT region, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area1990
 FROM forestation 
 WHERE YEAR = 1990 
 GROUP BY region 
 ORDER BY Percent_Forest_Area1990 DESC), 
T2 AS
(SELECT region, 
    ROUND(((SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100)::Numeric, 2) AS Percent_Forest_Area2016
 FROM forestation 
 WHERE YEAR = 2016 
 GROUP BY region 
 ORDER BY Percent_Forest_Area2016 DESC)
SELECT fra.region, 
    fra.Percent_Forest_Area1990, 
    tra.Percent_Forest_Area2016 
FROM T1 AS fra
JOIN T2 AS tra ON fra.region = tra.region
WHERE fra.Percent_Forest_Area1990 > tra.Percent_Forest_Area2016
GROUP BY fra.region, 
    fra.Percent_Forest_Area1990, 
    tra.Percent_Forest_Area2016 
LIMIT 2;

-- COUNTRY-LEVEL DETAIL

-- Which 5 countries saw the largest amount decrease in forest area from 1990 to 2016? What was the difference in forest area for each?
WITH T1 AS
(SELECT country_name, 
    ROUND(SUM(forest_area_sqkm)) AS Forest_Area1990
 FROM forestation 
 WHERE YEAR = 1990 
 GROUP BY country_name, forest_area_sqkm 
 ORDER BY Forest_Area1990 DESC), 
T2 AS
(SELECT country_name, 
    ROUND(SUM(forest_area_sqkm)) AS Forest_Area2016
 FROM forestation 
 WHERE YEAR = 2016 
 GROUP BY country_name, forest_area_sqkm
 ORDER BY Forest_Area2016 DESC)
SELECT fra.country_name, 
    fra.Forest_Area1990, 
    tra.Forest_Area2016, 
    (tra.Forest_Area2016 - fra.Forest_Area1990) AS Difference_Land_Area 
FROM T1 AS fra 
JOIN T2 AS tra ON fra.country_name = tra.country_name
WHERE fra.country_name != 'World'
GROUP BY fra.country_name, 
    tra.Forest_Area2016, 
    fra.Forest_Area1990
ORDER BY Difference_Land_Area 
LIMIT 5;

-- Which 5 countries saw the largest percent decrease in forest area from 1990 to 2016? What was the percent change to 2 decimal places for each?
WITH T1 AS
(SELECT country_name, 
    region, 
    ROUND((forest_area_sqkm)::Numeric, 2) AS Percent_Forest_Area1990
 FROM forestation 
 WHERE YEAR = 1990 
 GROUP BY country_name, region, forest_area_sqkm),
T2 AS
(SELECT country_name, 
    region, 
    ROUND((forest_area_sqkm)::Numeric, 2) AS Percent_Forest_Area2016
 FROM forestation 
 WHERE YEAR = 2016 
 GROUP BY country_name, region, forest_area_sqkm)
SELECT fra.country_name, 
    fra.region, 
    fra.Percent_Forest_Area1990, 
    tra.Percent_Forest_Area2016, 
    (fra.Percent_Forest_Area1990 - tra.Percent_Forest_Area2016) AS Difference_Land_Area,
LIMIT 5;

-- If countries were grouped by percent forestation in quartiles, which group had the most countries in it in 2016?

WITH T1 AS (
    SELECT 
        country_name, 
        YEAR, 
        (SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100 AS Percent_Forest_in_Quartiles
    FROM 
        forestation 
    WHERE 
        YEAR = 2016 
    GROUP BY 
        country_name, YEAR
) 
SELECT 
    DISTINCT(quartiles), 
    COUNT(country_name) OVER(PARTITION BY quartiles) 
FROM (
    SELECT 
        country_name, 
        CASE
            WHEN Percent_Forest_in_Quartiles < 25 THEN '0-25%'
            WHEN Percent_Forest_in_Quartiles >= 25 AND Percent_Forest_in_Quartiles < 50 THEN '25-50%'
            WHEN Percent_Forest_in_Quartiles >= 50 AND Percent_Forest_in_Quartiles < 75 THEN '50-75%'
            ELSE '75-100%'
        END AS quartiles
    FROM 
        T1
    WHERE 
        Percent_Forest_in_Quartiles IS NOT NULL
        AND YEAR = 2016
) sub;

-- List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.

WITH T1 AS (
    SELECT 
        country_name, 
        YEAR, 
        (SUM(forest_area_sqkm) / SUM(total_area_sq_mi * 2.59)) * 100 AS Percent_Forest_in_Quartiles
    FROM 
        forestation 
    WHERE 
        YEAR = 2016 
    GROUP BY 
        country_name, YEAR
) 
SELECT 
    DISTINCT(quartiles), 
    COUNT(country_name) OVER(PARTITION BY quartiles) 
FROM (
    SELECT 
        country_name, 
        CASE
            WHEN Percent_Forest_in_Quartiles < 25 THEN '0-25%'
            WHEN Percent_Forest_in_Quartiles >= 25 AND Percent_Forest_in_Quartiles < 50 THEN '25-50%'
            WHEN Percent_Forest_in_Quartiles >= 50 AND Percent_Forest_in_Quartiles < 75 THEN '50-75%'
            ELSE '75-100%'
        END AS quartiles
    FROM 
        T1
    WHERE 
        Percent_Forest_in_Quartiles IS NOT NULL
        AND YEAR = 2016
) sub;

-- List all of the countries that were in the 4th quartile (percent forest > 75%) in 2016.

SELECT 
    country_name, 
    region, 
    forest_percent AS Percent_Forest_in_Quartiles 
FROM 
    forestation
WHERE 
    forest_percent > 75 
    AND year = 2016
GROUP BY 
    country_name, region, forest_percent
ORDER BY 
    Percent_Forest_in_Quartiles DESC;