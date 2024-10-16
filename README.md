
# ForestQuery Deforestation Analysis

## Project Overview
This project was developed as part of a course on Udacity. The goal is to analyze global deforestation trends from 1990 to 2016 for ForestQuery, a non-profit organization focused on reducing deforestation worldwide. The project aims to provide insights into forest size changes globally, regionally, and at the country level, with the results intended to guide the organization's strategy and allocation of resources.

## Project Introduction
As a data analyst for ForestQuery, your task is to help the executive leadership team understand global deforestation trends. The leadership team is particularly interested in identifying which countries and regions have experienced the most significant reduction in forest area, as well as which areas have the largest remaining forest area in terms of absolute size and as a percentage of total land area. This information will support the organization’s initiatives to make the most of its limited resources in combating deforestation.

You’ve collected data from various sources, including tables of forest area, land area, and regional groupings. You have brought these together into a SQL database, which you query to extract key insights and prepare a comprehensive report.

## Features
- **Global Analysis:** Calculation of total global forest area in 1990 and 2016, along with the percent change and total decrease in forest area.
- **Regional Analysis:** Detailed comparison of forest area percentages for different regions, highlighting the highest and lowest forested regions.
- **Country-Level Insights:** Identification of the countries that experienced the largest decreases in forest area, both in absolute terms and percentage change.
- **Quartile-Based Grouping:** Grouping countries by forestation quartiles and identifying those with high forestation percentages.
- **Actionable Recommendations:** Providing data-driven recommendations for resource allocation to maximize ForestQuery’s impact.

## Technologies Used
- **SQL:** Core technology used to query and analyze the dataset.
- **PostgreSQL:** Database management system used for running queries and managing the tables.

## Datasets:
- **Forest Area Data:** Contains information on forest area per country from 1990 to 2016.
- **Land Area Data:** Provides the total land area of each country.
- **Regions Data:** Groups countries into regions for regional analysis.

## Installation and Setup
To set up this project locally:

1. Install PostgreSQL or any compatible SQL database system.
2. Clone this repository to your local machine.
3. Create a database and import the provided dataset files (forest_area.csv, land_area.csv, regions.csv).
4. Run the SQL queries provided in the Appendix section to create the necessary views and generate the report.

### Sample SQL Setup Commands:
```sql
-- Example command to create the forestation view
CREATE VIEW forestation AS
SELECT fa.country_code, fa.year, fa.forest_area_sqkm, la.land_area_sqmi, r.region,
       (fa.forest_area_sqkm / (la.land_area_sqmi * 2.59)) * 100 AS percent_forest_area
FROM forest_area fa
JOIN land_area la ON fa.country_code = la.country_code AND fa.year = la.year
JOIN regions r ON fa.country_code = r.country_code;
```

## Usage
After setting up the database and running the SQL scripts, you can execute the queries found in the Appendix section to generate insights and produce the final report. Each query is structured to answer specific questions about global, regional, and country-level deforestation trends.

For example, to calculate the total forest area in 1990, you can use a query like this:
```sql
-- Total forest area in 1990
SELECT SUM(forest_area_sqkm) AS total_forest_area_1990
FROM forest_area
WHERE country_code = 'World' AND year = 1990;
```

## Project Structure
- **Global Situation:** Analysis of global forest area trends between 1990 and 2016.
- **Regional Outlook:** Comparison of forest area percentages across different regions.
- **Country-Level Detail:** Country-specific insights on deforestation.
- **Recommendations:** Data-driven suggestions for ForestQuery’s leadership team.
- **Appendix:** SQL queries used in the analysis.

## Challenges and Considerations
One of the main challenges in this project was ensuring consistency in measurement units. The forest area is measured in square kilometers, while land area is measured in square miles. This required converting land area into square kilometers (1 square mile = 2.59 square kilometers) before calculating forest area percentages.

Another challenge was ensuring that the analysis accounted for both absolute changes in forest area and relative percentage changes, which provided a more comprehensive understanding of the impact of deforestation.

## Results
### Global Situation:
- **Total Forest Area in 1990:** [Result from SQL query]
- **Total Forest Area in 2016:** [Result from SQL query]
- **Percent Change (1990-2016):** [Result from SQL query]
- **Comparison to Country's Total Area:** [Result from SQL query]

### Regional Outlook:
- **Highest Percent Forest in 2016:** [Region and percent]
- **Lowest Percent Forest in 2016:** [Region and percent]
- **Regions with Decreasing Forest Area (1990-2016):** [List of regions]

### Country-Level Detail:
- **Top 5 Countries with Largest Forest Area Decrease (1990-2016):** [List of countries and amount]
- **Top 5 Countries with Largest Percent Decrease in Forest Area (1990-2016):** [List of countries and percentages]
- **Countries in 4th Quartile (Forest Area > 75% in 2016):** [List of countries]

## Appendix: SQL Queries
A full list of SQL queries used to generate the above results can be found in the SQL_Queries.sql file.

## Recommendations
Based on the analysis, the following recommendations are suggested for ForestQuery:

- Focus resources on regions and countries that have seen the largest decreases in forest area.
- Prioritize communication efforts in regions with the lowest forestation percentages to raise awareness.
- Allocate personnel to countries that still maintain a high percentage of forest area for conservation efforts.
