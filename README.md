# Video Game Sales Analysis using MySQL
Exploratory data analysis on video game sales.

![pexels-pixabay-163036](https://user-images.githubusercontent.com/46164783/153789710-556da471-c27f-44d7-a00d-d392d4c6b288.jpg)


## About Dataset

The dataset is sourced from [kaggle](https://www.kaggle.com/gregorut/videogamesales) which contains a list of video games with sales from 1980 to 2020. The dataset has 11 columns and 16598  records. I decided to drop records with null values in order to make the result of the analysis more accurate. In addition, I split the dataset into two related tables based on the principle of data normalization - the "game_info" table contains the basic information of the game, and the "game_sales" table contains the sales records of the game in various regions. 

## Using Temporary Table
Since we will need to join both tables multiple times to perform analysis, I would create a temporary table for resusiblity purpose. Using temporary can make the query a lot shorter and also speed up the runtime to increase efficiency.
``` sql
CREATE TEMPORARY TABLE game_sales_temp
SELECT
    i.game_id,
    name,
    platform,
    year,
    genre,
    publisher,
    na_sales,
    eu_sales,
    jp_sales,
    other_sales,
    global_sales
FROM game_info i
JOIN game_sales s
	ON i.game_id = s.game_id;

SELECT * FROM game_sales_temp;

``` 

Let’s take a look of the completed dataset
![1completed_dataset](https://user-images.githubusercontent.com/46164783/153784114-37df7c84-5783-44bf-ad50-e673c5c3ed14.jpg)

###### Denotation of each columns:
- game_id- unique game id
- name - The games name
- platform - Platform of the games release (i.e. PC,PS4, etc.)
- year - Year of the game's release
- genre - Genre of the game
- publisher - Publisher of the game
- na_sales - Sales in North America (in millions)
- eu_sales - Sales in Europe (in millions)
- jp_sales - Sales in Japan (in millions)
- other_sales - Sales in the rest of the world (in millions)
- global_sales - Total worldwide sales.

## Data Analysis
### 1. Find the top 3 most popular platforms in terms of counts of games.
``` sql
SELECT 
	platform,
    COUNT(name) AS num_games
FROM game_sales_temp
GROUP BY 1
ORDER BY 2 DESC
LIMIT 3;
```
![2top3_platforms](https://user-images.githubusercontent.com/46164783/153784398-3dcab8fe-5e21-45df-aa48-dfd42b565c0f.jpg)
![top3_platforms_by_num_games](https://user-images.githubusercontent.com/46164783/153784507-4472c480-44fd-41ac-987e-f883cd1f6451.jpg)
- DS, PS2 and PS3 are three dominant platforms globally

### 2. A specific game can be played on multiple platforms, find games which were played on more than 5 platforms
``` sql
SELECT 
	name,
    COUNT(platform) AS num_platform
FROM game_sales_temp
GROUP BY 1
HAVING COUNT(platform) > 5
ORDER BY 2 DESC;
``` 
![3game_more_than_5_platforms](https://user-images.githubusercontent.com/46164783/153784781-fc2652fc-751f-4714-8447-138a1edf76e0.jpg)

- There are 133 games were played on more than 5 platforms
- 'Need for Speed: Most Wanted' is the game that played on most platforms


###  3. report the genre with the most global sales for each platform
``` sql
WITH cte AS(
SELECT
	platform,
    genre,
    ROUND(SUM(global_sales),2) AS total_sales,
	DENSE_RANK() OVER (PARTITION BY platform ORDER BY ROUND(SUM(global_sales),2) DESC) AS rank_sales
FROM game_sales_temp
GROUP BY 1,2)
SELECT
	platform,
    genre,
    total_sales
FROM cte
WHERE rank_sales = 1
ORDER BY total_sales DESC;
```
![4platform_genre_sales](https://user-images.githubusercontent.com/46164783/153784981-35fdebfb-4f04-4847-9ea8-39d742417d1c.jpg)

### 4. Find the top 3 video games that have the most sales for each genre 
``` sql
WITH cte AS (
SELECT
	genre,
    name,
    ROUND(SUM(global_sales),2) AS total_sales,
    ROW_NUMBER() OVER (PARTITION BY genre ORDER BY SUM(global_sales) DESC) AS sales_rank
FROM game_sales_temp
GROUP BY 1,2
)
SELECT * FROM cte WHERE sales_rank <= 3;
```
![5top3_sales_each_genre](https://user-images.githubusercontent.com/46164783/153785079-a2650689-3ffb-436e-b783-3939adb523ab.jpg)

### 5. Find the percentage of global sales amount for each genre
``` sql
SELECT 
	genre,
    ROUND(SUM(global_sales),2) AS total_sales,
    ROUND((SUM(global_sales)/(SELECT SUM(global_sales) FROM game_sales_completed))*100,2) AS 'sale%'
FROM game_sales_temp
GROUP BY 1
ORDER BY 3 DESC;
```
![6percent_sales_by_genres](https://user-images.githubusercontent.com/46164783/153785324-c87787b1-79e5-4735-91ec-031de6104bbf.jpg)
![sale%_by_genre](https://user-images.githubusercontent.com/46164783/153785288-998be3d7-31da-40e9-b3f9-198ec250d3e3.jpg)

### 6. Find game type prefernces for each region
``` sql
SELECT
	genre,
    ROUND(SUM(NA_Sales),2) AS North_America_sales,
	ROUND(SUM(EU_Sales),2) AS Europe_sales,
    ROUND(SUM(JP_Sales),2) AS Japan_sales
FROM game_sales_temp
GROUP BY 1;
-- ORDER BY 2 DESC #most popular game in North America
-- ORDER BY 3 DESC #most popular game in North Europe
-- ORDER BY 4 DESC #most popular game in North Japan;
```
![regional_genre](https://user-images.githubusercontent.com/46164783/153785489-42d8aa13-9341-48d5-9712-871c20919610.jpg)
![regional_genreT](https://user-images.githubusercontent.com/46164783/153785561-6cf473d6-fc02-49fe-8f11-91158acf9ed6.jpg)
- Action game is the most popular game type in both North America and Europe
- Role-playing game is the most popular game in Japan

### 7. Find publisher whose sales greater than the average global sales of all publishers.
``` sql
WITH cte AS
(SELECT
	publisher,
    ROUND(SUM(global_sales),2) AS total_sales
FROM game_info i
LEFT JOIN game_sales s
	ON i.game_id = s.game_id
GROUP BY 1
ORDER BY 2 DESC) 
SELECT 
	publisher,
    total_sales
FROM cte
WHERE total_sales > (SELECT AVG(total_sales) AS avg_sales FROM cte);
```
![pulisher_sale_greater_avg](https://user-images.githubusercontent.com/46164783/153785807-aaf42665-b23c-4f69-a50f-2abf581228f2.jpg)
- There are total 39 publishers have total global sales greater than the average global sales of all publishers.

### 8. Find the year with the most games released for each genre
``` sql
WITH cte AS(
SELECT
	genre,
    year,
    COUNT(*) AS num_games
FROM game_info i
LEFT JOIN game_sales s
	ON i.game_id = s.game_id
GROUP BY 1,2
ORDER BY 1,2)
SELECT
	genre,
    year,
    num_games
FROM cte
WHERE (genre, num_games) IN (SELECT genre, MAX(num_games) AS most_releases FROM cte
							 GROUP BY 1)
ORDER BY year;
```
![year_genre_most_sales](https://user-images.githubusercontent.com/46164783/153786032-60cf2dcb-59b2-4fed-8a00-030c41f7b94f.jpg)
- We can see all genres have most sales during the period of 2003 - 2009.

### 9. Find the year, number of games and their names that sold more than 15 million worldwide. Sort the game name lexicographically.
``` sql
SELECT
	year,
    COUNT(DISTINCT name) AS num_movie,
    GROUP_CONCAT(DISTINCT name ORDER BY name SEPARATOR ', ') AS 'movie(s)'
FROM game_sales_temp
WHERE global_sales > 15
GROUP BY 1
ORDER BY 1;
```
![games_with_15_sales](https://user-images.githubusercontent.com/46164783/153786478-caa19c79-9f66-4c9c-9523-8f9744fce5f0.jpg)

### 10. Report the total number of video games that released in in each decade, exclude the 20s 
``` sql
SELECT
	ROUND(SUM(CASE WHEN year LIKE '198%' THEN global_sales ELSE 0 END),2) AS num_80s_games,
	ROUND(SUM(CASE WHEN year LIKE '199%' THEN global_sales ELSE 0 END),2) AS num_90s_games,
	ROUND(SUM(CASE WHEN year LIKE '200%' THEN global_sales ELSE 0 END),2) AS num_00s_games,
	ROUND(SUM(CASE WHEN year LIKE '201%' THEN global_sales ELSE 0 END),2) AS num_10s_games
FROM game_sales_temp;
```
![decade_game_sales](https://user-images.githubusercontent.com/46164783/153787080-7cf4e596-fe14-44c4-8aac-76a99ec4d97a.jpg)
![game_sales_by_years](https://user-images.githubusercontent.com/46164783/153787342-ff6764b9-a484-4af7-9ff2-65bd8ab3d3c3.jpg)
- Consecutive years from 2002 - 2011 are the top 10 years that have the most sales, peaked in 2008.

### 11. Which region performed the best in terms of sales?
``` sql
SELECT * FROM game_sales_completed;
SELECT
	ROUND(SUM(NA_Sales),2) AS na_sales,
    ROUND((SUM(NA_Sales)/SUM(global_sales))*100,2) AS 'na_sales%',
    ROUND(SUM(EU_Sales),2) AS eu_sales,
	ROUND((SUM(EU_Sales)/SUM(global_sales))*100,2) AS 'eu_sales_ratio%',
    ROUND(SUM(JP_Sales),2) AS jp_sales,
    ROUND((SUM(JP_Sales)/SUM(global_sales))*100,2) AS 'jp_sales%',
    ROUND(SUM(Other_Sales),2) AS other_sales,
    ROUND((SUM(Other_Sales)/SUM(global_sales))*100,2) AS 'other_sales%'
FROM game_sales_temp;
```
![regional_sale%](https://user-images.githubusercontent.com/46164783/153787524-aeafe086-a5cc-4b16-b66a-13ad823997e4.jpg)
![region_sale%](https://user-images.githubusercontent.com/46164783/153787598-a55695d3-7b4f-45a8-8328-7e32268cf628.jpg)
- North America has the half of the global sales
