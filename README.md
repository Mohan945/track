#!/bin/ksh

#########################
# Locking Process Start #
#########################

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/tnsnames.ora"

# Database credentials
DB_USER="SYSTEM"
DB_PASS="cowboy_1"

# Hardcoded list of databases
DB_NAMES=("CI01SYST" "OA21SYST" "MI22SYST")  # Add your database names here

# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Process each database for locking accounts
for DB_NAME in "${DB_NAMES[@]}"; do
    echo "[$(date)] Processing database: $DB_NAME" | tee -a "$LOG_FILE"

    # Check if the database exists in the TNS file
    grep -q -w "$DB_NAME" "$TNS_FILE"
    if [ $? -ne 0 ]; then
        echo "[$(date)] Database $DB_NAME not found in TNS file. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    # Build the SQL script dynamically for locking accounts
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
    echo "EXIT;" >> "$SQL_SCRIPT"   # Ensures SQL*Plus exits after execution

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
done

echo "[$(date)] Locking process completed." | tee -a "$LOG_FILE"

#################################
# End of Locking Process        #
# Begin Locked Accounts Report  #
#################################

# Temporary file to hold the report rows
REPORT_FILE="/tmp/locked_report.txt"
> "$REPORT_FILE"  # Clear previous report

# Loop through each database to check for accounts locked today
for DB_NAME in "${DB_NAMES[@]}"; do
    # Query Oracle for accounts locked today via ALTER commands.
    LOCKED_ACCOUNTS=$(sqlplus -s "${DB_USER}/${DB_PASS}@${DB_NAME}" <<EOF
SET HEADING OFF
SET FEEDBACK OFF
SET PAGESIZE 0
SET LINESIZE 200
SELECT username FROM dba_audit_trail
 WHERE action_name='ALTER USER'
   AND UPPER(sql_text) LIKE '%ACCOUNT LOCK%'
   AND TO_CHAR(timestamp, 'YYYY-MM-DD') = TO_CHAR(SYSDATE, 'YYYY-MM-DD');
EXIT;
EOF
)
    # Clean up the output
    LOCKED_ACCOUNTS=$(echo "$LOCKED_ACCOUNTS" | sed '/^\s*$/d')

    # For each returned username, append a table row with DB name and EmpID
    if [ -n "$LOCKED_ACCOUNTS" ]; then
        for USER in $LOCKED_ACCOUNTS; do
            echo "<tr><td>${DB_NAME}</td><td>${USER}</td></tr>" >> "$REPORT_FILE"
        done
    fi
done

# Build the email with ONLY the HTML table (no <html> or <body>)
MAIL_BODY=$(cat <<EOF
<table border="1" cellspacing="0" cellpadding="5">
  <tr>
    <th>DB Name</th>
    <th>EmpID</th>
  </tr>
$(cat "$REPORT_FILE")
</table>
EOF
)

# Specify your email recipient here (adjust as needed)
RECIPIENT="recipient@example.com"

# Construct the email with proper headers and send via sendmail.
(
  echo "To: ${RECIPIENT}"
  echo "Subject: Today's Locked Accounts Report"
  echo "MIME-Version: 1.0"
  echo "Content-Type: text/html"
  echo ""
  echo "$MAIL_BODY"
) | sendmail -t

# Clean up temporary file
rm -f "$REPORT_FILE"
