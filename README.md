SELECT s.username, s.status, s.machine, s.program, s.module, q.sql_text
FROM v$session s
LEFT JOIN v$sql q ON s.sql_id = q.sql_id
WHERE s.username = 'SLBR73';

SELECT username, returncode, timestamp
FROM dba_audit_trail
WHERE username = 'SLBR73'
ORDER BY timestamp DESC;
