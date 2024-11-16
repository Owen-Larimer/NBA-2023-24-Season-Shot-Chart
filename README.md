# NBA-2023-24-Season-Shot-Chart
A detailed look into NBA shooting statistic leaders by zone throughout the 2023-24 NBA season.

### Project Overview

In this project, the goal was to analyze and visualize shooting statistics for the NBA's 2023-2024 season. By looking at these statistics for various zones, we can glean insights into which players are the best offensively from all areas of the court in terms of efficiency and overall. 

### Data Sources

NBA shot data: The primary dataset used in this project is one of my own making, the "NBA_2023_24_shot_data.xlsx" file, containing a row for each player who played during the season, and their **FGM** (Field Goal Makes), **FGA** (Field Goal Attempts), and **FG%** (Field Goal Percentage) from 6 different areas on the court.
- This data was obtained from the official NBA site [https://www.nba.com/stats](https://www.nba.com/stats). API's exist to pull such data from years past and real time; however, it was simple enough for my needs to just extract the data from their page into excel.

### Tools

- Excel - Initialized Dataset; Some Cleaning
- Jupyter Notebook (Python and Pandas) - Major Cleaning; Some Analysis and Visualization
- Excel and Tableau - Visualization and Interactables
  
### Data Cleaning/Preparation Phase

In cleaning, we performed the following:
1. Removed all duplicate values by creating a secondary set and partitioning the data.
2. Standardized all data by trimming white space.
3. Turned a date column from a text type to a datetime.
4. Repaired any blank or null values we could, and removed the rest

### Exploratory Data Analysis Phase

During the explore phase, we looked at the following:
1. Found maximum people laid off at once.
2. Found companies that went completely under and laid off all employees.
3. Discovered the companies, industries, and countries that had the most total people laid off throughout COVID.

#### Data Analysis Code Examples:

##### Rolling Total of Layoffs
```sql
WITH Rolling_Total AS 
(SELECT SUBSTRING(`date`,1,7) AS `MONTH`, SUM(total_laid_off) AS total_off
FROM layoffs_staging3
WHERE SUBSTRING(`date`,1,7) IS NOT NULL
GROUP BY `MONTH`
ORDER BY 1 ASC
)
SELECT `MONTH`, total_off,
SUM(total_off) OVER(ORDER BY `MONTH`) AS rolling_total
FROM Rolling_Total;
```
##### Layoff Rankings by Year:
```sql
WITH Company_Year (company, years, total_laid_off) AS
(
SELECT company, YEAR(`date`), SUM(total_laid_off)
FROM layoffs_staging3
GROUP BY company, YEAR(`date`)
), Company_Year_Rank AS 
(SELECT *, DENSE_RANK() OVER (PARTITION BY years ORDER BY total_laid_off DESC) AS Ranking
FROM Company_Year
WHERE years is NOT NULL
)
SELECT *
FROM Company_Year_Rank
WHERE Ranking <= 5;
```

### Findings
A summary of some findings from the analysis:
1. The most laid off at a time was 12000 people.
2. Well over 100 companies laid off their entire workforce.
3. Consumer and Retail industries had by far the most layoffs
4. The US had by far the most amount of people laid off
5. A total of 378689 people were laid off (rolling total).
6. Amazon had the most employees laid off at 18150 total
7. Google had the most laid off at one time at 12000, about 6% of it's entire workforce
8. Rankings for top 5 company layoffs for each year within the dataset (layoff rankings).

### Limitations

The main limitation of this project was the fact that the "percentage laid off" column was largely useless to me without having each company's workforce total at the time of the layoffs within the dataset. Therefore, I was only able to use that column when it was 1, as that meant the entire workforce was laid off.

### From Here
The main goal of this project was just to push my SQL ability to its limits. It is still possible to load any of my resulting datasets into Tableau or Pandas to get some solid visualizations that represent the data, and that may be a short addition to this project in the future. However, for now, I've accomplished what I set out to do.
