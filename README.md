#!/bin/bash

# === Variables ===
DB_NAME="EB37PROD"
SCHEMA="DRM_PROD"
EXPORT_DIR="DATA_PUMP_DIR"
LOCAL_DIR="/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD"
REMOTE_USER="oracle"
REMOTE_HOST="exa5dbafmo1"
REMOTE_DIR="/path/on/exadata"
TODAY=$(date +%Y%m%d)
LOGFILE_MAIN="/tmp/expdp_EB37PROD_${TODAY}.log"

DUMPFILE="DRM_EB37_${TODAY}_%U.dmp"
LOGFILE="DRM_EB37_${TODAY}.log"

# === Oracle environment ===
export ORACLE_SID=${DB_NAME}
export ORAENV_ASK=NO
. /usr/local/bin/oraenv >> $LOGFILE_MAIN 2>&1

# === Read password from file ===
PWD_FILE=~/expdp_pwd.txt
PASSWORD=$(<"$PWD_FILE")

# === Run export ===
echo "Starting export on ${DB_NAME}..." >> $LOGFILE_MAIN
expdp system/"${PASSWORD}" \
  directory=${EXPORT_DIR} \
  schemas=${SCHEMA} \
  dumpfile=${DUMPFILE} \
  logfile=${LOGFILE} \
  parallel=5 >> $LOGFILE_MAIN 2>&1

# === SCP to Exadata ===
echo "Copying dump files to Exadata..." >> $LOGFILE_MAIN
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}_*.dmp ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/ >> $LOGFILE_MAIN 2>&1
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}.log ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/ >> $LOGFILE_MAIN 2>&1

# === Email notification ===
mailx -s "EXPDP EB37PROD Backup Report - ${TODAY}" you@example.com < $LOGFILE_MAIN

echo "Export completed and files copied. Email sent." >> $LOGFILE_MAIN
