#!/bin/ksh

LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"
REPORT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_report.html"
EMAIL_RECIPIENTS="security_team@example.com"

# Clear previous report
> "$REPORT_FILE"

# Variables
declare -A LOCKED_USERS
CURRENT_DB=""

# Process the log file
while read -r line; do
    if [[ $line == *"USER_LOCKED:"* ]]; then
        DB_NAME=$(echo "$line" | awk -F'IN_DB: ' '{print $2}')
        USERNAME=$(echo "$line" | awk -F'USER_LOCKED: ' '{print $2}' | awk '{print $1}')
        LOCKED_USERS["$DB_NAME"]+="$USERNAME,"
    fi
done < "$LOG_FILE"

# Start HTML formatting
echo "<html><body>" > "$REPORT_FILE"

if [ ${#LOCKED_USERS[@]} -eq 0 ]; then
    echo "<h3>No users got locked today.</h3>" >> "$REPORT_FILE"
else
    echo "<h3>Locked Users Report</h3>" >> "$REPORT_FILE"
    echo "<table border='1' cellspacing='0' cellpadding='5'>" >> "$REPORT_FILE"
    echo "<tr><th>Database</th><th>Locked Users</th></tr>" >> "$REPORT_FILE"

    for DB in "${!LOCKED_USERS[@]}"; do
        USER_LIST=$(echo "${LOCKED_USERS[$DB]}" | sed 's/,$//')  # Remove trailing comma
        echo "<tr><td>$DB</td><td>$USER_LIST</td></tr>" >> "$REPORT_FILE"
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
