# SQL Data Exploration Project: COVID-19 Analysis
## Project Overview
This project involves exploratory data analysis of COVID-19 data using SQL. The analysis covers various aspects of the pandemic, including case counts, death rates, infection rates, and vaccination progress across different countries and continents.

## Dataset Description
The project utilizes two datasets:

**CovidDeaths**: Contains information about COVID-19 cases, deaths, and population by location and date.
**CovidVaccinations**: Contains information about COVID-19 vaccinations by location and date.
## SQL Queries and Analysis:
### Selecting All Data from Tables
SELECT *
FROM [portffolio project].[dbo].[CovidDeaths];

SELECT* 
FROM [portffolio project].[dbo].[CovidVaccinations];

### Selecting Specific Data from the Table
### This query selects location, date, total cases, new cases, total deaths, and population from the CovidDeaths table where the continent is not null, ordered by location and date.
SELECT Location, date, total_cases, new_cases, total_deaths, population
FROM [portffolio project]..CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 1,2;

### Total Cases vs Total Deaths
### This query shows the likelihood of dying if you contract COVID-19 in Algeria.
SELECT Location, date, total_cases, total_deaths, (total_deaths/total_cases)*100 AS DeathPercentage
FROM [portffolio project]..CovidDeaths
WHERE location LIKE '%alger%'
AND continent IS NOT NULL 
ORDER BY 1,2;

### Total Cases vs Population
### This query shows the percentage of the population infected with COVID-19.
SELECT Location, date, Population, total_cases, (total_cases/population)*100 AS PercentPopulationInfected
FROM [portffolio project]..CovidDeaths
ORDER BY 1,2;

### Countries with Highest Infection Rate Compared to Population
### This query finds the countries with the highest infection rate as a percentage of their population.
SELECT Location, Population, MAX(total_cases) AS HighestInfectionCount, MAX((total_cases/Population)*100) AS PercentPopulationInfected
FROM [portffolio project]..CovidDeaths
GROUP BY Location, Population
ORDER BY PercentPopulationInfected DESC;

### Countries with Highest Death Count per Population
### This query finds the countries with the highest death count per population.
SELECT Location, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM [portffolio project]..CovidDeaths
WHERE continent IS NOT NULL 
GROUP BY Location
ORDER BY TotalDeathCount DESC;

### Continent-wise Analysis
### Continents with the Highest Death Count per Population.
SELECT continent, MAX(CAST(Total_deaths AS int)) AS TotalDeathCount
FROM [portffolio project]..CovidDeaths
WHERE continent IS NOT NULL 
GROUP BY continent
ORDER BY TotalDeathCount DESC;

### Global Numbers
### Total Cases and Deaths.
SELECT SUM(new_cases) AS total_cases, SUM(CAST(new_deaths AS int)) AS total_deaths, SUM(CAST(new_deaths AS int))/SUM(New_Cases)*100 AS DeathPercentage
FROM [portffolio project]..CovidDeaths
WHERE continent IS NOT NULL 
ORDER BY 1,2;

### Total Population vs Vaccinations
### This query shows the percentage of the population that has received at least one COVID vaccine.
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM [portffolio project]..CovidDeaths dea
JOIN [portffolio project]..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL 
ORDER BY 2,3;

### Using CTE to Perform Calculations
### This query uses a Common Table Expression (CTE) to calculate the rolling number of people vaccinated.
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

### Using Temp Table to Perform Calculations
### This query uses a temporary table to store and calculate the percentage of the population vaccinated.
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

### Creating View to Store Data for Later Visualizations
### This query creates a view to store the percentage of the population vaccinated for later use in visualizations.
CREATE VIEW PercenttPopulationVaccinated AS
SELECT dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, 
SUM(CONVERT(int,vac.new_vaccinations)) OVER (PARTITION BY dea.Location ORDER BY dea.location, dea.Date) AS RollingPeopleVaccinated
FROM [portffolio project]..CovidDeaths dea
JOIN [portffolio project]..CovidVaccinations vac ON dea.location = vac.location AND dea.date = vac.date
WHERE dea.continent IS NOT NULL;
SELECT * FROM PercenttPopulationVaccinated;

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
