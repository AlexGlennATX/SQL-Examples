# SQL-Examples
Examples of my work in SQL for professional projects

## Wholesale customer session data
*Get an idea of how well our dealers have adopted the wholesale customer portal, frequency of use, how widespread adoption is etc
*Sessoinize data for the first time, to export to Sheets for secondary users

## Sessionize data for export
WITH sessions AS (
  SELECT
	user_id,
	timestamp,
	CASE
  	WHEN EXTRACT(EPOCH FROM (timestamp - LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp))) > 900
  	OR LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) IS NULL
  	THEN 1
  	ELSE 0
	END AS new_session
  FROM "Shopify"."wholesale_customers"
)
SELECT
  *,
  SUM(new_session) OVER (PARTITION BY user_id ORDER BY timestamp) as session_id
FROM sessions
ORDER BY timestamp, user_id;


## Sessionize, count, visuzlize
WITH sessions AS (
  SELECT
	user_id,
	timestamp,
	CASE
  	WHEN EXTRACT(EPOCH FROM (timestamp - LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp))) > 900
  	OR LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) IS NULL
  	THEN 1
  	ELSE 0
	END AS new_session
  FROM "shopify"."wholesale_customers"
),
daily_sessions AS (
  SELECT
	*,
	SUM(new_session) OVER (PARTITION BY user_id ORDER BY timestamp) as session_id
  FROM sessions
)
SELECT
  date_trunc('day', timestamp) as day,
  COUNT(DISTINCT user_id || '_' || session_id) as number_of_sessions
FROM daily_sessions
GROUP BY date_trunc('day', timestamp)
ORDER BY day;


