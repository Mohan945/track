#!/bin/ksh

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"

# Database credentials
DB_USER="SYSTEM"
DB_PASS="cowboy_1"


# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Build the SQL script dynamically
> "$SQL_SCRIPT"

echo "SET SERVEROUTPUT ON;" >> "$SQL_SCRIPT"
echo "SPOOL $LOG_FILE;" >> "$SQL_SCRIPT"
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
echo "    DBMS_OUTPUT.PUT_LINE('No matching users found to lock.');" >> "$SQL_SCRIPT"
echo "  END IF;" >> "$SQL_SCRIPT"

echo "END;" >> "$SQL_SCRIPT"
echo "/" >> "$SQL_SCRIPT"
echo "SPOOL OFF;" >> "$SQL_SCRIPT"

# Execute SQL script
sqlplus -s "$DB_USER/$DB_PASS" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors. Check log for details." | tee -a "$LOG_FILE"
fi

# Cleanup temporary files
rm -f "$SQL_SCRIPT"
