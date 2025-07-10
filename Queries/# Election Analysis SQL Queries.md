UK General Election Analysis 

Database: PostgreSQL
Data Source: election table (Raw Data)
Last Updated: 10/07/2025


### 1. Summary Statistics
### Constituency Wins by Party

### QUERY: 

SELECT 
    winningparty AS "Winning Party", 
    COUNT(*) AS "Total Constituencies",
    ROUND(COUNT(*) * 100.0 / (SELECT COUNT(*) FROM election), 1) AS "Percentage"
FROM election
GROUP BY winningparty
ORDER BY COUNT(*) DESC;



### 2. Brexit Correlation Analysis
### Party Performance by Referendum Vote
### Query


SELECT 
    winningparty AS party,
	COUNT(winningparty) AS "Total wins",
    SUM(CASE WHEN leavemajority > 0 THEN 1 ELSE 0 END) AS "Leave Wins",
    SUM(CASE WHEN leavemajority < 0 THEN 1 ELSE 0 END) AS "Remain Wins",
    ROUND(
        100.0 * 
        SUM(CASE WHEN leavemajority < 0 THEN 1 ELSE 0 END) / 
        COUNT(*),
        1
    ) AS "Remain Percentage",
    ROUND(
        100.0 * 
        SUM(CASE WHEN leavemajority < 0 THEN 1 ELSE 0 END) /
        SUM(SUM(CASE WHEN leavemajority < 0 THEN 1 ELSE 0 END)) OVER (),
        1
    ) AS "Remain wins / Total Remain Wins (%)",
	  ROUND(
        100.0 * 
        SUM(CASE WHEN leavemajority > 0 THEN 1 ELSE 0 END) /
        SUM(SUM(CASE WHEN leavemajority > 0 THEN 1 ELSE 0 END)) OVER (),
        1
    ) AS "Leave wins / Total Leave Wins (%)"
	
FROM election
WHERE winningparty IS NOT NULL
GROUP BY winningparty
ORDER BY "Total wins" DESC


SELECT 
    region,
    COUNT(CASE WHEN "2017_winner" = 'Conservative' THEN 1 END) AS con_wins,
    ROUND((COUNT(CASE WHEN "2017_winner" = 'Conservative' THEN 1 END) * 100.0 / COUNT(*)), 1) AS "Con wins %",
    COUNT(CASE WHEN "2017_winner" = 'Labour' THEN 1 END) AS labour_wins,
    ROUND((COUNT(CASE WHEN "2017_winner" = 'Labour' THEN 1 END) * 100.0 / COUNT(*)), 1) AS "Lab wins %",
    COUNT(CASE WHEN "2017_winner" NOT IN ('Conservative', 'Labour') THEN 1 END) AS other_wins,
    ROUND((COUNT(CASE WHEN "2017_winner" NOT IN ('Conservative', 'Labour') THEN 1 END) * 100.0 / COUNT(*)), 1) AS "Other wins %",
    COUNT(*) AS total_constituencies
FROM election
GROUP BY region
ORDER BY region;


SELECT 
    region,
    ROUND(AVG(leavemajority), 2) AS avg_leave_margin,
    COUNT(CASE WHEN leavemajority > 0 THEN 1 END) AS leave_constituencies,
    COUNT(CASE WHEN leavemajority < 0 THEN 1 END) AS remain_constituencies
FROM election
GROUP BY region
ORDER BY avg_leave_margin DESC;




### 3. Victory Margins Analysis
### Top 5 Majority Victories by Party
### Conservative
### Query

WITH con_top_majorities AS (
    SELECT 
        constituency, 
        region,
        winningvotes - secondplacevotes AS majority,
        RANK() OVER (ORDER BY (winningvotes - secondplacevotes) DESC) AS rank
    FROM election
    WHERE winningparty = 'Conservative'
)
SELECT 
    rank,
    constituency,
    region,
    majority
FROM con_top_majorities
WHERE rank <= 5;


WITH lab_top_majorities AS (
    SELECT 
        constituency, 
        region,
        winningvotes - secondplacevotes AS majority,
        RANK() OVER (ORDER BY (winningvotes - secondplacevotes) DESC) AS rank
    FROM election
    WHERE winningparty = 'Labour'
)
SELECT 
    rank,
    constituency,
    region,
    majority
FROM lab_top_majorities
WHERE rank <= 5;



### 4. Closest Marginal Seats
### Ranking the top 10 Constituencies by smallest majority
### Query

SELECT 
    constituency,
    region,
    winningparty,
    winningvotes - secondplacevotes AS majority,
    RANK() OVER (ORDER BY ABS(winningvotes - secondplacevotes)) AS marginal_rank
FROM election
ORDER BY marginal_rank
LIMIT 10;


