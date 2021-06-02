Clean up with CTEs
In chapter 2, you generated a list of countries and the number of matches in each country with more than 10 total goals. 
The query in that exercise utilized a subquery in the FROM statement in order to filter the matches before counting them in the main query. 
Below is the query you created:

SELECT
  c.name AS country,
  COUNT(sub.id) AS matches
FROM country AS c
INNER JOIN (
  SELECT country_id, id 
  FROM match
  WHERE (home_goal + away_goal) >= 10) AS sub
ON c.id = sub.country_id
GROUP BY country;
You can list one (or more) subqueries as common table expressions (CTEs) by declaring them ahead of your main query, 
which is an excellent tool for organizing information and placing it in a logical order.

In this exercise, let's rewrite a similar query using a CTE.

Instructions
100 XP
Complete the syntax to declare your CTE.
Select the country_id and match id from the match table in your CTE.
Left join the CTE to the league table using country_id.

-- Set up your CTE
with match_list as (
    SELECT 
  		country_id, 
  		id
    FROM match
    WHERE (home_goal + away_goal) >= 10)
-- Select league and count of matches from the CTE
SELECT
    l.name AS league,
    COUNT(match_list.id) AS matches
FROM league AS l
-- Join the CTE to the league table
LEFT JOIN match_list ON l.id = match_list.country_id
GROUP BY l.name;

Organizing with CTEs
Previously, you modified a query based on a statement you completed in chapter 2 using common table expressions.

This time, let's expand on the exercise by looking at details about matches with very high scores using CTEs. 
Just like a subquery in FROM, you can join tables inside a CTE.

Instructions
Declare your CTE, where you create a list of all matches with the league name.
Select the league, date, home, and away goals from the CTE.
Filter the main query for matches with 10 or more goals.

-- Set up your CTE
With match_list as (
  -- Select the league, date, home, and away goals
    SELECT 
  		l.name AS league, 
     	l.country_id, 
       m.date,
  		m.home_goal, 
  		m.away_goal,
       (m.home_goal + m.away_goal) AS total_goals
    FROM match AS m
    LEFT JOIN league as l ON m.country_id = l.id)
-- Select the league, date, home, and away goals from the CTE
SELECT league, date, home_goal, away_goal
FROM match_list
-- Filter by total goals
WHERE total_goals >=10;

Declare a CTE that calculates the total goals from matches in August of the 2013/2014 season.
Left join the CTE onto the league table using country_id from the match_list CTE.
Filter the list on the inner subquery to only select matches in August of the 2013/2014 season.

-- Set up your CTE
with match_list as (
    SELECT 
  		country_id,
  	   (home_goal + away_goal) AS goals
    FROM match
  	-- Create a list of match IDs to filter data in the CTE
    WHERE id IN (
       SELECT id
       FROM match
       WHERE season = '2013/2014' AND EXTRACT(MONTH FROM date) = 08))
-- Select the league name and average of goals in the CTE
SELECT 
	l.name,
    avg(match_list.goals)
FROM league AS l
-- Join the CTE onto the league table
LEFT JOIN match_list ON l.id = match_list.country_id
GROUP BY l.name;

Get team names with a subquery
Let's solve a problem we've encountered a few times in this course so far -- 
How do you get both the home and away team names into one final query result?

Out of the 4 techniques we just discussed, this can be performed using subqueries, correlated subqueries, and CTEs. 
Let's practice creating similar result sets using each of these 3 methods over the next 3 exercises, starting with subqueries in FROM.

Instructions 1/2
Create a query that left joins team to match in order to get the identity of the home team. This becomes the subquery in the next step.

SELECT 
	m.id, 
    t.team_long_name AS hometeam
-- Left join team to match
FROM match AS m
inner join team as t
ON m.hometeam_id = team_api_id;

Instructions 2/2
50 XP
2
Add a second subquery to the FROM statement to get the away team name, changing only the hometeam_id. Left join both subqueries to the match table on the id column.
Warning: if your code is timing out, you have probably made a mistake in the JOIN and tried to join on the wrong fields which caused the table to be too big! 
Read the provided code and comments carefully, and check your ON conditions!

SELECT
	m.date,
    -- Get the home and away team names
    hometeam,
    awayteam,
    m.home_goal,
    m.away_goal
FROM match AS m

