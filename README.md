SELECT username,
       timestamp,
       returncode,
       os_username,
       userhost
FROM dba_audit_session
WHERE returncode != 0
ORDER BY timestamp DESC;
