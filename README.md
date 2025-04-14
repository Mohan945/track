
# If DUMP_DATE is not set, default to yesterday's date.
: ${DUMP_DATE:=$(date -d "yesterday" +%Y%m%d)}

impdp system/"your_password" \
  directory=$IMP_DUMP_DIR \
  dumpfile="DRM_EB37_${DUMP_DATE}_%U.dmp" \
  logfile=imp_${TARGET_SCHEMA}_${DUMP_DATE}.log \
  remap_schema=${SRC_SCHEMA}:${TARGET_SCHEMA} \
  parallel=5
