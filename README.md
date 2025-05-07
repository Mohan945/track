SET PAGESIZE 1000
SET LINESIZE 200
COLUMN sql_text FORMAT A100
COLUMN sharable_mem FORMAT 999,999,999

SELECT
  sharable_mem,
  executions,
  parse_calls,
  loads,
  sql_id,
  SUBSTR(sql_text, 1, 100) AS sql_text
FROM
  v$sqlarea
WHERE
  sharable_mem > 1048576
ORDER BY
  sharable_mem DESC;
