#!/bin/bash

# === Variables ===
DB_NAME="EB37PROD"
SCHEMA="DRM_PROD"
EXPORT_DIR="DATA_PUMP_DIR"
LOCAL_DIR="/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD"
REMOTE_USER="oracle"
REMOTE_HOST="exa5dbafmo1"
REMOTE_DIR="/path/on/exadata"  # <-- change to actual target path
TODAY=$(date +%Y%m%d)

DUMPFILE="DRM_EB37_${TODAY}_%U.dmp"
LOGFILE="DRM_EB37_${TODAY}.log"

# === Oracle environment ===
export ORACLE_SID=${DB_NAME}
export ORAENV_ASK=NO
. /usr/local/bin/oraenv

# === Run export ===
expdp system/"YourPasswordHere" \
  directory=${EXPORT_DIR} \
  schemas=${SCHEMA} \
  dumpfile=${DUMPFILE} \
  logfile=${LOGFILE} \
  parallel=5

# === SCP to Exadata ===
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}_*.dmp ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/
scp ${LOCAL_DIR}/DRM_EB37_${TODAY}.log ${REMOTE_USER}@${REMOTE_HOST}:${REMOTE_DIR}/

echo "Export completed and files copied to ${REMOTE_HOST}:${REMOTE_DIR}"
