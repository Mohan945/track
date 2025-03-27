#!/bin/bash

# Define file paths (NFS location for shared access)
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"   # File containing all employee data
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"  # Filtered list of Employee IDs
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"  # Log file for tracking

# Clear log file before execution
> "$LOG_FILE"

# Extract Employee IDs where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk '{print $3}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Execute the SQL script dynamically
{
    echo "SET SERVEROUTPUT ON;"
    echo "SPOOL $LOG_FILE;"
    echo "BEGIN"
    echo "  FOR emp IN (SELECT column_value AS emp_id FROM TABLE(sys.dbms_debug_vc2coll("
    awk '{print "'"'"'" $1 "'"'"'","}' "$SQL_INPUT_FILE" | sed '$ s/,$//'
    echo "  ))) LOOP"
    echo "    FOR user_rec IN (SELECT username FROM dba_users WHERE username LIKE emp.emp_id || '_%') LOOP"
    echo "      EXECUTE IMMEDIATE 'ALTER USER ' || user_rec.username || ' ACCOUNT LOCK';"
    echo "    END LOOP;"
    echo "  END LOOP;"
    echo "END;"
    echo "/"
    echo "SPOOL OFF;"
} | sqlplus -s / as sysdba >> "$LOG_FILE" 2>&1

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors. Check log for details." | tee -a "$LOG_FILE"
fi

awk: cmd. line:1: {print "'" $1 "'","}
awk: cmd. line:1:                   ^ unterminated string
awk: cmd. line:1: {print "'" $1 "'","}
awk: cmd. line:1:                   ^ syntax error
