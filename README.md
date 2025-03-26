#!/bin/bash

# Define paths
LOG_DIR="/mount/PRODDBA/oracle_scripts/audit/logs"
TEXT_REPORT="$LOG_DIR/user_audit_report_$(date '+%Y-%m-%d').txt"
EMAIL_RECIPIENT="your_email@example.com"

# Ensure log directory exists
mkdir -p "$LOG_DIR"

# Database details
DB_TNS="CI01SYST"  # Single database for testing

# Clear previous report
> "$TEXT_REPORT"

# Prompt for credentials securely
read -p "Enter DB Username: " DB_USER
read -s -p "Enter Password: " DB_PASS
echo ""

# Add report header
echo "🚨 **Exadata Security Report - $(date '+%Y-%m-%d')**" >> "$TEXT_REPORT"
echo "Database: **$DB_TNS**" >> "$TEXT_REPORT"
echo "" >> "$TEXT_REPORT"

# Function to execute SQL query and append results in table format
run_sql() {
    local title=$1
    local query=$2
    local header=$3

    echo "### $title" >> "$TEXT_REPORT"
    echo "+----------------------+---------------------+--------------------+" >> "$TEXT_REPORT"
    echo "| $header |" >> "$TEXT_REPORT"
    echo "+----------------------+---------------------+--------------------+" >> "$TEXT_REPORT"

    sqlplus -s /nolog <<EOF | awk -F'|' '{printf "| %-20s | %-20s | %-20s |\n", $1, $2, $3}' >> "$TEXT_REPORT"
    CONNECT $DB_USER/$DB_PASS@$DB_TNS
    SET HEADING OFF;
    SET PAGESIZE 500;
    SET LINESIZE 500;
    SET FEEDBACK OFF;

    $query

    EXIT;
EOF

    echo "+----------------------+---------------------+--------------------+" >> "$TEXT_REPORT"
    echo "" >> "$TEXT_REPORT"
}

# Run queries with table format
run_sql "🔒 Locked Users (Today)" \
    "SELECT USERNAME || '|' || TO_CHAR(LOCK_DATE, 'YYYY-MM-DD HH24:MI:SS') || '|' || 'User locked today' 
     FROM DBA_USERS 
     WHERE ACCOUNT_STATUS LIKE 'LOCKED%' 
     AND LOCK_DATE >= TRUNC(SYSDATE);" \
    "USERNAME | LOCK DATE | STATUS"

run_sql "🚫 Failed Login Attempts (Last 24 Hours)" \
    "SELECT USERNAME || '|' || TO_CHAR(TIMESTAMP, 'YYYY-MM-DD HH24:MI:SS') || '|' || 'Failed Login' 
     FROM DBA_AUDIT_SESSION 
     WHERE ACTION_NAME = 'LOGON' 
     AND RETURN_CODE != 0 
     AND TIMESTAMP >= SYSDATE - 1;" \
    "USERNAME | TIMESTAMP | STATUS"

run_sql "⏳ Users with Expiring Passwords (Next 7 Days)" \
    "SELECT USERNAME || '|' || TO_CHAR(EXPIRY_DATE, 'YYYY-MM-DD') || '|' || 'Expiring Soon' 
     FROM DBA_USERS 
     WHERE EXPIRY_DATE <= SYSDATE + 7;" \
    "USERNAME | EXPIRY DATE | STATUS"

run_sql "🛑 Inactive Accounts (No Login in 90+ Days)" \
    "SELECT USERNAME || '|' || TO_CHAR(LAST_LOGIN, 'YYYY-MM-DD HH24:MI:SS') || '|' || 'Inactive Account' 
     FROM DBA_USERS 
     WHERE LAST_LOGIN < SYSDATE - 90;" \
    "USERNAME | LAST LOGIN | STATUS"

run_sql "🔑 Users with Excessive Privileges" \
    "SELECT GRANTEE || '|' || 'N/A' || '|' || PRIVILEGE 
     FROM DBA_SYS_PRIVS 
     WHERE PRIVILEGE IN ('DBA', 'SYSDBA');" \
    "USERNAME | PRIVILEGE"

# Send report via email
mail -s "🚨 Exadata Security Report - $(date '+%Y-%m-%d')" "$EMAIL_RECIPIENT" < "$TEXT_REPORT"

echo "Report sent to $EMAIL_RECIPIENT"