-- Join the home subquery to the match table
LEFT JOIN (
  SELECT match.id, team.team_long_name AS hometeam
  FROM match
  LEFT JOIN team
  ON match.hometeam_id = team.team_api_id) AS home
ON home.id = m.id

-- Join the away subquery to the match table
LEFT JOIN (
  SELECT match.id, team.team_long_name AS awayteam
  FROM match
  LEFT JOIN team
  -- Get the away team ID in the subquery
  ON match.awayteam_id = team.team_api_id) AS away
ON away.id = m.id;

Get team names with correlated subqueries
Let's solve the same problem using correlated subqueries -- How do you get both the home and away team names into one final query result?

This can easily be performed using correlated subqueries. But how might that impact the performance of your query? 
Complete the following steps and let's find out!

Please note that your query will run more slowly than the previous exercise!

Instructions 1/2
Using a correlated subquery in the SELECT statement, match the team_api_id column from team to the hometeam_id from match.

SELECT
    m.date,
   (SELECT team_long_name
    FROM team AS t
    -- Connect the team to the match table
    WHERE t.team_api_id = hometeam_id) AS hometeam
FROM match AS m;

Instructions 2/2
Create a second correlated subquery in SELECT, yielding the away team's name.
Select the home and away goal columns from match in the main query.

SELECT
    m.date,
    (SELECT team_long_name
     FROM team AS t
     WHERE t.team_api_id = m.hometeam_id) AS hometeam,
    -- Connect the team to the match table
    (SELECT team_long_name
     FROM team AS t
     WHERE t.team_api_id = awayteam_id) AS awayteam,
    -- Select home and away goals
     m.home_goal,
     m.away_goal
FROM match AS m;

Get team names with CTEs
You've now explored two methods for answering the question, How do you get both the home and away team names into one final query result?

Let's explore the final method - common table expressions. Common table expressions are similar to the subquery method for generating results, 
mainly differing in syntax and the order in which information is processed.

Instructions 1/3
Select id from match and team_long_name from team. Join these two tables together on hometeam_id in match and team_api_id in team.

SELECT 
	-- Select match id and team long name
    m.id, 
    t.team_long_name AS hometeam
FROM match AS m
-- Join team to match using team_api_id and hometeam_id
LEFT JOIN team AS t 
ON m.hometeam_id = t.team_api_id;

Declare the query from the previous step as a common table expression. SELECT everything from the CTE into the main query. 
Your results will not change at this step!

-- Declare the home CTE
with home as (
	SELECT m.id, t.team_long_name AS hometeam
	FROM match AS m
	LEFT JOIN team AS t 
	ON m.hometeam_id = t.team_api_id)
-- Select everything from home
SELECT *
FROM home;

Instructions 3/3
Let's declare the second CTE, away. Join it to the first CTE on the id column.
The date, home_goal, and away_goal columns have been added to the CTEs. SELECT them into the main query.

WITH home AS (
  SELECT m.id, m.date, 
  		 t.team_long_name AS hometeam, m.home_goal
  FROM match AS m
  LEFT JOIN team AS t 
  ON m.hometeam_id = t.team_api_id),
-- Declare and set up the away CTE
away as (
  SELECT m.id, m.date, 
  		 t.team_long_name AS awayteam, m.away_goal
  FROM match AS m
  LEFT JOIN team AS t 
  ON m.awayteam_id = t.team_api_id)
-- Select date, home_goal, and away_goal
SELECT 
	home.date,
    home.hometeam,
    away.awayteam,
    home.home_goal,
    away.away_goal
-- Join away and home on the id column
FROM home
INNER JOIN away
ON home.id = away.id;

The match is OVER
The OVER() clause allows you to pass an aggregate function down a data set, similar to subqueries in SELECT. 
The OVER() clause offers significant benefits over subqueries in select -- namely, your queries will run faster, 
and the OVER() clause has a wide range of additional functions and clauses you can include with it that we will cover later on in this chapter.

In this exercise, you will revise some queries from previous chapters using the OVER() clause.

Instructions
Select the match ID, country name, season, home, and away goals from the match and country tables.
Complete the query that calculates the average number of goals scored overall and then includes the aggregate value in each row using a window function.

