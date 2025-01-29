# SQL-Examples
Examples of my work in SQL for professional projects

## Wholesale customer session data
*Get an idea of how well our dealers have adopted the wholesale customer portal, frequency of use, how widespread adoption is etc
*Sessoinize data for the first time, to export to Sheets for secondary users

## Sessionize data for export
WITH sessions<br> AS (
  SELECT
	user_id,
	timestamp,
	<br>CASE
  	WHEN EXTRACT(EPOCH FROM (timestamp - LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp))) > 900
  	OR LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) IS NULL
  	<br>THEN 1
  	<br>ELSE 0
	<br>END AS new_session
  <br>FROM "Shopify"."wholesale_customers"
)
<br>SELECT
  *,
  <br>SUM(new_session) OVER (PARTITION BY user_id ORDER BY timestamp) as session_id
<br>FROM sessions
<br>ORDER BY timestamp, user_id;


## Sessionize, count, visuzlize
WITH sessions AS (
  <br>SELECT
	user_id,
	timestamp,
	<br>CASE
  	WHEN EXTRACT(EPOCH FROM (timestamp - LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp))) > 900
  	OR LAG(timestamp) OVER (PARTITION BY user_id ORDER BY timestamp) IS NULL
  	<br>THEN 1
  	<br>ELSE 0
	<br>END AS new_session
  <br>FROM "shopify"."wholesale_customers"
),
<br>daily_sessions AS (
  <br>SELECT
	*,
	<br>SUM(new_session) OVER (PARTITION BY user_id ORDER BY timestamp) as session_id
  <br>FROM sessions
)
<br>SELECT
  date_trunc('day', timestamp) as day,
  COUNT(DISTINCT user_id || '_' || session_id) as number_of_sessions
<br>FROM daily_sessions
<br>GROUP BY date_trunc('day', timestamp)
<br>ORDER BY day;


