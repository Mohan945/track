#!/bin/ksh

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/tmp/lock_users.sql"

# Database credentials
DB_USER="your_db_user"
DB_PASS="your_db_password"
DB_NAME="your_test_db"

# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs for accounts that need to be locked
egrep "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $3}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Build the SQL script dynamically
> "$SQL_SCRIPT"

echo "SET SERVEROUTPUT ON;" >> "$SQL_SCRIPT"
echo "SPOOL $LOG_FILE;" >> "$SQL_SCRIPT"
echo "BEGIN" >> "$SQL_SCRIPT"
echo "  FOR emp IN (SELECT column_value AS emp_id FROM TABLE(sys.dbms_debug_vc2coll(" >> "$SQL_SCRIPT"

awk '{print "  ''" $1 "''" ","}' "$SQL_INPUT_FILE" | sed '$ s/,$//' >> "$SQL_SCRIPT"

echo "  ))) LOOP" >> "$SQL_SCRIPT"
echo "    FOR user_rec IN (" >> "$SQL_SCRIPT"
echo "      SELECT username FROM dba_users WHERE INSTR(username, emp.emp_id) > 0" >> "$SQL_SCRIPT"
echo "    ) LOOP" >> "$SQL_SCRIPT"
echo "      DBMS_OUTPUT.PUT_LINE('Locking user: ' || user_rec.username);" >> "$SQL_SCRIPT"
echo "      EXECUTE IMMEDIATE 'ALTER USER ' || user_rec.username || ' ACCOUNT LOCK';" >> "$SQL_SCRIPT"
echo "    END LOOP;" >> "$SQL_SCRIPT"
echo "  END LOOP;" >> "$SQL_SCRIPT"
echo "END;" >> "$SQL_SCRIPT"
echo "/" >> "$SQL_SCRIPT"
echo "SPOOL OFF;" >> "$SQL_SCRIPT"

# Execute SQL script
sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" < "$SQL_SCRIPT" | tee -a "$LOG_FILE"

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors. Check log for details." | tee -a "$LOG_FILE"
fi

# Cleanup temporary files
rm -f "$SQL_SCRIPT"
