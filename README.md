SELECT
    owner AS schema_name,
    ROUND(SUM(bytes)/1024/1024/1024, 2) AS size_gb
FROM
    dba_segments
WHERE
    owner = UPPER('&SCHEMA_NAME')
GROUP BY
    owner;
