#!/bin/ksh
####################################
# Locking Process with Dynamic Env #
# SCRIPT : Leavers_Automated
# VERSION : 002
# AUTHOR : Mohan T (CO70989)
# DATE : 2nd April 2025
# PURPOSE : Locks users based on leavers/movers list
####################################

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/tnsnames.ora"
DB_ENV_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/all_inst_nonDataGuard_Y"
EMAIL_REPORT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/locked_users_report.txt"

# Database credentials (DO NOT DISPLAY IN OUTPUT)
DB_USER="DBSNMP"
DB_PASS="c0mplacent"

# List of databases to process
DB_NAMES=("CI01PROD" "MI22PROD" "EB21PROD" "WT23PROD")

# Clear previous log files
> "$LOG_FILE"
> "$EMAIL_REPORT"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if Employee IDs exist
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Prepare email table header
echo "==========================================" >> "$EMAIL_REPORT"
echo "         Locked Users Report              " >> "$EMAIL_REPORT"
echo "==========================================" >> "$EMAIL_REPORT"
echo "Database      |  User ID                  " >> "$EMAIL_REPORT"
echo "------------------------------------------" >> "$EMAIL_REPORT"

# Process each database
for DB_NAME in "${DB_NAMES[@]}"; do
    echo "[$(date)] Processing database: $DB_NAME" | tee -a "$LOG_FILE"

    # Check if the database exists in the TNS file
    if ! grep -q -w "$DB_NAME" "$TNS_FILE"; then
        echo "[$(date)] Database $DB_NAME not found in TNS file. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    # Extract database prefix
    CONN_STR=$(echo "$DB_NAME" | cut -c 1-4)

    # Extract ORACLE_HOME from environment file
    ORACLE_HOME=$(awk -F: -v db="$CONN_STR" '$1 == db {print $2}' "$DB_ENV_FILE")

    # Debugging logs
    echo "[$(date)] Extracted CONN_STR: $CONN_STR" | tee -a "$LOG_FILE"
    echo "[$(date)] Extracted ORACLE_HOME: $ORACLE_HOME" | tee -a "$LOG_FILE"

    # Validate ORACLE_HOME
    if [ -z "$ORACLE_HOME" ] || [ ! -d "$ORACLE_HOME" ]; then
        echo "[$(date)] No valid ORACLE_HOME found for $DB_NAME. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    export ORACLE_HOME
    export PATH="$ORACLE_HOME/bin:$PATH"

    # Verify sqlplus exists
    if ! command -v sqlplus > /dev/null; then
        echo "[$(date)] sqlplus not found in $ORACLE_HOME. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    # Build the SQL script
    > "$SQL_SCRIPT"
    echo "SET SERVEROUTPUT ON;" >> "$SQL_SCRIPT"
    echo "SPOOL $LOG_FILE APPEND;" >> "$SQL_SCRIPT"
    echo "DECLARE" >> "$SQL_SCRIPT"
    echo "  v_user_found NUMBER := 0;" >> "$SQL_SCRIPT"
    echo "BEGIN" >> "$SQL_SCRIPT"

    while read EMP_ID; do
        if [ -n "$EMP_ID" ]; then
            echo "  FOR user_rec IN (SELECT username FROM dba_users WHERE username LIKE '%${EMP_ID}%') LOOP" >> "$SQL_SCRIPT"
            echo "    DBMS_OUTPUT.PUT_LINE('LOCKED:' || '$DB_NAME|' || user_rec.username);" >> "$SQL_SCRIPT"
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

    # Execute SQL script and capture locked users
    LOCKED_USERS=$(sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" @"$SQL_SCRIPT" | grep "LOCKED:" | awk -F'|' '{printf "%-13s | %-20s\n", $2, $3}')

    if [ -n "$LOCKED_USERS" ]; then
        echo "$LOCKED_USERS" >> "$EMAIL_REPORT"
    fi

    SQL_EXIT_CODE=${PIPESTATUS[0]}

    if [ "$SQL_EXIT_CODE" -eq 0 ] || [ "$SQL_EXIT_CODE" -eq 2 ]; then
        echo "[$(date)] Successfully processed database: $DB_NAME" | tee -a "$LOG_FILE"
    else
        echo "[$(date)] Error processing database: $DB_NAME (Exit Code: $SQL_EXIT_CODE)" | tee -a "$LOG_FILE"
    fi

    echo "" | tee -a "$LOG_FILE"
done

# Close email table
echo "==========================================" >> "$EMAIL_REPORT"

# Send email
mailx -s "Locked Users Report - $(date +'%Y-%m-%d')" recipient@example.com < "$EMAIL_REPORT"

echo "[$(date)] Locking process completed and email sent." | tee -a "$LOG_FILE"
