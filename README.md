# === Configuration ===
DB_NAME="EB37GPRO"
SCHEMA="DRM_PROD"
EXPORT_DIR="DATA_PUMP_DIR"
LOCAL_DIR="/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD"
REMOTE_DIR="/mount/PRODDBA/oracle/EB37DRMQ/"
EMAIL_TO="gouk_oracle@standardlife.com"
TODAY=$(date +%Y%m%d)

DUMPFILE="DRM_EB37PROD_${TODAY}_%U.dmp"
LOGFILE="DRM_EB37PROD_${TODAY}.log"
LOGFILE_FULL="/mount/PRODDBA/oracle_scripts/leavers/export_scp_${DB_NAME}_${TODAY}.log"
EMAIL_SUBJECT="Export and Copy Report for ${DB_NAME} - ${TODAY}"

# === Start Logging ===
exec > >(tee -a "$LOGFILE_FULL") 2>&1

echo "================================================================================"
echo "Export + Copy Script Started - $(date)"
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

# === Read password from file ===
PWD_FILE="/mount/PRODDBA/oracle_scripts/leavers/db_credentials.txt"
PASSWORD=$(<"$PWD_FILE")

# === DB Connection Test ===
echo "Testing DB Connection..."
echo "select name from v\$database;" | sqlplus -s system/"${PASSWORD}"
if [ $? -ne 0 ]; then
    echo "ERROR: Oracle environment or DB connection failed."
    echo "Export FAILED: Cannot connect to DB $DB_NAME." | mail -s "Export FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi

# === Run export ===
echo "Starting export on ${DB_NAME}..." >> $LOGFILE_FULL
expdp system/"${PASSWORD}" \
  directory=${EXPORT_DIR} \
  schemas=${SCHEMA} \
  dumpfile=${DUMPFILE} \
  logfile=${LOGFILE} \
  parallel=5 >> $LOGFILE_FULL 2>&1

if [ $? -ne 0 ]; then
    echo "ERROR: Export failed."
    echo "Export FAILED for $DB_NAME." | mail -s "Export FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi
echo "Export completed successfully."

# === Remove old files from REMOTE_DIR ===
echo "Removing old DRM_EB37 files from $REMOTE_DIR..."
rm -f ${REMOTE_DIR}/DRM_EB37*.dmp
rm -f ${REMOTE_DIR}/DRM_EB37*.log

# === Copy to REMOTE_DIR ===
echo "Copying files to $REMOTE_DIR..."
cp ${LOCAL_DIR}/DRM_EB37_${TODAY}_*.dmp ${REMOTE_DIR}
cp ${LOCAL_DIR}/DRM_EB37_${TODAY}.log ${REMOTE_DIR}

if [ $? -ne 0 ]; then
    echo "ERROR: Copy failed."
    echo "COPY FAILED for $DB_NAME." | mail -s "COPY FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi
echo "Copy completed successfully."

# === Email final log ===
echo "Sending email to $EMAIL_TO..."
mail -s "$EMAIL_SUBJECT" "$EMAIL_TO" < "$LOGFILE_FULL"

echo "================================================================================"
echo "Script Completed - $(date)"
echo "================================================================================"
