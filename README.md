#!/bin/bash

# Define paths
LOG_DIR="/mount/PRODDBA/oracle_scripts/audit/logs"
HTML_REPORT="$LOG_DIR/user_audit_report_$(date '+%Y-%m-%d').html"
EMAIL_RECIPIENT="your_email@example.com"

# Ensure log directory exists
mkdir -p "$LOG_DIR"

# Database details
DB_TNS="CI01SYST"  # Modify for multiple DBs if needed

# Prompt for credentials securely
read -p "Enter DB Username: " DB_USER
read -s -p "Enter Password: " DB_PASS
echo ""

# Start HTML report
echo "<html><body>" > "$HTML_REPORT"
echo "<h2>🚨 Exadata Security Report - $(date '+%Y-%m-%d')</h2>" >> "$HTML_REPORT"
echo "<p><strong>Database:</strong> $DB_TNS</p>" >> "$HTML_REPORT"

# Function to execute SQL and format output as HTML
run_sql() {
    local title="$1"
    local query="$2"

    echo "<h3>$title</h3>" >> "$HTML_REPORT"
    echo "<table border='1' cellpadding='5' cellspacing='0'>" >> "$HTML_REPORT"
    echo "<tr style='background-color:#f2f2f2;'><th>USERNAME</th><th>DETAILS</th></tr>" >> "$HTML_REPORT"

    sqlplus -s /nolog <<EOF | awk 'NR>1 {print "<tr><td>" $1 "</td><td>" $2 "</td></tr>"}' >> "$HTML_REPORT"
    CONNECT $DB_USER/$DB_PASS@$DB_TNS
    SET HEAD OFF PAGESIZE 500 LINESIZE 500 FEEDBACK OFF;
    $query
    EXIT;
EOF

    echo "</table><br>" >> "$HTML_REPORT"
}

# Queries with proper SQL logic
run_sql "🔒 Locked Users (Today)" "
    SELECT USERNAME, 'User locked today'
    FROM DBA_USERS 
    WHERE ACCOUNT_STATUS = 'LOCKED' 
    AND LOCK_DATE >= TRUNC(SYSDATE);
"

run_sql "🚫 Failed Login Attempts (Last 24 Hours)" "
    SELECT USERNAME, 'Failed Login Attempt'
    FROM DBA_AUDIT_SESSION
    WHERE ACTION_NAME = 'LOGON'
    AND TIMESTAMP >= SYSDATE - 1;
"

run_sql "⏳ Users with Expiring Passwords (Next 7 Days)" "
    SELECT USERNAME, 'Password Expiring Soon'
    FROM DBA_USERS 
    WHERE EXPIRY_DATE <= SYSDATE + 7;
"

run_sql "🛑 Inactive Accounts (No Login in 90+ Days)" "
    SELECT USERNAME, 'Inactive Account'
    FROM DBA_USERS 
    WHERE LAST_LOGIN < SYSDATE - 90;
"

run_sql "🔑 Users with Excessive Privileges" "
    SELECT GRANTEE, PRIVILEGE
    FROM DBA_SYS_PRIVS 
    WHERE PRIVILEGE IN ('DBA', 'SYSDBA');
"

# End HTML report
echo "</body></html>" >> "$HTML_REPORT"

# Send email with HTML format
mail -a "Content-Type: text/html" -s "🚨 Exadata Security Report - $(date '+%Y-%m-%d')" "$EMAIL_RECIPIENT" < "$HTML_REPORT"

echo "Report sent to $EMAIL_RECIPIENT"
