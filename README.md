#!/bin/ksh
####################################
# Locking Users - EB21PROD Only    #
# SCRIPT : Leavers_Automated_EB21  #
# VERSION : 002
# AUTHOR : Mohan T (CO70989)
# DATE : 2nd April 2025
# PURPOSE : Lock users in EB21PROD DB #
####################################

# Define Paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users_EB21.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_EB21.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/tnsnames.ora"
DB_ENV_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/all_inst_nonDataGuard_Y"

# Database credentials
DB_USER="DBSNMP"
DB_PASS="c0mplacent"
DB_NAME="EB21PROD"  # EB21 Specific

# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if Employee IDs exist
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

echo "[$(date)] Processing database: $DB_NAME" | tee -a "$LOG_FILE"

#############################################
# Dynamic ORACLE_HOME Selection for EB21PROD
#############################################

# Extract ORACLE_HOME from environment file
ORACLE_HOME=$(awk -F: -v db="EB21" '$1 == db {print $2}' "$DB_ENV_FILE")

# Debugging logs
echo "[$(date)] Extracted ORACLE_HOME: $ORACLE_HOME" | tee -a "$LOG_FILE"

# Validate ORACLE_HOME
if [ -z "$ORACLE_HOME" ] || [ ! -d "$ORACLE_HOME" ]; then
    echo "[$(date)] No valid ORACLE_HOME found for $DB_NAME. Skipping." | tee -a "$LOG_FILE"
    exit 1
fi

export ORACLE_HOME
export PATH="$ORACLE_HOME/bin:$PATH"
export LD_LIBRARY_PATH="$ORACLE_HOME/lib:$LD_LIBRARY_PATH"

echo "[$(date)] Set ORACLE_HOME: $ORACLE_HOME" | tee -a "$LOG_FILE"

# Set ORACLE_SID dynamically (for RAC compatibility)
export ORACLE_SID=$(ps -ef | grep pmon | grep -i "EB21" | awk -F_ '{print $3}')

# Check sqlplus
if ! command -v sqlplus > /dev/null; then
    echo "[$(date)] sqlplus not found in $ORACLE_HOME. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Set correct TNS_ADMIN path
export TNS_ADMIN="/mount/PRODDBA/oracle_scripts/recert/leavers/test"

# Test connection with tnsping
echo "[$(date)] Testing connection with tnsping..." | tee -a "$LOG_FILE"
tnsping EB21PROD | tee -a "$LOG_FILE"

#############################################
# Build the SQL script dynamically
#############################################
> "$SQL_SCRIPT"
echo "SET SERVEROUTPUT ON;" >> "$SQL_SCRIPT"
echo "SPOOL $LOG_FILE APPEND;" >> "$SQL_SCRIPT"
echo "DECLARE" >> "$SQL_SCRIPT"
echo "  v_user_found NUMBER := 0;" >> "$SQL_SCRIPT"
echo "BEGIN" >> "$SQL_SCRIPT"

while read EMP_ID; do
    if [ -n "$EMP_ID" ]; then
        echo "  FOR user_rec IN (SELECT username FROM dba_users WHERE username LIKE '%${EMP_ID}%') LOOP" >> "$SQL_SCRIPT"
        echo "    DBMS_OUTPUT.PUT_LINE('Locking user: ' || user_rec.username);" >> "$SQL_SCRIPT"
        echo "    EXECUTE IMMEDIATE 'ALTER USER ' || user_rec.username || ' ACCOUNT LOCK';" >> "$SQL_SCRIPT"
        echo "    v_user_found := 1;" >> "$SQL_SCRIPT"
        echo "  END LOOP;" >> "$SQL_SCRIPT"
    fi
done < "$SQL_INPUT_FILE"

echo "  IF v_user_found = 0 THEN" >> "$SQL_SCRIPT"
echo "    DBMS_OUTPUT.PUT_LINE('No matching users found to lock.');" >> "$SQL_SCRIPT"
echo "  END IF;" >> "$SQL_SCRIPT"
echo "END;" >> "$SQL_SCRIPT"
echo "/" >> "$SQL_SCRIPT"
echo "SPOOL OFF;" >> "$SQL_SCRIPT"
echo "EXIT;" >> "$SQL_SCRIPT"

#############################################
# Execute SQL script (Using Multiple Methods)
#############################################

# Option 1: Using TNS Name (Standard)
echo "[$(date)] Connecting using TNS alias..." | tee -a "$LOG_FILE"
sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"
SQL_EXIT_CODE=${PIPESTATUS[0]}

# Option 2: Using Full Connection String (If Option 1 Fails)
if [ "$SQL_EXIT_CODE" -ne 0 ]; then
    echo "[$(date)] TNS alias failed. Trying full connection string..." | tee -a "$LOG_FILE"
    sqlplus -s "$DB_USER/$DB_PASS@\"(DESCRIPTION=(ADDRESS_LIST=(ADDRESS=(PROTOCOL=TCP)(HOST=rac-scan.example.com)(PORT=1521)))(CONNECT_DATA=(SERVICE_NAME=EB21PROD)))\"" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"
    SQL_EXIT_CODE=${PIPESTATUS[0]}
fi

# Option 3: Using EZCONNECT (If Both Fail)
if [ "$SQL_EXIT_CODE" -ne 0 ]; then
    echo "[$(date)] Full connection string failed. Trying EZCONNECT..." | tee -a "$LOG_FILE"
    sqlplus -s "$DB_USER/$DB_PASS@rac-scan.example.com:1521/EB21PROD" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"
    SQL_EXIT_CODE=${PIPESTATUS[0]}
fi

# Final Check
if [ "$SQL_EXIT_CODE" -eq 0 ] || [ "$SQL_EXIT_CODE" -eq 2 ]; then
    echo "[$(date)] Successfully processed database: $DB_NAME" | tee -a "$LOG_FILE"
else
    echo "[$(date)] Error processing database: $DB_NAME (Exit Code: $SQL_EXIT_CODE)" | tee -a "$LOG_FILE"
fi

echo "[$(date)] Locking process completed for $DB_NAME." | tee -a "$LOG_FILE"
exit 0
