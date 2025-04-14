# === Step 8: Email Notification (Added) ===
# Change the email address to your desired recipient.
MAIL_TO="dba_team@example.com"
MAIL_SUBJECT="Schema Refresh Completed for $TARGET_SCHEMA on $(date +%Y%m%d)"
mail -s "$MAIL_SUBJECT" "$MAIL_TO" < "$LOGFILE"
