# Covid-Project

--Simple Data exploring Death Data (Reference)
SELECT * 
FROM `covidcase-358410.CovidData.CovidDeaths`

--Simple Data exploring Death Data

SELECT location, date, people_fully_vaccinated
FROM `covidcase-358410.CovidData.CovidVaccinations`

--Countries vs Infection Rate

SELECT location, population, MAX(total_cases) as highest_infection_count, MAX((total_cases/population)*100) as percent_population_infected
FROM `covidcase-358410.CovidData.CovidDeaths`
WHERE continent is not null
and continent is not null
GROUP BY location, population 
Order BY percent_population_infected desc

--Continents vs Death Rate

SELECT continent, max (total_deaths) as total_death_count
FROM `covidcase-358410.CovidData.CovidDeaths`
WHERE continent is not null
GROUP BY continent
Order BY total_death_count desc

--Death Percent 

SELECT date, SUM(new_cases) as total_cases, SUM(new_deaths) as total_deaths, SUM(new_deaths)/SUM(new_cases)*100 as death_percentage
FROM `covidcase-358410.CovidData.CovidDeaths`
WHERE location = 'United States'
AND continent is not null
GROUP BY date
ORDER BY 1,2
 
--Join (Reference)

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
From `covidcase-358410.CovidData.CovidDeaths` as dea
Join `covidcase-358410.CovidData.CovidVaccinations` as vac
On dea.location = vac.location
Where dea.continent is not null
and dea.date = vac.date
order By 1,2,3

--Location Population vs Vaccinated

With Pop_Vac AS (Select dea.continent, dea.location, dea.date, dea.population, COALESCE (vac.people_fully_vaccinated,0) as fully_vaccinated 
From `covidcase-358410.CovidData.CovidDeaths` as dea
LEFT JOIN `covidcase-358410.CovidData.CovidVaccinations` as vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
GROUP BY dea.continent, dea.location, dea.date, dea.population, vac.people_fully_vaccinated)
Select *, (fully_vaccinated/Population)*100 as percent_vaccinated
From Pop_Vac

With topmax AS (Select dea.location, dea.population, Max(vac.people_fully_vaccinated) as max_vac
From `covidcase-358410.CovidData.CovidDeaths` as dea
JOIN `covidcase-358410.CovidData.CovidVaccinations` as vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
GROUP BY  dea.location, dea.population)
Select *,(maxvac/Population)*100 as percent_vaccinated
From top_max

Select dea.location, Max(vac.people_fully_vaccinated) as max_vac
From `covidcase-358410.CovidData.CovidDeaths` as dea
JOIN `covidcase-358410.CovidData.CovidVaccinations` as vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null
GROUP BY  dea.location

--World Population vs. Vaccinations 

SELECT vac.location, max(vac.people_fully_vaccinated/dea.population) as world_vac
FROM `covidcase-358410.CovidData.CovidVaccinations` as vac
JOIN `covidcase-358410.CovidData.CovidDeaths` as dea
  On vac.location = dea.location
	and vac.date = dea.date
WHERE vac.location = 'World'
group by vac.location

-- Location Population vs Vaccinations 

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations, SUM(vac.new_vaccinations) OVER (Partition by dea.Location Order by dea.location, dea.Date) as population_vaccinated
FROM `covidcase-358410.CovidData.CovidVaccinations` as vac
JOIN `covidcase-358410.CovidData.CovidDeaths` as dea
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3
