#!/bin/bash

# Directory where log files are stored
LOG_DIR="/path/to/log/files"

# Output merged log file
MERGED_LOG_FILE="/path/to/log/files/merged_logs.log"

# Clear existing merged log file if it exists
> "$MERGED_LOG_FILE"

# Loop through all log files in the directory
for log_file in "$LOG_DIR"/*.log; do
    if [ -f "$log_file" ]; then
        # Extract filename without path
        log_filename=$(basename "$log_file")

        # Add filename as a heading in the merged log file
        echo "========== $log_filename ==========" >> "$MERGED_LOG_FILE"
        cat "$log_file" >> "$MERGED_LOG_FILE"
        echo -e "\n" >> "$MERGED_LOG_FILE"  # Add a blank line for readability
    fi
done

echo "✅ Merged log file created: $MERGED_LOG_FILE"
