# SQL Data Exploration Project: COVID-19 Analysis
## Project Overview
This project involves exploratory data analysis of COVID-19 data using SQL. The analysis covers various aspects of the pandemic, including case counts, death rates, infection rates, and vaccination progress across different countries and continents.

## Dataset Description
The project utilizes two datasets:

**CovidDeaths**: Contains information about COVID-19 cases, deaths, and population by location and date.
**CovidVaccinations**: Contains information about COVID-19 vaccinations by location and date.
## SQL Queries and Analysis:
### Using CTE to Perform Calculations
### This query uses a Common Table Expression (CTE) to calculate the rolling number of people vaccinated.
```sql
WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated) AS
(
  SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
  SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
  FROM [portffolio project]..CovidDeaths dea
  JOIN [portffolio project]..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
  WHERE dea.continent IS NOT NULL 
)
SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentPopulationVaccinated
FROM PopvsVac;
```
### Using Temp Table to Perform Calculations
### This query uses a temporary table to store and calculate the percentage of the population vaccinated.
```sql
DROP TABLE IF EXISTS #PercentPopulationVaccinated;
CREATE TABLE #PercentPopulationVaccinated
(
  Continent NVARCHAR(255),
  Location NVARCHAR(255),
  Date DATETIME,
  Population NUMERIC,
  New_vaccinations NUMERIC,
  RollingPeopleVaccinated NUMERIC
);
INSERT INTO #PercentPopulationVaccinated
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM [portffolio project]..CovidDeaths dea
JOIN [portffolio project]..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date;
SELECT *, (RollingPeopleVaccinated/Population)*100 AS PercentPopulationVaccinated
FROM #PercentPopulationVaccinated;
```
### Creating View to Store Data for Later Visualizations
### This query creates a view to store the percentage of the population vaccinated for later use in visualizations.
```sql
CREATE VIEW PercenttPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM [portffolio project]..CovidDeaths dea
JOIN [portffolio project]..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
SELECT * FROM PercenttPopulationVaccinated;
```

## Illustrations


**Illustration 1:** Total Cases vs Total Deaths
![total death vstotal cases](https://github.com/user-attachments/assets/fabcbb97-46ed-4f78-ba57-7e9f1edc1277)

**Illustration 2:** Total Cases vs Population
![total cases vs population](https://github.com/user-attachments/assets/3f670b7a-a3a9-4a09-a642-0b5f99e4ab3c)

**Illustration 3:** Countries with Highest Infection Rate Compared to Population
![countries with hughest infection rate compared to population](https://github.com/user-attachments/assets/8b6f0741-4790-49ab-88fe-222b49061bce)
## Conclusion
This project demonstrates how to use SQL for exploratory data analysis on COVID-19 data. By querying and analyzing the data, we can gain insights into infection rates, death rates, and vaccination progress across different regions. The use of views, CTEs, and temporary tables helps organize and calculate data for more complex analyses and visualizations.

## Future Work
**Further Analysis:** Expand the analysis to include more metrics and deeper insights.

**Data Visualization:** Use tools like Tableau or Power BI to create interactive dashboards based on the results of these queries.

**Automation:** Automate the data extraction and analysis process for real-time insights.
