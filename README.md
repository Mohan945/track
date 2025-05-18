#!/bin/bash

# === Config ===
TODAY=$(date +%Y%m%d)
LOCAL_DIR="/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD"
REMOTE_DIR="/mount/PRODDBA/oracle/EB37DRMQ"
DUMP_PATTERN="DRM_EB37PROD_${TODAY}_*.dmp"
LOGFILE="DRM_EB37PROD_${TODAY}.log"

echo "=================== CP Test Script ==================="
echo "DATE: $TODAY"
echo "LOCAL_DIR: $LOCAL_DIR"
echo "REMOTE_DIR: $REMOTE_DIR"
echo "======================================================"

# === Step 1: Check if dump files exist ===
DUMPS_FOUND=$(ls ${LOCAL_DIR}/${DUMP_PATTERN} 2>/dev/null | wc -l)
if [ "$DUMPS_FOUND" -eq 0 ]; then
    echo "ERROR: No dump files found matching pattern: ${LOCAL_DIR}/${DUMP_PATTERN}"
    exit 1
else
    echo "INFO: Found $DUMPS_FOUND dump files to copy."
fi

# === Step 2: Check if log file exists ===
if [ ! -f "${LOCAL_DIR}/${LOGFILE}" ]; then
    echo "ERROR: Log file not found: ${LOCAL_DIR}/${LOGFILE}"
    exit 1
else
    echo "INFO: Log file found: ${LOGFILE}"
fi

# === Step 3: Clean remote directory ===
echo "Cleaning old files from ${REMOTE_DIR}..."
rm -f ${REMOTE_DIR}/DRM_EB37*.dmp
rm -f ${REMOTE_DIR}/DRM_EB37*.log

# === Step 4: Copy files ===
echo "Copying dump files to ${REMOTE_DIR}..."
cp ${LOCAL_DIR}/${DUMP_PATTERN} ${REMOTE_DIR}/

echo "Copying log file to ${REMOTE_DIR}..."
cp ${LOCAL_DIR}/${LOGFILE} ${REMOTE_DIR}/

echo "======================================================"
echo "Copy operation completed successfully!"
ls -l ${REMOTE_DIR}/DRM_EB37*${TODAY}*
