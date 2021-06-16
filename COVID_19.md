# COVID-19 Dataset Analysis

Analysis on publicly available COVID-19 data through May 12, 2021.

<a href="https://github.com/ajmercado1229/COVID19_Project.git">View original files in Github</a>

<a href="https://public.tableau.com/views/COVIDVisualization_16226608667290/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link">View the Dashboard on Tableau Public</a>

## Querying & Visualization
Microsoft SQL Server & Tableau Public

---

### Global Numbers Total

```sql
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

### Total Deaths per Continent

```sql
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

### Countries with the Highest Infection Rate compared to the Population.

```sql
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

### Infection Rates Over Time by Country

```sql
SELECT Location, 
       Population,
       date, 
       MAX(total_cases) as HighestInfectionCount,  
       MAX((total_cases/population))*100 as PercentPopulationInfected
FROM PortfolioProject.dbo.CovidDeaths
GROUP BY Location, Population, date
ORDER BY PercentPopulationInfected DESC
```

We can use this query to view the changing infection rates over time by country to evaluate which countries are becoming more highly infected and which countries are doing a better job at containing the virus. This visualization has been filtered to only show certain countries, but it can be changed to include any country within the dataset. Tableau also has a great forecasting tool to predict future results based on past patterns and data. This dataset includes dates up until May 12th, 2021 and Tableau can show estimates through the end of 2021.


<img src="images/covid04.PNG?raw=true"/>

---

### COVID-19 Dashboard

The visualizations can be conveniently combined to form a single interactive dashboard. From here you are able to toggle different filters to view various stats. 


<img src="images/covid05.PNG?raw=true"/>

---

## Bonus Queries for Analysis

---

### Total Cases vs Total Deaths

Shows the likelihood of dying if you contract COVID-19 in your country.
In this case, it is set to specify the United States.

```sql
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

### Total Cases vs Population

Shows what percentage of the population has contracted COVID-19
In this case, it is set to specify the United States.

```sql
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

### Countries with Highest Death Count per Population

```sql
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

### Global Numbers by Date 

Shows the global number of cases and deaths over time.

```sql
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

### Total Population vs Vaccination

Shows a rolling number of people vaccinated per country.

```sql
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

### Current Death Percentage by Country

Shows the current death percentage by country. In this case the dataset extends until May 12th, 2021 but going forward data could be updated and the query would remain functional.
```sql
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
