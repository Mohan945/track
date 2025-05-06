SELECT s.sid, s.serial#, s.username, t.used_blocks*8/1024 AS MB_USED, t.tablespace
FROM v$sort_usage t, v$session s
WHERE t.session_addr = s.saddr
ORDER BY MB_USED DESC;
