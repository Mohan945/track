#!/bin/ksh

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/tnsnames.ora"
LOCKED_USERS_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/locked_users.html"

# Database credentials
DB_USER="SYSTEM"
DB_PASS="cowboy_1"

# Hardcoded list of databases
DB_NAMES=("CI01SYST" "OA21SYST" "MI22SYST")  # Add your database names here

# Clear previous log file and locked users file
> "$LOG_FILE"
> "$LOCKED_USERS_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Add table header to the locked users file
echo "<table border='1'>" > "$LOCKED_USERS_FILE"
echo "<tr><th>Database Name</th><th>Employee ID</th></tr>" >> "$LOCKED_USERS_FILE"

# Process each database in the predefined list
for DB_NAME in "${DB_NAMES[@]}"; do
    echo "[$(date)] Processing database: $DB_NAME" | tee -a "$LOG_FILE"

    # Check if the database exists in the TNS file
    grep -q -w "$DB_NAME" "$TNS_FILE"
    if [ $? -ne 0 ]; then
        echo "[$(date)] Database $DB_NAME not found in TNS file. Skipping." | tee -a "$LOG_FILE"
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
    echo "EXIT;" >> "$SQL_SCRIPT"  # Ensures SQL*Plus exits after execution

    # Execute SQL script and capture the exit status
    sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" @"$SQL_SCRIPT" | tee -a "$LOG_FILE"
    SQL_EXIT_CODE=${PIPESTATUS[0]}

    # Allow exit codes 0 (success) and 2 (minor warnings)
    if [ "$SQL_EXIT_CODE" -eq 0 ] || [ "$SQL_EXIT_CODE" -eq 2 ]; then
        echo "[$(date)] Successfully processed database: $DB_NAME" | tee -a "$LOG_FILE"
    else
        echo "[$(date)] Critical error processing database: $DB_NAME (Exit Code: $SQL_EXIT_CODE)" | tee -a "$LOG_FILE"
    fi

    # Insert a blank line between databases in the log
    echo "" | tee -a "$LOG_FILE"

    # Fetch users locked today in the current database
    LOCKED_USERS=$(sqlplus -s "$DB_USER/$DB_PASS@$DB_NAME" <<EOF
SET HEADING OFF;
SET FEEDBACK OFF;
SELECT username FROM dba_users WHERE ACCOUNT_STATUS='LOCKED' AND LOCK_DATE >= TRUNC(SYSDATE);
EXIT;
EOF
)

    # Add locked users to the HTML table
    for USER in $LOCKED_USERS; do
        echo "<tr><td>$DB_NAME</td><td>$USER</td></tr>" >> "$LOCKED_USERS_FILE"
    done
done

# Close the HTML table
echo "</table>" >> "$LOCKED_USERS_FILE"

# Check if any users were locked today before sending email
if grep -q "<tr><td>" "$LOCKED_USERS_FILE"; then
    (
        echo "From: sender@example.com"
        echo "To: recipient@example.com"
        echo "Subject: Locked Users Report"
        echo "MIME-Version: 1.0"
        echo "Content-Type: text/html"
        echo ""
        cat "$LOCKED_USERS_FILE"  # Ensure the email only contains the table
    ) | /usr/bin/mailx -s "Locked Users Report" recipient@example.com
else
    echo "[$(date)] No users were locked today. Email not sent." | tee -a "$LOG_FILE"
fi

echo "[$(date)] Locking process completed." | tee -a "$LOG_FILE"
