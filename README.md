#!/bin/ksh

LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
REPORT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_report.html"
EMAIL_RECIPIENTS="security_team@example.com"

# Clear previous report
> "$REPORT_FILE"

# Temporary file to store results
TEMP_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/locked_users.tmp"
> "$TEMP_FILE"

# Process the log file to extract locked users
CURRENT_DB=""
USER_LOCKED=0

while read -r line; do
    if [[ $line == *"Processing database:"* ]]; then
        CURRENT_DB=$(echo "$line" | awk '{print $NF}')
    elif [[ $line == *"Locking user:"* ]]; then
        USERNAME=$(echo "$line" | awk '{print $NF}')
        echo "$CURRENT_DB|$USERNAME" >> "$TEMP_FILE"
        USER_LOCKED=1
    fi
done < "$LOG_FILE"

# Start HTML formatting
echo "<html><body>" > "$REPORT_FILE"

if [ "$USER_LOCKED" -eq 0 ]; then
    echo "<h3>No users got locked today.</h3>" >> "$REPORT_FILE"
else
    echo "<h3>Locked Users Report</h3>" >> "$REPORT_FILE"
    echo "<table border='1' cellspacing='0' cellpadding='5'>" >> "$REPORT_FILE"
    echo "<tr><th>Database</th><th>Locked Users</th></tr>" >> "$REPORT_FILE"

    # Sort and process unique databases
    awk -F'|' '{print $1}' "$TEMP_FILE" | sort -u | while read -r DB_NAME; do
        USERS=$(awk -F'|' -v db="$DB_NAME" '$1 == db {print $2}' "$TEMP_FILE" | paste -sd ", " -)
        echo "<tr><td>$DB_NAME</td><td>$USERS</td></tr>" >> "$REPORT_FILE"
    done

    echo "</table>" >> "$REPORT_FILE"
fi

echo "</body></html>" >> "$REPORT_FILE"

# Send the email
/usr/sbin/sendmail -t <<EOF
To: $EMAIL_RECIPIENTS
Subject: Oracle User Locking Report
Content-Type: text/html
$(cat "$REPORT_FILE")
EOF

# Cleanup temp file
rm -f "$TEMP_FILE"
