SELECT username, action_name, returncode, to_char(timestamp, 'DD-MON-YYYY HH24:MI:SS') AS login_time,
       to_char(logoff_time, 'DD-MON-YYYY HH24:MI:SS') AS logout_time,
       (logoff_time - timestamp) * 1440 AS session_duration_mins
FROM dba_audit_session
WHERE username = 'SLBR73'
ORDER BY timestamp DESC;
