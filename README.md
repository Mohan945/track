SELECT
  ROUND(SUM(bytes) / 1024 / 1024) AS total_sga_mb,
  ROUND(SUM(CASE WHEN name = 'free memory' THEN bytes ELSE 0 END) / 1024 / 1024) AS free_sga_mb,
  ROUND(
    100 * SUM(CASE WHEN name = 'free memory' THEN bytes ELSE 0 END) / SUM(bytes),
    2
  ) AS free_percent
FROM
  v$sgastat;