SELECT 
	-- Select the id, country name, season, home, and away goals
	m.id, 
    c.name AS country, 
    m.season,
	m.home_goal,
	m.away_goal,
    -- Use a window to include the aggregate average in each row
	avg(m.home_goal + m.away_goal) over() AS overall_avg
FROM match AS m
LEFT JOIN country AS c ON m.country_id = c.id;

What's OVER here?
Window functions allow you to create a RANK of information according to any variable you want to use to sort your data. 
When setting this up, you will need to specify what column/calculation you want to use to calculate your rank. This is done by including an ORDER BY clause inside the OVER() clause. Below is an example:

SELECT 
    id,
    RANK() OVER(ORDER BY home_goal) AS rank
FROM match;
In this exercise, you will create a data set of ranked matches according to which leagues, on average, score the most goals in a match.

Instructions
Select the league name and average total goals scored from league and match.
Complete the window function so it calculates the rank of average goals scored across all leagues in the database.
Order the rank by the average total of home and away goals scored.

SELECT 
	-- Select the league name and average goals scored
	l.name AS league,
    avg(m.home_goal + m.away_goal) AS avg_goals,
    -- Rank each league according to the average goals
    rank() over(order by AVG(m.home_goal + m.away_goal)) AS league_rank
FROM league AS l
LEFT JOIN match AS m 
ON l.id = m.country_id
WHERE m.season = '2011/2012'
GROUP BY l.name
-- Order the query by the rank you created
ORDER BY league_rank;

output
league			avg_goals		league_rank
Poland Ekstraklasa	2.1958333333333333	1
France Ligue 1		2.5157894736842105	2
Italy Serie A		2.5837988826815642	3
Switzerland Super League2.6234567901234568	4
Scotland Premier League	2.6359649122807018	5
Portugal Liga ZON Sagres2.6416666666666667	6
Spain LIGA BBVA		2.7631578947368421	7
England Premier League	2.8052631578947368	8
Germany 1. Bundesliga	2.8594771241830065	9
Belgium 


Flip OVER your results
In the last exercise, the rank generated in your query was organized from smallest to largest. 
By adding DESC to your window function, you can create a rank sorted from largest to smallest.

SELECT 
    id,
    RANK() OVER(ORDER BY home_goal DESC) AS rank
FROM match;
Instructions
100 XP
Complete the same parts of the query as the previous exercise.
Complete the window function to rank each league from highest to lowest average goals scored.
Order the main query by the rank you just created.

SELECT 
	-- Select the league name and average goals scored
	l.name AS league,
    avg(m.home_goal + m.away_goal) AS avg_goals,
    -- Rank leagues in descending order by average goals
    rank() over(order by avg(m.home_goal + m.away_goal) desc) AS league_rank
FROM league AS l
LEFT JOIN match AS m 
ON l.id = m.country_id
WHERE m.season = '2011/2012'
GROUP BY l.name
-- Order the query by the rank you created
order by league_rank;

PARTITION BY a column
The PARTITION BY clause allows you to calculate separate "windows" based on columns you want to divide your results. 
For example, you can create a single column that calculates an overall average of goals scored for each season.

In this exercise, you will be creating a data set of games played by Legia Warszawa (Warsaw League), the top ranked team in Poland, 
and comparing their individual game performance to the overall average for that season.

Where do you see more outliers? Are they Legia Warszawa's home or away games?

Complete the two window functions that calculate the home and away goal averages. 
Partition the window functions by season to calculate separate averages for each season.
Filter the query to only include matches played by Legia Warszawa, id = 8673.

SELECT
	date,
	season,
	home_goal,
	away_goal,
	CASE WHEN hometeam_id = 8673 THEN 'home' 
		 ELSE 'away' END AS warsaw_location,
    -- Calculate the average goals scored partitioned by season
    avg(home_goal) OVER(PARTITION BY season) AS season_homeavg,
    avg(away_goal) OVER(PARTITION BY season) AS season_awayavg
FROM match
-- Filter the data set for Legia Warszawa matches only
WHERE 
	hometeam_id = 8673 
    OR awayteam_id = 8673
ORDER BY (home_goal + away_goal) DESC;

