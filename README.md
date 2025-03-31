
#!/bin/ksh

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"
TNS_FILE="/oem/oracle/admin/network/tnsnames.ora"
DB_LIST="/mount/PRODDBA/oracle_scripts/recert/leavers/test/db_list.txt"

# Database credentials
DB_USER="SYSTEM"

# Prompt for password securely
echo -n "Enter password for $DB_USER: "
stty -echo
read DB_PASS
stty echo
echo ""

# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Function to get ORACLE_HOME from /etc/oratab
get_oracle_home() {
    DB_NAME="$1"
    ORACLE_HOME=$(grep "^$DB_NAME:" /etc/oratab | cut -d: -f2)

    if [ -z "$ORACLE_HOME" ]; then
        echo "[$(date)] ERROR: ORACLE_HOME not found for database $DB_NAME in /etc/oratab." | tee -a "$LOG_FILE"
        return 1
    fi

    export ORACLE_HOME
    export PATH="$ORACLE_HOME/bin:$PATH"
    return 0
}

# Process each database in the predefined list
while read DB_NAME; do
    echo "[$(date)] Processing database: $DB_NAME" | tee -a "$LOG_FILE"

    # Get ORACLE_HOME dynamically
    get_oracle_home "$DB_NAME"
    if [ $? -ne 0 ]; then
        echo "[$(date)] Skipping database: $DB_NAME due to missing ORACLE_HOME." | tee -a "$LOG_FILE"
        continue
    fi

    # Validate if the database exists in the TNS file
    if ! grep -q -w "$DB_NAME" "$TNS_FILE"; then
        echo "[$(date)] ERROR: Database $DB_NAME not found in TNS file. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

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
    echo "    DBMS_OUTPUT.PUT_LINE('No matching users found to lock.');" >> "$SQL_SCRIPT"
    echo "  END IF;" >> "$SQL_SCRIPT"
    echo "END;" >> "$SQL_SCRIPT"
    echo "/" >> "$SQL_SCRIPT"
    echo "SPOOL OFF;" >> "$SQL_SCRIPT"

    # Execute SQL script
    sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"

    # Check for errors
    if [ $? -eq 0 ]; then
        echo "[$(date)] Successfully processed database: $DB_NAME" | tee -a "$LOG_FILE"
    else
        echo "[$(date)] ERROR processing database: $DB_NAME" | tee -a "$LOG_FILE"
    fi

done < "$DB_LIST"

echo "[$(date)] Locking process completed." | tee -a "$LOG_FILE"
