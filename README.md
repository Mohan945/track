#!/bin/bash

# Define file paths
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/logs/locked_users_report_$(date '+%Y-%m-%d').log"
EMAIL_RECIPIENT="your_email@example.com"

# List of databases (TNS entries)
DATABASES=("DB1" "DB2" "DB3")  # Replace with actual TNS names

# Clear log file
> "$LOG_FILE"

# Start logging
echo "Locked Users Report - $(date)" | tee -a "$LOG_FILE"
echo "------------------------------------" | tee -a "$LOG_FILE"

# Loop through each database and check locked users
for DB in "${DATABASES[@]}"; do
    echo "Checking database: $DB" | tee -a "$LOG_FILE"
    
    sqlplus -s /nolog <<EOF >> "$LOG_FILE" 2>&1
    CONNECT system/your_password@$DB
    SET HEADING OFF;
    SET FEEDBACK OFF;
    SELECT '$DB' AS DATABASE_NAME, USERNAME 
    FROM DBA_USERS 
    WHERE ACCOUNT_STATUS = 'LOCKED'
    AND USERNAME IN (SELECT USERNAME FROM DBA_AUDIT_TRAIL WHERE ACTION_NAME = 'ALTER USER' AND TIMESTAMP >= SYSDATE - 1);
    EXIT;
EOF

    echo "------------------------------------" | tee -a "$LOG_FILE"
done

# Send the report via email
mail -s "Daily Locked Users Report - $(date '+%Y-%m-%d')" "$EMAIL_RECIPIENT" < "$LOG_FILE"
