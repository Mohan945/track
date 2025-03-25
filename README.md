#!/bin/bash

# Define file paths (NFS location for shared access)
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"   # File containing all employee data
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"  # Filtered list of Employee IDs
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"  # Log file for tracking

# Create/clear log file
> "$LOG_FILE"

# Extract Employee IDs from input file where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $3}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Execute the SQL script using OEM-provided credentials
sqlplus -s system/"cowboy_1"@CI01SYST  @/mount/PRODDBA/oracle_scripts/recert/leavers/test/test1.sql "$SQL_INPUT_FILE" >> "$LOG_FILE" 2>&1

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors." | tee -a "$LOG_FILE"
fi