date		season	   home_goal away_goal	warsaw_location	season_homeavg	season_awayavg
2013-09-14	2013/2014	3	5	away	1.7666666666666667	1.2333333333333333
2014-09-13	2014/2015	4	3	home	1.5666666666666667	1.3333333333333333
2013-07-20	2013/2014	5	1	home	1.7666666666666667	1.2333333333333333
2013-10-20	2013/2014	4	1	home	1.7666666666666667	1.2333333333333333
2013-06-02	2012/2013	5	0	home	1.5666666666666667	1.1333333333333333
2013-02-23	2012/2013	3	2	away	1.5666666666666667	1.1333333333333333
2014-08-09	2014/2015	5	0	home	1.5666666666666667	1.3333333333333333
2012-10-28	2012/2013	3	2	home	1.5666666666666667	1.1333333333333333
2013-09-25	2013/2014	2	3	away	1.7666666666666667	1.2333333333333333

PARTITION BY multiple columns
The PARTITION BY clause can be used to break out window averages by multiple data points (columns). 
You can even calculate the information you want to use to partition your data! For example, 
you can calculate average goals scored by season and by country, or by the calendar year (taken from the date column).

In this exercise, you will calculate the average number home and away goals scored Legia Warszawa, and their opponents, 
partitioned by the month in each season.

Construct two window functions partitioning the average of home and away goals by season and month.
Filter the data set by Legia Warszawa's team ID (8673) so that the window calculation excludes all teams who did not play against them.

SELECT 
	date,
	season,
	home_goal,
	away_goal,
	CASE WHEN hometeam_id = 8673 THEN 'home' 
         ELSE 'away' END AS warsaw_location,
	-- Calculate average goals partitioned by season and month
    avg(home_goal) over(partition by season, 
         	EXTRACT(month FROM date)) AS season_mo_home,
    avg(away_goal) over(partition by season, 
            EXTRACT(month FROM date)) AS season_mo_away
FROM match
WHERE 
	hometeam_id = 8673 
    OR awayteam_id = 8673
ORDER BY (home_goal + away_goal) DESC;

date	        season	   home_goal away_goal	warsaw_location	season_mo_home	season_mo_away
2013-09-14	2013/2014	3	5	away	2.2500000000000000	2.5000000000000000
2014-09-13	2014/2015	4	3	home	2.0000000000000000	2.6666666666666667
2013-07-20	2013/2014	5	1	home	2.5000000000000000	2.0000000000000000
2014-08-09	2014/2015	5	0	home	2.0000000000000000	1.00000000000000000000
2012-10-28	2012/2013	3	2	home	1.6666666666666667	2.0000000000000000
2013-06-02	2012/2013	5	0	home	5.0000000000000000	0E-20
2013-12-15	2013/2014	4	1	home	2.2500000000000000	0.25000000000000000000
2013-10-20	2013/2014	4	1	home	2.2500000000000000	0.75000000000000000000
2013-09-25	2013/2014	2	3	away	2.2500000000000000	2.5000000000000000
2013-02-23	2012/2013	3	2	away	3.0000000000000000	2.0000000000000000
2015-03-08	2014/2015	1	3	away	2.0000000000000000	1.5000000000000000
2015-02-15	2014/2015	1	3	home	0.50000000000000000000	1.5000000000000000
2012-11-18	2012/2013	1	3	away	1.00000000000000000000	1.7500000000000000
2011-08-07	2011/2012	1	3	away	1.5000000000000000	2.2500000000000000
2011-08-12	2011/2012	3	1	home	1.5000000000000000	2.2500000000000000
2013-11-23	2013/2014	3	1	home	1.6666666666666667	0.66666666666666666667
2011-08-29	2011/2012	1	3	away	1.5000000000000000	2.2500000000000000
2012-09-29	2012/2013	2	2	away	2.0000000000000000	1.5000000000000000
2012-02-26	2011/2012	0	4	away	1.00000000000000000000	2.0000000000000000


Sliding windows
Slide to the left
Sliding windows allow you to create running calculations between any two points in a window using functions 
such as PRECEDING, FOLLOWING, and CURRENT ROW. You can calculate running counts, sums, averages, and other aggregate functions 
between any two points you specify in the data set.

In this exercise, you will expand on the examples discussed in the video, calculating the running total of goals scored by the 
FC Utrecht when they were the home team during the 2011/2012 season. Do they score more goals at the end of the season as the home or away team?

Instructions
100 XP
Complete the window function by:
Assessing the running total of home goals scored by FC Utrecht.
Assessing the running average of home goals scored.
Ordering both the running average and running total by date.

