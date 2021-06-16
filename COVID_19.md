# COVID-19 Dataset Analysis

Analysis on publicly available COVID-19 data through May 12, 2021.

<a href="https://github.com/ajmercado1229/COVID19_Project.git">View original files in Github</a>

<a href="https://public.tableau.com/views/COVIDVisualization_16226608667290/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link">View the Dashboard on Tableau Public</a>

## Querying & Visualization
Microsoft SQL Server & Tableau Public

---

### 9) Global Numbers Total

Shows the total number of cases & deaths globally.

```
SELECT SUM(new_cases) AS TotalCases,
	   SUM(CAST(new_deaths AS INT)) AS TotalDeaths,
	   (SUM(CAST(new_deaths AS INT))/SUM(new_cases))*100 AS DeathPercentage
FROM PortfolioProject.dbo.CovidDeaths
WHERE continent IS NOT NULL
ORDER BY 1,2
```

This query will return results showing the Total Cases globally as well as the Total Deaths. From there you can derive the Total Death Percentage. The results can be uploaded via Excel into Tableau Public for visualization.


<img src="images/covid01.PNG?raw=true"/>

---

### 15) Total Deaths per Continent

```
SELECT location, 
       SUM(cast(new_deaths as int)) as TotalDeathCount
FROM PortfolioProject.dbo.CovidDeaths
WHERE continent IS NULL 
AND location NOT IN ('World', 'European Union', 'International')
GROUP BY location
ORDER BY TotalDeathCount DESC
```

This query returns the Total Death numbers per continent. Some locations must be excluded because they are either subtotals or totals and would duplicate some of the data.


<img src="images/covid06.PNG?raw=true"/>

---

### 5) Countries with the Highest Infection Rate compared to the Population.

```
SELECT location,
	   population,
	   MAX(total_cases) AS HighestInfectionCount,
	   MAX((total_cases/population))*100 AS PopulationPercentageInfected
FROM PortfolioProject.dbo.CovidDeaths
GROUP BY location,
	 population
ORDER BY PopulationPercentageInfected DESC
```

From the dataset we can inquire about the populations and highest total cases per country and be able to create a new field called PopulationPercentageInfected. We can import the results into Tableau and create a map showing highest infection rates across the world.


<img src="images/covid03.PNG?raw=true"/>

---

### 14) Infection Rates Over Time by Country

```
SELECT Location, 
       Population,
       date, 
       MAX(total_cases) as HighestInfectionCount,  
       MAX((total_cases/population))*100 as PercentPopulationInfected
FROM PortfolioProject.dbo.CovidDeaths
GROUP BY Location, Population, date
ORDER BY PercentPopulationInfected DESC
```
---


### 3) Total Cases vs Total Deaths

Shows the likelihood of dying if you contract COVID-19 in your country.
In this case, it is set to specify the United States.

```
SELECT location,
	   date,
	   total_cases,
	   total_deaths,
	   (total_deaths/total_cases)*100 AS DeathPercentage
FROM PortfolioProject.dbo.CovidDeaths
WHERE location like '%states%'
ORDER BY 1,2
```
---

### 4) Total Cases vs Population

Shows what percentage of the population has contracted COVID-19
In this case, it is set to specify the United States.

```
SELECT location,
	   date,
	   population,
	   total_cases,
	   (total_cases/population)*100 AS PopulationPercentage
FROM PortfolioProject.dbo.CovidDeaths
WHERE location like '%states%'
ORDER BY 1,2
```
---



### 6) Countries with Highest Death Count per Population

```
SELECT location,
	   MAX(CAST(total_deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject.dbo.CovidDeaths
WHERE continent IS NOT NULL
GROUP BY location,
	 population
ORDER BY TotalDeathCount DESC
```

* total_deaths is NVARCHAR so it must be cast as an integer to aggregate it.
* The location field includes both countries and continents so the where continent is not null line will ensure that only countries are shown and not continent summaries or duplicate data.


---

### 7) Continents with Highest Death Count per Population

```
SELECT location,
	   MAX(CAST(total_deaths AS INT)) AS TotalDeathCount
FROM PortfolioProject.dbo.CovidDeaths
WHERE continent IS NULL
GROUP BY location
ORDER BY TotalDeathCount DESC
```
---

