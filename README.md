# Portfolio_1
SQL Project



--- PART 1
--- 
SELECT location, date, total_cases ,new_cases, total_deaths, population 
FROM Covid_Deaths.dbo.CovidDeaths$
ORDER by 1,2;


--- Looking at total cases vs total deaths

SELECT location, date, total_cases , total_deaths,(total_deaths/total_cases)*100 as percentage_of_Death  
FROM Covid_Deaths.dbo.CovidDeaths$
ORDER by 5 DESC;

--- update: amount of cases and deathsis representative

/** population denstity is at least 1 million
to  improve the case of expected value for percentage of Deaths


**/


 
 
 SELECT  location, continent, population
 FROM Covid_Deaths.dbo.CovidDeaths$
 WHERE POPULATION > 1000000;

 SELECT COUNT(*) 
 FROM Covid_Deaths.dbo.CovidDeaths$
 WHERE POPULATION > 1000000;

 --- result: 69980

--- Select top 20 countries with the highest spread of corona virus

SELECT TOP 20 location, date, total_cases , total_deaths,(total_deaths/total_cases)*100 as percentage_of_Death, population 
FROM Covid_Deaths.dbo.CovidDeaths$
WHERE POPULATION > 1000000
ORDER by 5 DESC;

--- The next confounding factor was time : time must pass to obtain a stable assessment of amount of people infected. 
--- Worldwide beginning 2021
SELECT TOP 20 
    location, 
    date, 
    total_cases, 
    total_deaths,
    (total_deaths / total_cases) * 100 AS percentage_of_Death, 
    population  
FROM 
    Covid_Deaths.dbo.CovidDeaths$
WHERE 
    population > 1000000 
    AND date = '2021-01-01'
ORDER BY 
    (total_deaths / total_cases) DESC;


	--- worldwide eginning 2022

	SELECT TOP 20 
    location, 
    date, 
    total_cases, 
    total_deaths,
    (total_deaths / total_cases) * 100 AS percentage_of_Death, 
    population  
FROM 
    Covid_Deaths.dbo.CovidDeaths$
WHERE 
    population > 1000000 
    AND date = '2021-01-01'
ORDER BY 
    (total_deaths / total_cases) DESC;






--- in Europe
SELECT TOP 20 
    location, 
    date, 
    total_cases, 
    total_deaths,
    (total_deaths / total_cases) * 100 AS percentage_of_Death, 
    population  
FROM 
    Covid_Deaths.dbo.CovidDeaths$
WHERE 
    population > 1000000 
    AND date = '2021-01-01'
	AND continent = 'Europe'
ORDER BY 
    (total_deaths / total_cases) DESC;





	---- last date for each country
SELECT TOP 20
    cd.location, 
    cd.date, 
    cd.total_cases, 
    cd.total_deaths,
    (cd.total_deaths / cd.total_cases) * 100 AS percentage_of_Death, 
    cd.population  
FROM 
    Covid_Deaths.dbo.CovidDeaths$ cd
JOIN (
    SELECT 
        location, 
        MAX(date) AS max_date
    FROM 
        Covid_Deaths.dbo.CovidDeaths$
    GROUP BY 
        location
) max_dates ON cd.location = max_dates.location AND cd.date = max_dates.max_date
WHERE 
    cd.population > 1000000
ORDER BY 
    (cd.total_deaths / cd.total_cases) DESC;



---The latest observations indicate that Yemen has the highest death rates, followed by Mexico and then Syria.
--- europe wide last date 

SELECT TOP 5
    cd.location, 
    cd.date, 
    cd.total_cases, 
    cd.total_deaths,
    (cd.total_deaths / cd.total_cases) * 100 AS percentage_of_Death, 
    cd.population  
FROM 
    Covid_Deaths.dbo.CovidDeaths$ cd
JOIN (
    SELECT 
        location, 
        MAX(date) AS max_date
    FROM 
        Covid_Deaths.dbo.CovidDeaths$
    GROUP BY 
        location
) max_dates ON cd.location = max_dates.location AND cd.date = max_dates.max_date
WHERE 
    cd.population > 1000000
	AND continent = 'Europe'
ORDER BY 
    (cd.total_deaths / cd.total_cases) DESC;

--- Across Europe, the data shows the highest death rates in Bosnia, followed by Bulgaria, and then Hungary.


--- PART 2


--- Looking at the Countries with the Highest Infection Rate compared to Population.

SELECT location, population ,MAX(total_cases) as HighestInfection_count
FROM Covid_Deaths.dbo.CovidDeaths$
GROUP by population, location
ORDER by 1,2;

--- Global Numbers

Select SUM(new_cases) as total_cases, SUM(cast(new_deaths as int)) as total_deaths, SUM(cast(new_deaths as int))/SUM(New_Cases)*100 as DeathPercentage
From Covid_Deaths.dbo.CovidDeaths$
--Where location like '%states%'
where continent is not null 
--Group By date
order by 1,2



-- Total Population vs Vaccinations
-- Shows Percentage of Population that has recieved at least one Covid Vaccine

Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From Covid_Deaths.dbo.CovidDeaths$ dea
Join Covid_Vaccinations.dbo.CovidVaccinations$ vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 
order by 2,3












-- Using CTE to perform Calculation on Partition By in previous query

WITH PopvsVac (Continent, Location, Date, Population, New_Vaccinations, RollingPeopleVaccinated, PercentageVaccinated)
AS
(SELECT 
        dea.continent, 
        dea.location, 
        dea.date, 
        dea.population, 
        vac.new_vaccinations,
        SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) AS RollingPeopleVaccinated,
        (SUM(CONVERT(int, vac.new_vaccinations)) OVER (PARTITION BY dea.location ORDER BY dea.location, dea.date) / dea.population) * 100 AS PercentageVaccinated
    FROM 
        Covid_Deaths.dbo.CovidDeaths$ dea
    JOIN 
        Covid_Vaccinations.dbo.CovidVaccinations$ vac
    ON 
        dea.location = vac.location
        AND dea.date = vac.date
    WHERE 
        dea.continent IS NOT NULL )





Select *, (RollingPeopleVaccinated/Population)*100
From PopvsVac



-- Using Temp Table to perform Calculation on Partition By in previous query

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
From Covid_Deaths.dbo.CovidDeaths$ dea
Join Covid_Vaccinations.dbo.CovidVaccinations$ vac
	On dea.location = vac.location
	and dea.date = vac.date
--where dea.continent is not null 
--order by 2,3

Select *, (RollingPeopleVaccinated/Population)*100
From #PercentPopulationVaccinated




-- Creating View to store data for later visualizations

Create View PercentPopulationVaccinated as
Select dea.continent, dea.location, dea.date, dea.population, vac.new_vaccinations
, SUM(CONVERT(int,vac.new_vaccinations)) OVER (Partition by dea.Location Order by dea.location, dea.Date) as RollingPeopleVaccinated
--, (RollingPeopleVaccinated/population)*100
From PortfolioProject..CovidDeaths dea
Join PortfolioProject..CovidVaccinations vac
	On dea.location = vac.location
	and dea.date = vac.date
where dea.continent is not null 