SELECT 
	date,
	home_goal,
	away_goal,
    -- Create a running total and running average of home goals
    sum(home_goal) over(ORDER BY date 
         ROWS BETWEEN unbounded preceding AND current row) AS running_total,
    avg(home_goal) over(ORDER BY date 
         ROWS BETWEEN unbounded preceding AND current row) AS running_avg
FROM match
WHERE 
	hometeam_id = 9908 
	AND season = '2011/2012';

date	   home_goal away_goal	running_total	running_avg
2011-08-14	2	2	2	2.0000000000000000
2011-08-27	3	1	5	2.5000000000000000
2011-09-18	2	2	7	2.3333333333333333
2011-10-01	3	0	10	2.5000000000000000
2011-10-22	1	4	11	2.2000000000000000
2011-11-06	6	4	17	2.8333333333333333
2011-12-04	2	6	19	2.7142857142857143
2011-12-11	2	2	21	2.6250000000000000
2012-01-22	1	1	22	2.4444444444444444
2012-02-12	1	1	23	2.3000000000000000
2012-02-19	3	0	26	2.3636363636363636
2012-03-04	0	0	26	2.1666666666666667
2012-03-18	3	1	29	2.2307692307692308
2012-03-30	3	2	32	2.2857142857142857
2012-04-15	4	2	36	2.4000000000000000
2012-04-28	1	3	37	2.3125000000000000
2012-05-02	2	2	39	2.2941176470588235

Slide to the right
Now let's see how FC Utrecht performs when they're the away team. You'll notice that the total for the season is at the bottom of the data set you queried. 
Depending on your results, this could be pretty long, and scrolling down is not very helpful.

In this exercise, you will slightly modify the query from the previous exercise by sorting the data set in reverse order and 
calculating a backward running total from the CURRENT ROW to the end of the data set (earliest record).

Instructions
Complete the window function by:
Assessing the running total of home goals scored by FC Utrecht.
Assessing the running average of home goals scored.
Ordering both the running average and running total by date, descending.

SELECT 
	-- Select the date, home goal, and away goals
     date,
    home_goal,
    away_goal,
    -- Create a running total and running average of home goals
    sum(home_goal) over(ORDER BY date DESC
         ROWS BETWEEN unbounded preceding AND current row) AS running_total,
    avg(home_goal) over(ORDER BY date DESC
         ROWS BETWEEN unbounded preceding AND current row) AS running_avg
FROM match
WHERE 
	awayteam_id = 9908 
    AND season = '2011/2012';

date	   home_goal away_goal	running_total	running_avg
2012-05-06	1	3	1	1.00000000000000000000
2012-04-21	0	2	1	0.50000000000000000000
2012-04-12	3	0	4	1.3333333333333333
2012-03-25	3	1	7	1.7500000000000000
2012-03-11	1	1	8	1.6000000000000000
2012-02-26	1	0	9	1.5000000000000000
2012-02-05	0	2	9	1.2857142857142857
2012-01-28	2	0	11	1.3750000000000000
2011-12-17	1	0	12	1.3333333333333333
2011-11-25	2	0	14	1.4000000000000000
2011-11-20	2	2	16	1.4545454545454545
2011-10-30	3	1	19	1.5833333333333333
2011-10-15	1	0	20	1.5384615384615385
2011-09-24	1	0	21	1.5000000000000000
2011-09-11	2	3	23	1.5333333333333333
2011-08-20	2	1	25	1.5625000000000000
2011-08-06	0	0	25	1.4705882352941176

SELECT 
	-- Select the date, home goal, and away goals
     date,
    home_goal,
    away_goal,
    -- Create a running total and running average of home goals
    sum(home_goal) over(ORDER BY date DESC
         ROWS BETWEEN current row and unbounded following) AS running_total,
    avg(home_goal) over(ORDER BY date DESC
         ROWS BETWEEN current row and unbounded following) AS running_avg
FROM match
WHERE 
	awayteam_id = 9908 
    AND season = '2011/2012';

