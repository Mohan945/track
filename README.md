# === Step 6: Detect Dump File Pattern Dynamically ===
DUMP_DIR_PATH="/mount/PRODDBA/oracle/EB37DRMQ"
IMP_DUMP_DIR="DATA_PUMP_DRMQ"

echo "Detecting dump pattern in: $DUMP_DIR_PATH"
DUMPFILE_PATTERN=""

for file in "$DUMP_DIR_PATH"/DRM_EB37_*.dmp; do
  if [ -f "$file" ]; then
    FILENAME=$(basename "$file")
    # Convert something like DRM_EB37_20250413_01.dmp => DRM_EB37_20250413_%U.dmp
    DUMPFILE_PATTERN=$(echo "$FILENAME" | sed -E 's/_[0-9]+\.dmp$/_%U.dmp/')
    echo "Detected dump pattern: $DUMPFILE_PATTERN"
    break
  fi
done

if [ -z "$DUMPFILE_PATTERN" ]; then
  echo "ERROR: No dump file found in $DUMP_DIR_PATH"
  exit 1
fi

echo "Starting import using: $DUMPFILE_PATTERN"

impdp system/"cowboy_1" \
  directory=$IMP_DUMP_DIR \
  dumpfile=$DUMPFILE_PATTERN \
  logfile=imp_${TARGET_SCHEMA}_$(date +%Y%m%d).log \
  remap_schema=${SRC_SCHEMA}:${TARGET_SCHEMA}