### 8) Global Numbers by Date 

```
SELECT date,
	   SUM(new_cases) AS TotalCases,
	   SUM(CAST(new_deaths AS INT)) AS TotalDeaths,
	   (SUM(CAST(new_deaths AS INT))/SUM(new_cases))*100 AS DeathPercentage
FROM PortfolioProject.dbo.CovidDeaths
WHERE continent IS NOT NULL
GROUP BY date
ORDER BY 1,2
```
---



### 10) Total Population vs Vaccination

```
SELECT DEA.continent,
	   DEA.location,
	   DEA.date,
	   DEA.population,
	   VAC.new_vaccinations,
	   SUM(CAST(VAC.new_vaccinations AS INT)) 
			OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS RollingPeopleVaccinated
FROM PortfolioProject.dbo.CovidDeaths AS DEA
JOIN PortfolioProject.dbo.CovidVaccinations AS VAC
	ON DEA.location = VAC.location
	AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
ORDER BY 2,3
```
---

### 11) Total Population vs Vaccination CTE (common table expression): allows it to be referenced later.


```
WITH PopVsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated)
AS
(
SELECT DEA.continent,
	   DEA.location,
	   DEA.date,
	   DEA.population,
	   VAC.new_vaccinations,
	   SUM(CAST(VAC.new_vaccinations AS INT)) 
			OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS RollingPeopleVaccinated
FROM PortfolioProject.dbo.CovidDeaths AS DEA
JOIN PortfolioProject.dbo.CovidVaccinations AS VAC
	ON DEA.location = VAC.location
	AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL
)

SELECT *,
	   (RollingPeopleVaccinated/Population)*100 AS PercentOfPopulationVaccinated
FROM PopVsVac
WHERE Location = 'United States'
```
---

### 12) Total Population vs Vaccination Temp Table

```
DROP TABLE IF EXISTS #PercentPopulationVaccinated

CREATE TABLE #PercentPopulationVaccinated
(
Continent NVARCHAR(255),
Location NVARCHAR(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

INSERT INTO #PercentPopulationVaccinated
SELECT DEA.continent,
	   DEA.location,
	   DEA.date,
	   DEA.population,
	   VAC.new_vaccinations,
	   SUM(CAST(VAC.new_vaccinations AS INT)) 
			OVER (PARTITION BY DEA.location ORDER BY DEA.location, DEA.date) AS RollingPeopleVaccinated
FROM PortfolioProject.dbo.CovidDeaths AS DEA
JOIN PortfolioProject.dbo.CovidVaccinations AS VAC
	ON DEA.location = VAC.location
	AND DEA.date = VAC.date
WHERE DEA.continent IS NOT NULL


SELECT *,
	   (RollingPeopleVaccinated/Population)*100
FROM #PercentPopulationVaccinated
```
---

### 13) Current Death Percentage by Country

```
SELECT B.location,
	   B.date,
	   B.total_cases,
	   B.total_deaths,
	   (B.total_deaths/B.total_cases)*100 AS DeathPercentage
FROM PortfolioProject.dbo.CovidDeaths AS B
INNER JOIN(
	SELECT location,
			MAX(date) AS MaxDate
	FROM PortfolioProject.dbo.CovidDeaths
	GROUP BY location) AS A
ON A.location = B.location 
AND B.date = A.MaxDate
WHERE continent IS NOT NULL
ORDER BY location
```
---





## Visualizing COVID-19 Data in Tableau Public

Some of the queries performed above will be used to create visualizations in 
<a href="https://public.tableau.com/views/COVIDVisualization_16226608667290/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link">Tableau Public</a>
---

### Countries with the Highest Infection Rate compared to the Population (#5)



---

### Global Numbers by Date (#8)



---

### Infection Rates Over Time by Country (#14)



---

### Total Deaths per Continent (#15)

<img src="images/covid04.PNG?raw=true"/>

---

## COVID-19 Dashboard

The visualizations can be conveniently combined to form a single interactive dashboard. From here you are able to toggle different filters to view various stats. 

<img src="images/covid05.PNG?raw=true"/>

As demonstrated, a large and complex dataset can be transformed with SQL and Tableau into an easily understandable and interactive dashboard.
