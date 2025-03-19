#!/bin/bash

# Define file paths
INPUT_FILE="employee_ids.txt"
SQL_INPUT_FILE="filtered_ids.txt"
LOG_FILE="lock_users.log"

# Create/clear log file
> "$LOG_FILE"

# Extract Employee IDs from the input file where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $3}' > "$SQL_INPUT_FILE"

# Run SQL script using the credentials from OEM Job (No SYSDBA)
sqlplus -s / @lock_users.sql "$SQL_INPUT_FILE" >> "$LOG_FILE" 2>&1

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors." | tee -a "$LOG_FILE"
fi
