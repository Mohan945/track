#!/bin/bash

# === Configuration ===
DB_NAME="EB37PROD"
SCHEMA="DRM_PROD"
EXPORT_DIR="DATA_PUMP_DIR"
LOCAL_DIR="/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD"
REMOTE_USER="oracle"
REMOTE_HOST="exa5dbafmo1"  # Replace with actual IP/hostname
REMOTE_DIR="/path/on/exadata"  # Replace with actual target path
EMAIL_TO="your.email@yourdomain.com"
TODAY=$(date +%Y%m%d)

DUMPFILE="DRM_EB37_${TODAY}_%U.dmp"
LOGFILE="DRM_EB37_${TODAY}.log"
LOGFILE_FULL="/tmp/export_scp_${DB_NAME}_${TODAY}.log"
EMAIL_SUBJECT="Export and SCP Report for ${DB_NAME} - ${TODAY}"

# === Start Logging ===
exec > >(tee -a "$LOGFILE_FULL") 2>&1

echo "================================================================================"
echo "Export + SCP Script Started - $(date)"
echo "================================================================================"

# === Oracle environment ===
export ORACLE_SID=${DB_NAME}
export ORAENV_ASK=NO
. /usr/local/bin/oraenv

# === Validate environment ===
echo "Environment Variables:"
echo "ORACLE_SID=$ORACLE_SID"
echo "ORACLE_HOME=$ORACLE_HOME"
echo "PATH=$PATH"
echo "SQL*Plus location: $(which sqlplus)"

# === DB Connection Test ===
echo "Testing DB Connection..."
echo "select name from v\$database;" | sqlplus -s / as sysdba
if [ $? -ne 0 ]; then
    echo "ERROR: Oracle environment or DB connection failed."
    echo "Export FAILED: Cannot connect to DB $DB_NAME." | mail -s "Export FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi

# === Run export ===
echo "Starting export..."
expdp system/"YourPasswordHere" \
  directory=${EXPORT_DIR} \
  schemas=${SCHEMA} \
  dumpfile=${DUMPFILE} \
  logfile=${LOGFILE} \
  parallel=5

if [ $? -ne 0 ]; then
    echo "ERROR: Export failed."
    echo "Export FAILED for $DB_NAME." | mail -s "Export FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi
echo "Export completed successfully."

# === SCP to Exadata ===
echo "Starting SCP transfer to $REMOTE_HOST..."
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}_*.dmp ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}.log ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/

if [ $? -ne 0 ]; then
    echo "ERROR: SCP failed."
    echo "SCP FAILED for $DB_NAME." | mail -s "SCP FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi
echo "SCP completed successfully."

# === Email final log ===
echo "Sending email to $EMAIL_TO..."
mail -s "$EMAIL_SUBJECT" "$EMAIL_TO" < "$LOGFILE_FULL"

echo "================================================================================"
echo "Script Completed - $(date)"
echo "================================================================================"
