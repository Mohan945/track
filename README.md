#!/bin/bash

# Define paths
LOG_DIR="/mount/PRODDBA/oracle_scripts/audit/logs"
HTML_REPORT="$LOG_DIR/user_audit_report_$(date '+%Y-%m-%d').html"
EMAIL_RECIPIENT="your_email@example.com"

# Ensure log directory exists
mkdir -p "$LOG_DIR"

# Database details (for testing one database)
DB_TNS="CI01SYST"

# Clear previous report
> "$HTML_REPORT"

# Prompt for credentials securely
read -p "Enter DB Username: " DB_USER
read -s -p "Enter Password: " DB_PASS
echo ""

# Write HTML header
cat <<EOF > "$HTML_REPORT"
<html>
<head>
  <style>
    body { font-family: Arial, sans-serif; }
    table { border-collapse: collapse; width: 100%; }
    th, td { border: 1px solid #cccccc; padding: 8px; text-align: left; }
    th { background-color: #f2f2f2; }
  </style>
</head>
<body>
  <h2>🚨 Exadata Security Report - $(date '+%Y-%m-%d')</h2>
  <p><strong>Database:</strong> $DB_TNS</p>
EOF

# Function to execute SQL query and append HTML table rows
run_sql_html() {
    local title="$1"
    local query="$2"

    echo "<h3>$title</h3>" >> "$HTML_REPORT"
    echo "<table>" >> "$HTML_REPORT"
    echo "<tr><th>USERNAME</th><th>TIMESTAMP</th><th>STATUS</th></tr>" >> "$HTML_REPORT"

    # Execute the query and spool output to a temporary file
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

    # Read the temporary file and output each row as an HTML table row.
    while IFS='|' read -r col1 col2 col3; do
      # Only add the row if there's content
      if [ -n "$col1" ]; then
          echo "<tr><td>$col1</td><td>$col2</td><td>$col3</td></tr>" >> "$HTML_REPORT"
      fi
    done < "$TMP_FILE"
    
    echo "</table><br>" >> "$HTML_REPORT"
    rm -f "$TMP_FILE"
}

# Run queries with HTML formatting

run_sql_html "🔒 Locked Users (Today)" \
"SELECT USERNAME || '|' || TO_CHAR(LOCK_DATE, 'YYYY-MM-DD HH24:MI:SS') || '|' || 'User locked today'
 FROM DBA_USERS 
 WHERE ACCOUNT_STATUS LIKE 'LOCKED%' 
   AND LOCK_DATE >= TRUNC(SYSDATE);"

run_sql_html "🚫 Failed Login Attempts (Last 24 Hours)" \
"SELECT USERNAME || '|' || TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') || '|' || 'Failed Login'
 FROM DBA_AUDIT_SESSION 
 WHERE ACTION_NAME = 'LOGON'
   AND TIMESTAMP >= SYSDATE - 1;"

run_sql_html "⏳ Users with Expiring Passwords (Next 7 Days)" \
"SELECT USERNAME || '|' || TO_CHAR(EXPIRY_DATE, 'YYYY-MM-DD') || '|' || 'Expiring Soon'
 FROM DBA_USERS 
 WHERE EXPIRY_DATE <= SYSDATE + 7;"

run_sql_html "🛑 Inactive Accounts (No Login in 90+ Days)" \
"SELECT USERNAME || '|' || TO_CHAR(LAST_LOGIN, 'YYYY-MM-DD HH24:MI:SS') || '|' || 'Inactive Account'
 FROM DBA_USERS 
 WHERE LAST_LOGIN < SYSDATE - 90;"

run_sql_html "🔑 Users with Excessive Privileges" \
"SELECT GRANTEE || '|' || 'N/A' || '|' || PRIVILEGE
 FROM DBA_SYS_PRIVS 
 WHERE PRIVILEGE IN ('DBA', 'SYSDBA');"

# Write HTML footer and close document
echo "</body></html>" >> "$HTML_REPORT"

# Send the HTML email (the -a option adds a header for content-type)
mail -a "Content-Type: text/html" -s "🚨 Exadata Security Report - $(date '+%Y-%m-%d')" "$EMAIL_RECIPIENT" < "$HTML_REPORT"

echo "HTML report sent to $EMAIL_RECIPIENT"
