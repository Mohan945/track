# === Step 6: Detect Dump File and Import ===
DUMP_DIR_PATH="/mount/PRODDBA/oracle/EB37DRMQ"
IMP_DUMP_DIR="DATA_PUMP_DRMQ"
found=0

echo "Scanning dump files in: $DUMP_DIR_PATH"

for file in "$DUMP_DIR_PATH"/DRM_EB37_*.dmp; do
  if [ -f "$file" ]; then
    FILENAME=$(basename "$file")
    echo "Found dump file: $FILENAME"
    DUMPFILE_PATTERN=$(echo "$FILENAME" | sed -E 's/_[0-9]+\.dmp$/_%U.dmp/')
    echo "Using dump pattern: $DUMPFILE_PATTERN"
    found=1
    break
  fi
done

if [ "$found" -eq 0 ]; then
  echo "ERROR: No dump files found for import!"
  exit 1
fi

IMP_LOGFILE="imp_${TARGET_SCHEMA}_$(date +%Y%m%d).log"

echo "Starting import using detected dump pattern..."
impdp system/"cowboy_1" \
  directory=$IMP_DUMP_DIR \
  dumpfile=$DUMPFILE_PATTERN \
  logfile=$IMP_LOGFILE \
  remap_schema=${SRC_SCHEMA}:${TARGET_SCHEMA}
