#!/bin/bash

# Define file paths
INPUT_FILE="employee_ids.txt"
SQL_INPUT_FILE="filtered_ids.txt"
LOG_FILE="lock_users.log"

# Clear the log file before execution
> "$LOG_FILE"

# Extract Employee IDs from the input file where the 'Details' field contains specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $3}' > "$SQL_INPUT_FILE"

# Check if the filtered file has data
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting..." | tee -a "$LOG_FILE"
    exit 0
fi

# Run SQL script (OEM provides the credentials at execution)
echo "[$(date)] Starting user locking process..." | tee -a "$LOG_FILE"
sqlplus -s /nolog <<EOF >> "$LOG_FILE" 2>&1
CONNECT / AS SYSDBA
@lock_users.sql $SQL_INPUT_FILE
EXIT
EOF

# Check SQL*Plus execution status
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors." | tee -a "$LOG_FILE"
fi
