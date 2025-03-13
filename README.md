#!/bin/bash

# Input file containing database names
DB_LIST="db_list.txt"

# Output log file for failed connections
FAILED_LOG="tnsping_failed.log"

# Clear previous failed log file
> $FAILED_LOG

# Loop through each database name and run tnsping
while IFS= read -r DB_NAME; do
    echo "Checking TNSPING for: $DB_NAME"
    
    # Run tnsping and capture the result
    TNS_RESULT=$(tnsping $DB_NAME 2>&1)

    # Check if tnsping failed
    if ! echo "$TNS_RESULT" | grep -q "OK"; then
        echo "$DB_NAME: FAILED" | tee -a $FAILED_LOG
    fi
done < "$DB_LIST"

echo "TNSPING check completed. Failed connections saved in $FAILED_LOG."
