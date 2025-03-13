
#!/bin/bash

# Input file containing database names
DB_LIST="db_list.txt"

# Output log file
LOG_FILE="tnsping_results.log"

# Clear previous log file
> $LOG_FILE

# Loop through each database name and run tnsping
while IFS= read -r DB_NAME; do
    echo "Checking TNSPING for: $DB_NAME"
    
    # Run tnsping and capture the result
    TNS_RESULT=$(tnsping $DB_NAME 2>&1)
    
    # Check if tnsping was successful
    if echo "$TNS_RESULT" | grep -q "OK"; then
        echo "$DB_NAME: SUCCESS" | tee -a $LOG_FILE
    else
        echo "$DB_NAME: FAILED" | tee -a $LOG_FILE
    fi
done < "$DB_LIST"

echo "TNSPING check completed. Results saved in $LOG_FILE."
