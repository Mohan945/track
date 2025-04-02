#!/bin/ksh
####################################
# Locking Process with Dynamic Env #
####################################

# Define file paths (same as helpdesk script)
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/tnsnames.ora"
DB_HOME_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/all_inst_nonDataGuard_Y"

# Database credentials
DB_USER="SYSTEM"
DB_PASS="cowboy_1"

# Hardcoded list of databases to process
DB_NAMES=("CI01SYST" "OA21SYST" "MI22SYST")  # Extend as needed

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

    #############################################
    # Dynamic Environment Setup for ORACLE_HOME #
    #############################################
    
    # Extract the first 4 characters from DB_NAME to match DB_HOME_FILE entry
    CONN_STR=`echo $DB_NAME | cut -c 1-4`
    
    # Extract the Oracle version from DB_HOME_FILE
    ORACLE_VERSION=`awk -F ":" -v db="$CONN_STR" '$1 == db {print $2}' "$DB_HOME_FILE"`
    
    # Find the corresponding ORACLE_HOME dynamically
    case "$ORACLE_VERSION" in
        "19.0.0.0")
            export ORACLE_HOME=`find /u01/app/oracle/product -maxdepth 2 -type d -name "19.0.0.0" | head -n 1`
            ;;
        "12.2.0.1")
            export ORACLE_HOME=`find /u01/app/oracle/product -maxdepth 2 -type d -name "12.2.0.1" | head -n 1`
            ;;
        *)
            echo "[$(date)] Unknown Oracle version for $DB_NAME. Skipping." | tee -a "$LOG_FILE"
            continue
            ;;
    esac

    # Ensure ORACLE_HOME is set
    if [ -z "$ORACLE_HOME" ]; then
        echo "[$(date)] ORACLE_HOME not found for version $ORACLE_VERSION. Skipping database: $DB_NAME." | tee -a "$LOG_FILE"
        continue
    fi

    export PATH="$ORACLE_HOME/bin:$PATH"
    export LD_LIBRARY_PATH="$ORACLE_HOME/lib"
    export ORACLE_SID="$DB_NAME"

    echo "[$(date)] Environment set: ORACLE_HOME=$ORACLE_HOME, ORACLE_SID=$ORACLE_SID" | tee -a "$LOG_FILE"

    # Verify sqlplus is accessible
    if ! command -v sqlplus >/dev/null 2>&1; then
        echo "[$(date)] sqlplus not found for $DB_NAME. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    #############################################
    # Build the SQL script dynamically          #
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

    #####################################################
    # Execute SQL script using dynamically created env #
    #####################################################
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
