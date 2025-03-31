#!/bin/ksh

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"
DB_LIST_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/db_list.txt"

# Hardcoded SYSTEM credentials (Use with caution!)
DB_USER="SYSTEM"
DB_PASS="YourSecurePassword"  # Replace with actual password

# Clear previous log file
> "$LOG_FILE"

# Validate the database list file
if [ ! -s "$DB_LIST_FILE" ]; then
    echo "[$(date)] Database list file is missing or empty. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Loop through each database from the file
while read DB_NAME; do
    echo "Processing database: $DB_NAME" | tee -a "$LOG_FILE"

    # Build the SQL script dynamically
    > "$SQL_SCRIPT"
    echo "SET SERVEROUTPUT ON;" >> "$SQL_SCRIPT"
    echo "SPOOL $LOG_FILE APPEND;" >> "$SQL_SCRIPT"
    echo "DECLARE" >> "$SQL_SCRIPT"
    echo "  v_user_found NUMBER := 0;" >> "$SQL_SCRIPT"
    echo "BEGIN" >> "$SQL_SCRIPT"

    while read EMP_ID; do
        echo "  FOR user_rec IN (SELECT username FROM dba_users WHERE username LIKE '%${EMP_ID}%') LOOP" >> "$SQL_SCRIPT"
        echo "    DBMS_OUTPUT.PUT_LINE('Locking user: ' || user_rec.username);" >> "$SQL_SCRIPT"
        echo "    EXECUTE IMMEDIATE 'ALTER USER ' || user_rec.username || ' ACCOUNT LOCK';" >> "$SQL_SCRIPT"
        echo "    v_user_found := 1;" >> "$SQL_SCRIPT"
        echo "  END LOOP;" >> "$SQL_SCRIPT"
    done < "$SQL_INPUT_FILE"

    echo "  IF v_user_found = 0 THEN" >> "$SQL_SCRIPT"
    echo "    DBMS_OUTPUT.PUT_LINE('No matching users found to lock in $DB_NAME.');" >> "$SQL_SCRIPT"
    echo "  END IF;" >> "$SQL_SCRIPT"
    echo "END;" >> "$SQL_SCRIPT"
    echo "/" >> "$SQL_SCRIPT"
    echo "SPOOL OFF;" >> "$SQL_SCRIPT"

    # Execute SQL script with hardcoded credentials
    sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"

    # Check for errors
    if [ $? -eq 0 ]; then
        echo "[$(date)] User locking process completed successfully for $DB_NAME." | tee -a "$LOG_FILE"
    else
        echo "[$(date)] Error occurred while processing $DB_NAME." | tee -a "$LOG_FILE"
    fi

done < "$DB_LIST_FILE"

# Cleanup
echo "Script execution completed." | tee -a "$LOG_FILE"
