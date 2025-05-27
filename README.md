SELECT username,
       MAX(timestamp) AS last_login
FROM dba_audit_session
WHERE username = 'USERNAME_IN_UPPERCASE'
  AND returncode = 0
GROUP BY username;
