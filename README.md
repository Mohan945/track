#!/bin/ksh
####################################
# Locking Process with Dynamic Env #
# SCRIPT : Leavers_Automated
# VERSION : 001
# AUTHOR : Mohan Tippasamudram (CO70989)
# DATE : 2nd April 2025
# PURPOSE : Lock users as part of daily Leavers/Movers process
# Working : This Script will generate a lock script that will be executed in all DB's to lock the users
####################################

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/leavers/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/leavers/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/leavers/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/leavers/lock.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/leavers/tnsnames.ora"
DB_ENV_FILE="/mount/PRODDBA/oracle_scripts/leavers/all_inst_nonDataGuard_Y"
DB_CRED_FILE="/mount/PRODDBA/oracle_scripts/leavers/db_credentials.txt"

# Database credentials
DB_USER="SYSTEM"

# Securely read the database password from an external file
if [ ! -f "$DB_CRED_FILE" ]; then
    echo "[$(date)] ERROR: Credential file not found: $DB_CRED_FILE" | tee -a "$LOG_FILE"
    exit 1
fi

DB_PASS=$(cat "$DB_CRED_FILE" | tr -d '[:space:]')  # Remove any accidental spaces/newlines

# List of databases to process
DB_NAMES=("QP25PROD" "AU42PROD" "BM23PROD" "CN03PROD" "CN23PROD" "CX21PROD" "DL21PROD" "FF01PROD" "GX21PROD" "MI22PROD" "OA21PROD" "RI22PROD" "TZ21PROD" "QP24PROD" "AU43PROD" "BM24PROD" "CD21PROD" "EB23PROD" "MI21PROD" "TZ23PROD" "WT21PROD" "WT22PROD" "BM41PROD" "AU41PROD" "BM21PROD" "CI01PROD" "GP22PROD" "IN05PROD" "KW21PROD" "RD02PROD" "RD23PROD" "SQ01PROD" "TZ02PROD" "WT23PROD" "RD24PROD" "EB21PROD" "EB37PROD")

# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $2}' > "$SQL_INPUT_FILE"

# Check if Employee IDs exist
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Process each database
for DB_NAME in "${DB_NAMES[@]}"; do
    echo "[$(date)] Processing database: $DB_NAME" | tee -a "$LOG_FILE"

    # Check if database exists in TNS file
    if ! grep -q -w "$DB_NAME" "$TNS_FILE"; then
        echo "[$(date)] Database $DB_NAME not found in TNS file. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    #############################################
    # Dynamic ORACLE_HOME & ORACLE_SID Selection#
    #############################################

    # Extract database prefix (first 4 characters)
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

    # Set TNS_ADMIN to the correct directory
    export TNS_ADMIN="/mount/PRODDBA/oracle_scripts/leavers"

    # Extract ORACLE_SID dynamically for RAC
    export ORACLE_SID=$(ps -ef | grep pmon | grep -i "$DB_NAME" | awk -F_ '{print $3}')
    echo "[$(date)] Set ORACLE_SID: $ORACLE_SID" | tee -a "$LOG_FILE"

    # Verify sqlplus exists
    if ! command -v sqlplus > /dev/null; then
        echo "[$(date)] sqlplus not found in $ORACLE_HOME. Skipping." | tee -a "$LOG_FILE"
        continue
    fi

    # Test connection using tnsping
    tnsping "$DB_NAME" | tee -a "$LOG_FILE"

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

    #############################################
    # Execute SQL script Securely (Hides Passwords) #
    #############################################
    sqlplus -s /nolog <<EOF | tee -a "$LOG_FILE"
    CONNECT $DB_USER/$DB_PASS@$DB_NAME
    SET SERVEROUTPUT ON;
    SET HEADING OFF;
    SET FEEDBACK OFF;
    @$SQL_SCRIPT
    EXIT;
EOF

    SQL_EXIT_CODE=${PIPESTATUS[0]}

    if [ "$SQL_EXIT_CODE" -eq 0 ] || [ "$SQL_EXIT_CODE" -eq 2 ]; then
        echo "[$(date)] Successfully processed database: $DB_NAME" | tee -a "$LOG_FILE"
    else
        echo "[$(date)] Error processing database: $DB_NAME (Exit Code: $SQL_EXIT_CODE)" | tee -a "$LOG_FILE"
    fi

    echo "" | tee -a "$LOG_FILE"
done

echo "[$(date)] Locking process completed." | tee -a "$LOG_FILE"
