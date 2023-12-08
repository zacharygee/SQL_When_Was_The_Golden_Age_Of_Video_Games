# When Was The Golden Age Of Video Games
This analysis aims to answer a critical question: Are video games getting better, or has the golden age of video games already passed? This README provides information and SQL queries to perform data analysis on the truncated Video Game Sales Data dataset on Kaggle.

## Table of Contents
1. [Identifying the 10 Best Selling Video Games](#identifying-the-10-best-selling-video-games)
2. [Identifying Missing Review Scores](#identifying-missing-review-scores)
3. [Identifying the Years with the Highest Average Critic Score](#identifying-the-years-with-the-highest-average-critic-score)
4. [Adjusting for Number of Games Sold Per Year](#adjusting-for-number-of-games-sold-per-year)
5. [Identifying the Years that Dropped Off the List](#identifying-the-years-that-dropped-off-the-list)
6. [Identifying the Years that Players Loved](#identifying-the-years-that-players-loved)
7. [Identifying the Years that Both Players and Critics Loved](#identifying-the-years-that-both-players-and-critics-loved)
8. [Sales in the Best Years](#sales-in-the-best-years)

## Identifying the 10 Best Selling Video Games
To start off this analysis, I look at the top 10 best selling video games of all time using the following query:
```
SELECT *
FROM game_sales
ORDER BY games_sold DESC
LIMIT 10
```
This query outputs the game title, platform, publisher, developer, number of games sold, and year that the game was released.

## Identifying Missing Review Scores
To explore one of the limitations of the database, I identify some of the missing values using the following query:
```
SELECT COUNT(*)
FROM game_sales AS g
LEFT JOIN reviews AS r
ON g.game = r.game
WHERE r.critic_score IS NULL
AND r.user_score IS NULL
```
This query first joins the two tables, game_sales and reviews, together and then counts the number of NULL critic_score and user_score values.

## Identifying the Years with the Highest Average Critic Score
To find the top 10 years with the highest average critic score, I use the following query:
```
SELECT g.year,
    ROUND(AVG(r.critic_score), 2) AS avg_critic_score
FROM game_sales AS g
LEFT JOIN reviews AS r
ON g.game = r.game
GROUP BY g.year
ORDER BY avg_critic_score DESC
LIMIT 10
```
This query outputs the 10 years with the highest average critic score rounded to two decimal places.

## Adjusting for Number of Games Sold Per Year
The results from using the query above may be skewed due to the number of games released within that year. The following query adjusts for this by only selecting years where the game count is greater than 4.
```
SELECT g.year,
    ROUND(AVG(r.critic_score), 2) AS avg_critic_score,
    COUNT(g.game) AS num_games
FROM game_sales AS g
JOIN reviews AS r
    ON g.game = r.game
GROUP BY g.year
HAVING COUNT(g.game) > 4
ORDER BY avg_critic_score DESC
LIMIT 10
```
By doing this, we are now able to see the top 10 list with years where the number of games released is strictly greater than 4.

## Identifying the Years that Dropped Off the List
To find the years that were removed from the first top 10 list, I used the following query:
```
SELECT year, 
    avg_critic_score
FROM top_critic_years
EXCEPT
SELECT year, 
    avg_critic_score
FROM top_critic_years_more_than_four_games
ORDER BY avg_critic_score DESC
```
This query uses the tables top_critic_years and top_critic_years_more_than_four_games which have the results from the two previous queries stored in them. To find the years that were removed due to having less than 4 games released, I used an EXCEPT clause.

## Identifying the Years that Players Loved
To find the top 10 years where the average user score was the highest, I used the following query:
```
SELECT g.year, 
    ROUND(AVG(user_score), 2) AS avg_user_score, 
    COUNT(g.game) AS num_games
FROM game_sales AS g
INNER JOIN reviews AS r
ON g.game = r.game
GROUP BY g.year
HAVING COUNT(g.game) > 4
ORDER BY avg_user_score DESC
LIMIT 10
```
This query rounds the avg_user_score to two decimal places and adjusts the list to only include years where the number of games released is greater than 4.

## Identifying the Years that Both Players and Critics Loved
To put it all together, I then find the years where both average player and critic scores were the highest using the following query:
```
SELECT year
FROM top_critic_years_more_than_four_games
INTERSECT
SELECT year
FROM top_user_years_more_than_four_games
```
This query uses the top_critic_years_more_than_four_games and top_user_years_more_than_four_games tables which store the previous queries' results. I then use an INTERSECT clause to only pull the years that exist in both tables.

## Sales in the Best Years
Finally, using the results from the previous query, I identify the total number of games sold for those years:
```
SELECT year,
    SUM(games_sold) AS total_games_sold
FROM game_sales
WHERE year IN ('1998', '2008', '2002')
GROUP BY year
ORDER BY total_games_sold DESC
```
