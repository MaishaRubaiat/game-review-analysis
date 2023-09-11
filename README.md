# game-review-analysis
Explore gaming history: Analyze critic/user scores &amp; sales data for top 400 games since '77, uncover the Golden Age &amp; business insights.
# find the ten best-selling video games in game_sales
%%sql
postgresql:///games
SELECT *
FROM game_sales
ORDER BY games_sold DESC
LIMIT 10
# determine how many games in the game_sales table are missing both a user_score and a critic_score
SELECT COUNT(g.game)
FROM game_sales g
LEFT JOIN reviews r
ON g.game = r.game
WHERE r.critic_score IS NULL AND r.user_score IS NULL
# Find the years with the highest average critic_score
SELECT g.year, ROUND(AVG(critic_score), 2) AS avg_critic_score
FROM game_sales g
JOIN reviews r
ON g.game = r.game
GROUP BY g.year
ORDER BY avg_critic_score DESC
LIMIT 10
# Find game critics' ten favorite years, this time with the stipulation that a year must have more than four games released in order to be considered.
SELECT g.year, ROUND(AVG(critic_score), 2) AS avg_critic_score, COUNT(g.game) AS num_games
FROM game_sales g
JOIN reviews r
ON g.game = r.game
GROUP BY g.year
HAVING COUNT(g.game) > 4
ORDER BY avg_critic_score DESC
LIMIT 10
# find the years that were on our first critics' favorite list but not the second.
SELECT year, avg_critic_score
FROM top_critic_years
EXCEPT 
SELECT year, avg_critic_score
FROM top_critic_years_more_than_four_games
ORDER BY avg_critic_score DESC
# Update your query from Task Four so that it returns years with ten highest avg_user_score values.
SELECT g.year, ROUND(AVG(user_score), 2) AS avg_user_score, COUNT(g.game) AS num_games
FROM game_sales g
JOIN reviews r
ON g.game = r.game
GROUP BY g.year
HAVING COUNT(g.game) > 4
ORDER BY avg_user_score DESC
LIMIT 10
# Create a list of years that appear on both the top_critic_years_more_than_four_games table and the top_user_years_more_than_four_games table.
SELECT year
FROM top_critic_years_more_than_four_games
INTERSECT
SELECT year
FROM top_user_years_more_than_four_games
# Add a column showing total games_sold in each year to the table you created in the previous task
SELECT year, SUM(games_sold) AS total_games_sold
FROM game_sales 
WHERE year IN
 (SELECT year
  FROM top_critic_years_more_than_four_games
  INTERSECT
  SELECT year
  FROM top_user_years_more_than_four_games)
GROUP BY year
ORDER BY total_games_sold DESC
