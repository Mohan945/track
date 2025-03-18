#!/bin/bash

# Define input file path
INPUT_FILE="employee_ids.txt"
SQL_INPUT_FILE="filtered_ids.txt"
TNS_ADMIN="/path/to/your/tnsadmin"  # Update with the actual TNS file path

# Extract Employee IDs from the input file where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $3}' > "$SQL_INPUT_FILE"

# Define Oracle login credentials (Prompt user for security)
echo "Enter Oracle username:"
read ORACLE_USER
echo "Enter Oracle password:"
stty -echo
read ORACLE_PASS
stty echo
echo

# Read databases from TNS file and execute SQL script on each
for DB in $(cat $TNS_ADMIN/tns_list.txt); do
    echo "Processing database: $DB"
    sqlplus -s "$ORACLE_USER/$ORACLE_PASS@$DB" @lock_users.sql "$SQL_INPUT_FILE"
done

echo "User locking process completed."