date	home_goal	away_goal	running_total	running_avg
2012-05-06	1	3	25	1.4705882352941176
2012-04-21	0	2	24	1.5000000000000000
2012-04-12	3	0	24	1.6000000000000000
2012-03-25	3	1	21	1.5000000000000000
2012-03-11	1	1	18	1.3846153846153846
2012-02-26	1	0	17	1.4166666666666667
2012-02-05	0	2	16	1.4545454545454545
2012-01-28	2	0	16	1.6000000000000000
2011-12-17	1	0	14	1.5555555555555556
2011-11-25	2	0	13	1.6250000000000000
2011-11-20	2	2	11	1.5714285714285714
2011-10-30	3	1	9	1.5000000000000000
2011-10-15	1	0	6	1.2000000000000000
2011-09-24	1	0	5	1.2500000000000000
2011-09-11	2	3	4	1.3333333333333333
2011-08-20	2	1	2	1.00000000000000000000
2011-08-06	0	0	0	0E-20


Setting up the home team CTE
In this course, we've covered ways in which you can use CASE statements, subqueries, common table expressions, 
and window functions in your queries to structure a data set that best meets your needs. For this exercise, you will be using all of 
these concepts to generate a list of matches in which Manchester United was defeated during the 2014/2015 English Premier League season.

Your first task is to create the first query that filters for matches where Manchester United played as the home team. 
This will become a common table expression in a later exercise.

Instructions
100 XP
Create a CASE statement that identifies each match as a win, lose, or tie for Manchester United.
Fill out the logical operators for each WHEN clause in the CASE statement (equals, greater than, less than).
Join the tables on home team ID from match, and team_api_id from team.
Filter the query to only include games from the 2014/2015 season where Manchester United was the home team.

FROM match AS m
-- Left join team on the home team ID and team API id
LEFT JOIN team AS t 
ON m.hometeam_id = t.team_api_id
WHERE 
	-- Filter for 2014/2015 and Manchester United as the home team
	m.season = '2014/2015'
	AND t.team_long_name = 'Manchester United';

id	team_long_name	        outcome
4013	Manchester United	MU Loss
4031	Manchester United	MU Win
4051	Manchester United	MU Win
4062	Manchester United	MU Win
4085	Manchester United	MU Win
4105	Manchester United	MU Win
4145	Manchester United	MU Loss
4164	Manchester United	MU Win
4181	Manchester United	MU Win
4203	Manchester United	MU Win
4225	Manchester United	MU Win
4255	Manchester United	MU Win
4261	Manchester United	MU Win
4294	Manchester United	MU Loss
4311	Manchester United	Tie
4334	Manchester United	MU Win
4354	Manchester United	MU Win
4364	Manchester United	MU Win
4381	Manchester United	Tie

Setting up the away team CTE
Great job! Now that you have a query identifying the home team in a match, you will perform a similar set of steps to identify the away team. 
Just like the previous step, you will join the match and team tables. Each of these two queries will be declared as a Common Table Expression in the following step.

The primary difference in this query is that you will be joining the tables on awayteam_id, and reversing the match outcomes in the CASE statement.

When altering CASE statement logic in your own work, you can reverse either the logical condition (i.e., home_goal > away_goal) 
or the outcome in THEN -- just make sure you only reverse one of the two!

Instructions
Complete the CASE statement syntax.
Fill out the logical operators identifying each match as a win, loss, or tie for Manchester United.
Join the table on awayteam_id, and team_api_id.

SELECT 
	m.id, 
    t.team_long_name,
    -- Identify matches as home/away wins or ties
	case when m.home_goal > m.away_goal then 'MU Loss'
		when m.home_goal < m.away_goal then 'MU Win'
        else 'Tie' end AS outcome
-- Join team table to the match table
FROM match AS m
LEFT JOIN team AS t 
ON m.awayteam_id = t.team_api_id
WHERE 
	-- Filter for 2014/2015 and Manchester United as the away team
	m.season = '2014/2015'
	AND t.team_long_name = 'Manchester United';

id	team_long_name	        outcome
4026	Manchester United	MU Loss
4039	Manchester United	MU Win
4075	Manchester United	MU Win
4089	Manchester United	Tie
4117	Manchester United	Tie
4126	Manchester United	Tie
4136	Manchester United	Tie
4155	Manchester United	MU Win
4178	Manchester United	Tie
4197	Manchester United	MU Loss
4216	Manchester United	MU Win
4230	Manchester United	Tie
4241	Manchester United	MU Win
4271	Manchester United	MU Loss
4282	Manchester United	MU Loss
4302	Manchester United	MU Win
4324	Manchester United	Tie
4342	Manchester United	MU Loss
4378	Manchester United	Tie

