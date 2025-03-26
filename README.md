#!/bin/bash
#
# Advanced Exadata User Security Report – HTML Email Version
# This script gathers user security data from the CI01SYST database,
# produces an HTML-formatted report, and sends it via sendmail.
#

# Define paths
LOG_DIR="/mount/PRODDBA/oracle_scripts/audit/logs"
HTML_REPORT="$LOG_DIR/user_audit_report_$(date '+%Y-%m-%d').html"
EMAIL_RECIPIENT="your_email@example.com"

# Ensure log directory exists
mkdir -p "$LOG_DIR"

# Database details
DB_TNS="CI01SYST"

# Prompt for credentials securely
read -p "Enter DB Username: " DB_USER
read -s -p "Enter Password: " DB_PASS
echo ""

# Create HTML report header
cat <<EOF > "$HTML_REPORT"
<html>
<head>
  <title>🚨 Exadata Security Report - $(date '+%Y-%m-%d')</title>
  <style>
    body { font-family: Arial, sans-serif; }
    table { border-collapse: collapse; width: 100%; margin-bottom: 20px; }
    th, td { border: 1px solid #cccccc; padding: 8px; text-align: left; }
    th { background-color: #f2f2f2; }
  </style>
</head>
<body>
  <h2>🚨 Exadata Security Report - $(date '+%Y-%m-%d')</h2>
  <p><strong>Database:</strong> $DB_TNS</p>
EOF

# Function: run_sql
# Executes a SQL query and appends its results as an HTML table to the report.
run_sql() {
    local title="$1"
    local query="$2"

    echo "<h3>$title</h3>" >> "$HTML_REPORT"
    echo "<table>" >> "$HTML_REPORT"
    echo "<tr><th>USERNAME</th><th>DETAILS</th></tr>" >> "$HTML_REPORT"

    # Execute SQL query via SQL*Plus and capture output in a temporary file
    TMP_FILE="$LOG_DIR/tmp_output.txt"
    rm -f "$TMP_FILE"
    sqlplus -s /nolog <<EOF > "$TMP_FILE" 2>/dev/null
CONNECT $DB_USER/$DB_PASS@$DB_TNS
SET HEAD OFF;
SET FEEDBACK OFF;
SET PAGESIZE 500;
SET LINESIZE 500;
SET TRIMSPOOL ON;
SET COLSEP '|';
$query
EXIT;
EOF

    # Process the temporary file to build HTML table rows.
    while IFS='|' read -r col1 col2; do
      # Only add non-empty rows
      if [ -n "$col1" ]; then
          echo "<tr><td>$col1</td><td>$col2</td></tr>" >> "$HTML_REPORT"
      fi
    done < "$TMP_FILE"
    rm -f "$TMP_FILE"

    echo "</table><br>" >> "$HTML_REPORT"
}

# Run SQL queries for each section

# 1. Locked Users (Today)
run_sql "🔒 Locked Users (Today)" "
    SELECT USERNAME || '|' || TO_CHAR(LOCK_DATE, 'YYYY-MM-DD HH24:MI:SS') || ' - User locked today'
    FROM DBA_USERS
    WHERE ACCOUNT_STATUS = 'LOCKED'
      AND LOCK_DATE >= TRUNC(SYSDATE);
"

# 2. Failed Login Attempts (Last 24 Hours)
run_sql "🚫 Failed Login Attempts (Last 24 Hours)" "
    SELECT USERNAME || '|' || TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') || ' - Failed Login'
    FROM DBA_AUDIT_TRAIL
    WHERE ACTION_NAME = 'LOGON'
      AND TIMESTAMP >= SYSDATE - 1;
"

# 3. Users with Expiring Passwords (Next 7 Days)
run_sql "⏳ Users with Expiring Passwords (Next 7 Days)" "
    SELECT USERNAME || '|' || TO_CHAR(EXPIRY_DATE, 'YYYY-MM-DD') || ' - Password expiring soon'
    FROM DBA_USERS
    WHERE EXPIRY_DATE <= SYSDATE + 7;
"

# 4. Inactive Accounts (No Login in 90+ Days)
run_sql "🛑 Inactive Accounts (No Login in 90+ Days)" "
    SELECT USERNAME || '|' || TO_CHAR(LAST_LOGIN, 'YYYY-MM-DD HH24:MI:SS') || ' - Inactive Account'
    FROM DBA_USERS
    WHERE LAST_LOGIN < SYSDATE - 90;
"

# 5. Users with Excessive Privileges
run_sql "🔑 Users with Excessive Privileges" "
    SELECT GRANTEE || '|' || PRIVILEGE
    FROM DBA_SYS_PRIVS
    WHERE PRIVILEGE IN ('DBA', 'SYSDBA');
"

# Close HTML report
echo "</body></html>" >> "$HTML_REPORT"

# Send the HTML report via sendmail
{
  echo "To: $EMAIL_RECIPIENT"
  echo "Subject: 🚨 Exadata Security Report - $(date '+%Y-%m-%d')"
  echo "Content-Type: text/html"
  echo ""
  cat "$HTML_REPORT"
} | /usr/sbin/sendmail -t

echo "HTML report sent to $EMAIL_RECIPIENT"
