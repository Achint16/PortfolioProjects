# PortfolioProjects

## Covid 19 Data Exploration 

##### Information On datasourse:
###### 1. From: https://ourworldindata.org/covid-deaths
###### 2. The data on the coronavirus pandemic.
##### Skills used: Joins, CTE's, Temp Tables, Windows Functions, Aggregate Functions, Creating Views, Converting Data Types



```
SELECT *  
FROM `autonomous-mote-339511.Covid_analysis.CovidDeaths` 
order by 3,4
LIMIT 1000
```
---------------------------------------------------------------------------------------------------------------------------------------------------
### Select Data that we are going to be starting with
```
SELECT 
    location,
    date,
    total_cases,
    new_cases,
    total_deaths,
    population
from `autonomous-mote-339511.Covid_analysis.CovidDeaths`
order by 1,2
```
-----------------------------------------------------------------------------------------------------------------------------------------------------

### looking at total cases vs total deaths and death percentage for India 

```
SELECT 
    location,
    date,
    total_cases,
    total_deaths,
    population,
    (total_deaths / total_cases) * 100 as DeathPercentage
from `autonomous-mote-339511.Covid_analysis.CovidDeaths`
WHERE 
    location = 'India'
order by 1,2
```
-------------------------------------------------------------------------------------------------------------------------------------------------------------
### total cases vs Population in india , cases % 
```
SELECT 
    location,
    date,
    total_cases,
    population,
    (total_cases/population)*100 as case_percentage
from `autonomous-mote-339511.Covid_analysis.CovidDeaths`
where location = 'India'
order by 1,2
```
-----------------------------------------------------------------------------------------------------------------------------------------
### showing the countries with highest death count 
```
select 
    max(cast(total_deaths as int)) as deathCount,
    continent
from `autonomous-mote-339511.Covid_analysis.CovidDeaths`
where 
    continent is not NULL    #we had to use where clause because it started grouping continents in location column where continent is null  
GROUP BY 
    continent
ORDER BY 
    deathCount desc
```
----------------------------------------------------------------------------------------------------------- 
### Global Numbers
```
select 
    date, 
    SUM(new_cases) as NewCases,
    sum(new_deaths) as NewDeaths,
    #total_cases,
    #total_deaths,
    (sum(new_deaths)/sum(new_cases)) * 100 as deathPercentage
from `autonomous-mote-339511.Covid_analysis.CovidDeaths`
where 
    continent is not NULL    #we had to use where clause because it started grouping continents in location column where continent is null  
GROUP BY 
    date 
ORDER BY 
    1,2 

```
-----------------------------------------------------------------------------------------------------------------
### Using another data set and functions like inner join
### looking at total vacination and total population 
```
select 
    dea.date,
    dea.location,
    dea.continent,
    dea.population,
    vac.new_vaccinations,
    sum(vac.new_vaccinations) over (partition by dea.location order by dea.location,dea.date desc ) as rolling_people_vacinated, #what this function does is it aggregates every time a new vacination is added to list,
 #   (rolling_people_vacinated/ dea.population)* 100 

from `autonomous-mote-339511.Covid_analysis.CovidDeaths` dea
join `autonomous-mote-339511.Covid_analysis.CovidVacination` vac
    on  dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
order by 2,3
```
--------------------------------------------------------------------------------------------------------------------------------------------------------
### Using CTE
```
with Popsvsvacs
as
(
select 
    dea.date,
    dea.location,
    dea.continent,
    dea.population,
    vac.new_vaccinations,
    sum(vac.new_vaccinations) over (partition by dea.location order by dea.location,dea.date desc ) as rolling_people_vacinated, #what this function does is it aggregates every time a new vacination is added to list,
 #   (rolling_people_vacinated/ dea.population)* 100 

from `autonomous-mote-339511.Covid_analysis.CovidDeaths` dea
join `autonomous-mote-339511.Covid_analysis.CovidVacination` vac
    on  dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
order by 2,3
)
select *,
    (rolling_people_vacinated/ population)* 100
from Popsvsvacs 
where location = 'India'
```
-----------------------------------------------------------------------------------------------------------------------------------------------------------
### Using Temp Table to perform Calculation on Partition By in previous query

```
DROP Table if exists #PercentPopulationVaccinated
Create Table #PercentPopulationVaccinated
(
Continent nvarchar(255),
Location nvarchar(255),
Date datetime,
Population numeric,
New_vaccinations numeric,
RollingPeopleVaccinated numeric
)

Insert into #PercentPopulationVaccinated
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated

```
-----------------------------------------------------------------------------------------------------------------------------------------------------------
### Creating a view
```
select 
    dea.date,
    dea.location,
    dea.continent,
    dea.population,
    vac.new_vaccinations,
    sum(vac.new_vaccinations) over (partition by dea.location order by dea.location,dea.date desc ) as rolling_people_vacinated, #what this function does is it aggregates every time a new vacination is added to list,
 #   (rolling_people_vacinated/ dea.population)* 100 

from `autonomous-mote-339511.Covid_analysis.CovidDeaths` dea
join `autonomous-mote-339511.Covid_analysis.CovidVacination` vac
    on  dea.location = vac.location
    and dea.date = vac.date
where dea.continent is not null 
order by 2,3
```