Putting the CTEs together
Now that you've created the two subqueries identifying the home and away team opponents, it's time to rearrange your 
query with the home and away subqueries as Common Table Expressions (CTEs). You'll notice that the main query includes the phrase, SELECT DISTINCT. 
Without identifying only DISTINCT matches, you will return a duplicate record for each game played.

Continue building the query to extract all matches played by Manchester United in the 2014/2015 season.

Instructions
Declare the home and away CTEs before your main query.
Join your CTEs to the match table using a LEFT JOIN.
Select the relevant data from the CTEs into the main query.
Select the date from match, team names from the CTEs, and home/ away goals from match in the main query.

-- Set up the home team CTE
with home as (
  SELECT m.id, t.team_long_name,
	  CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
		   WHEN m.home_goal < m.away_goal THEN 'MU Loss' 
  		   ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.hometeam_id = t.team_api_id),
-- Set up the away team CTE
away as (
  SELECT m.id, t.team_long_name,
	  CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
		   WHEN m.home_goal < m.away_goal THEN 'MU Loss' 
  		   ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.awayteam_id = t.team_api_id)
-- Select team names, the date and goals
SELECT DISTINCT
    m.date,
    home.team_long_name AS home_team,
    away.team_long_name AS away_team,
    m.home_goal,
    m.away_goal
-- Join the CTEs onto the match table
FROM match AS m
left JOIN home ON m.id = home.id
left JOIN away ON m.id = away.id
WHERE m.season = '2014/2015'
      AND (home.team_long_name = 'Manchester United' 
           OR away.team_long_name = 'Manchester United');

Add a window function
Fantastic! You now have a result set that retrieves the match date, home team, away team, and 
the goals scored by each team. You have one final component of the question left -- how badly did Manchester United lose in each match?

In order to determine this, let's add a window function to the main query that ranks matches by the absolute value of 
the difference between home_goal and away_goal. This allows us to directly compare the difference in scores without having 
to consider whether Manchester United played as the home or away team!

The equation is complete for you -- all you need to do is properly complete the window function!

Instructions
Set up the CTEs so that the home and away teams each have a name, ID, and score associated with them.
Select the date, home team name, away team name, home goal, and away goals scored in the main query.
Rank the matches and order by the difference in scores in descending order.

-- Set up the home team CTE
with home as (
  SELECT m.id, t.team_long_name,
	  CASE WHEN m.home_goal > m.away_goal THEN 'MU Win'
		   WHEN m.home_goal < m.away_goal THEN 'MU Loss' 
  		   ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.hometeam_id = t.team_api_id),
-- Set up the away team CTE
away as (
  SELECT m.id, t.team_long_name,
	  CASE WHEN m.home_goal > m.away_goal THEN 'MU Loss'
		   WHEN m.home_goal < m.away_goal THEN 'MU Win' 
  		   ELSE 'Tie' END AS outcome
  FROM match AS m
  LEFT JOIN team AS t ON m.awayteam_id = t.team_api_id)
-- Select columns and and rank the matches by date
SELECT DISTINCT
    m.date,
    home.team_long_name AS home_team,
    away.team_long_name AS away_team,
    m.home_goal, m.away_goal,
    rank() over(order by ABS(home_goal - away_goal) desc) as match_rank
-- Join the CTEs onto the match table
FROM match AS m
left JOIN home ON m.id = home.id
left JOIN away ON m.id = away.id
WHERE m.season = '2014/2015'
      AND ((home.team_long_name = 'Manchester United' AND home.outcome = 'MU Loss')
      OR (away.team_long_name = 'Manchester United' AND away.outcome = 'MU Loss'));

date	home_team	away_team	home_goal	away_goal	match_rank
2014-08-16	Manchester United	Swansea City	1	2	3
2014-09-21	Leicester City	   Manchester United	5	3	2
2014-11-02	Manchester City	Manchester United	1	0	3
2015-01-11	Manchester United	Southampton	0	1	3
2015-02-21	Swansea City	Manchester United	2	1	3
2015-04-18	Chelsea		Manchester United	1	0	3
2015-04-26	Everton		Manchester United	3	0	1
2015-05-02	Manchester UnitedWest Bromwich Albion	0	1	3









