#!/bin/bash

# Define file paths (using NFS for shared access)
INPUT_FILE="/mnt/nfs/employee_ids.txt"   # Master file with all employee details
SQL_INPUT_FILE="/mnt/nfs/filtered_ids.txt"  # File that will contain only the filtered Employee IDs
LOG_FILE="/mnt/nfs/lock_users.log"         # Log file for tracking

# Create/clear the log file
> "$LOG_FILE"

# Extract Employee IDs from the input file where the Details field contains specific keywords.
# Assumes the file is tab-delimited and the Employee ID is the third field.
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $3}' > "$SQL_INPUT_FILE"

# Check if the filtered file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Read each Employee ID from the filtered file and pass it as an argument to the SQL script
while IFS= read -r employee_id; do
    echo "[$(date)] Processing Employee ID: $employee_id" | tee -a "$LOG_FILE"
    
    # Execute the SQL script using OEM-provided credentials (no hardcoded username/password)
    sqlplus -s /nolog <<EOF >> "$LOG_FILE" 2>&1
CONNECT /
@/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.sql "$employee_id"
EXIT
EOF

    # Check for errors for this Employee ID
    if [ $? -eq 0 ]; then
        echo "[$(date)] User lock process for $employee_id completed successfully." | tee -a "$LOG_FILE"
    else
        echo "[$(date)] Error encountered while processing $employee_id." | tee -a "$LOG_FILE"
    fi
done < "$SQL_INPUT_FILE"

echo "[$(date)] User locking process completed for all employees." | tee -a "$LOG_FILE"
