SELECT s.sid, s.serial#, s.username, tu.tablespace,
       tu.segtype, tu.contents,
       ROUND(tu.blocks * 8 / 1024, 2) AS mb_used
FROM v$tempseg_usage tu
JOIN v$session s ON tu.session_addr = s.saddr
ORDER BY mb_used DESC;
