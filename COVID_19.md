# COVID-19 Dataset Analysis

Analysis on publicly available COVID-19 data through May 12, 2021.

<a href="https://github.com/ajmercado1229/COVID19_Project.git">View original files in Github</a>

<a href="https://public.tableau.com/views/COVIDVisualization_16226608667290/Dashboard1?:language=en-US&:display_count=n&:origin=viz_share_link">View the Dashboard on Tableau Public</a>

<div class='tableauPlaceholder' id='viz1623875471191' style='position: relative'><noscript><a href='#'><img alt='Dashboard 1 ' src='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;CO&#47;COVIDVisualization_16226608667290&#47;Dashboard1&#47;1_rss.png' style='border: none' /></a></noscript><object class='tableauViz'  style='display:none;'><param name='host_url' value='https%3A%2F%2Fpublic.tableau.com%2F' /> <param name='embed_code_version' value='3' /> <param name='site_root' value='' /><param name='name' value='COVIDVisualization_16226608667290&#47;Dashboard1' /><param name='tabs' value='no' /><param name='toolbar' value='yes' /><param name='static_image' value='https:&#47;&#47;public.tableau.com&#47;static&#47;images&#47;CO&#47;COVIDVisualization_16226608667290&#47;Dashboard1&#47;1.png' /> <param name='animate_transition' value='yes' /><param name='display_static_image' value='yes' /><param name='display_spinner' value='yes' /><param name='display_overlay' value='yes' /><param name='display_count' value='yes' /><param name='language' value='en-US' /></object></div>                <script type='text/javascript'>                    var divElement = document.getElementById('viz1623875471191');                    var vizElement = divElement.getElementsByTagName('object')[0];                    if ( divElement.offsetWidth > 800 ) { vizElement.style.width='100%';vizElement.style.height=(divElement.offsetWidth*0.75)+'px';} else if ( divElement.offsetWidth > 500 ) { vizElement.style.width='100%';vizElement.style.height=(divElement.offsetWidth*0.75)+'px';} else { vizElement.style.width='100%';vizElement.style.height='1227px';}                     var scriptElement = document.createElement('script');                    scriptElement.src = 'https://public.tableau.com/javascripts/api/viz_v1.js';                    vizElement.parentNode.insertBefore(scriptElement, vizElement);                </script>

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
