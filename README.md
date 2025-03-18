#!/bin/bash

# Define file paths
INPUT_FILE="employee_ids.txt"
SQL_INPUT_FILE="filtered_ids.txt"
LOG_FILE="lock_users.log"
TNS_ADMIN="/path/to/your/tnsadmin"  # Update with actual TNS file path
PASSWORD_FILE="/path/to/.oracle_pass"

# Oracle credentials
ORACLE_USER="your_username"
ORACLE_PASS=$(<"$PASSWORD_FILE")

# Create/clear log file
> "$LOG_FILE"

# Extract Employee IDs from the input file where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $3}' > "$SQL_INPUT_FILE"

# Read databases from TNS file and execute SQL script on each
while IFS= read -r DB; do
    echo "[$(date)] Processing database: $DB" | tee -a "$LOG_FILE"
    sqlplus -s "$ORACLE_USER/$ORACLE_PASS@$DB" @lock_users.sql "$SQL_INPUT_FILE" >> "$LOG_FILE" 2>&1
done < "$TNS_ADMIN/tns_list.txt"

echo "[$(date)] User locking process completed." | tee -a "$LOG_FILE"
