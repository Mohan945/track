#!/bin/bash

# Define file paths (NFS location for shared access)
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"

# Database credentials
DB_USER="your_db_user"
DB_PASS="your_db_password"
DB_NAME="your_test_db"

# Clear log file
> "$LOG_FILE"

# Extract Employee IDs
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $3}' > "$SQL_INPUT_FILE"

# Check if Employee IDs are found
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Execute SQL script
{
    echo "SET SERVEROUTPUT ON;"
    echo "SPOOL $LOG_FILE;"
    echo "BEGIN"
    echo "  FOR emp IN (SELECT column_value AS emp_id FROM TABLE(sys.dbms_debug_vc2coll("
    awk '{print "  ''" $1 "''" ","}' "$SQL_INPUT_FILE" | sed '$ s/,$//'
    echo "  ))) LOOP"
    echo "    FOR user_rec IN (SELECT username FROM dba_users WHERE username LIKE '%' || emp.emp_id || '%') LOOP"
    echo "      EXECUTE IMMEDIATE 'ALTER USER ' || user_rec.username || ' ACCOUNT LOCK';"
    echo "      DBMS_OUTPUT.PUT_LINE('User locked: ' || user_rec.username);"
    echo "    END LOOP;"
    echo "  END LOOP;"
    echo "END;"
    echo "/"
    echo "SPOOL OFF;"
} | sqlplus -s "$DB_USER/$DB_PASS@${DB_NAME}" >> "$LOG_FILE" 2>&1

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully for $DB_NAME." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors. Check log for details." | tee -a "$LOG_FILE"
fi
