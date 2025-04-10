#!/bin/bash

# === Variables ===
DB_NAME="EB37PROD"
SCHEMA="DRM_PROD"
EXPORT_DIR="DATA_PUMP_DIR"
LOCAL_DIR="/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD"
REMOTE_USER="oracle"
REMOTE_HOST="exa5dbafmo1"
REMOTE_DIR="/path/on/exadata"  # Replace with real target path
TODAY=$(date +%Y%m%d)
EMAIL_TO="your.email@domain.com"  # Replace with your email
HOSTNAME=$(hostname)

DUMPFILE="DRM_EB37_${TODAY}_%U.dmp"
LOGFILE="DRM_EB37_${TODAY}.log"

# === Oracle environment ===
export ORACLE_SID=${DB_NAME}
export ORAENV_ASK=NO
. /usr/local/bin/oraenv

# === Start Export ===
expdp system/"YourPasswordHere" \
  directory=${EXPORT_DIR} \
  schemas=${SCHEMA} \
  dumpfile=${DUMPFILE} \
  logfile=${LOGFILE} \
  parallel=5

EXPORT_STATUS=$?

# === SCP files ===
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}_*.dmp ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}.log ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/

SCP_STATUS=$?

# === Send Email Notification ===
if [[ $EXPORT_STATUS -eq 0 && $SCP_STATUS -eq 0 ]]; then
    SUBJECT="EB37 Export & SCP Successful - ${TODAY}"
    BODY="Export and SCP to ${REMOTE_HOST} completed successfully on ${HOSTNAME} for schema ${SCHEMA}."
else
    SUBJECT="EB37 Export/SCP Failed - ${TODAY}"
    BODY="One or more steps failed on ${HOSTNAME}.\n\nExport status: $EXPORT_STATUS\nSCP status: $SCP_STATUS\nPlease check the log file: ${LOCAL_DIR}/${LOGFILE}"
fi

echo -e "$BODY" | mail -s "$SUBJECT" "$EMAIL_TO"
