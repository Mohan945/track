#!/bin/bash

DUMP_DIR="/mount/PRODDBA/oracle/EB37DRMQ"
PATTERN="DRM_EB37_*.dmp"

echo "Looking in directory: $DUMP_DIR"
echo "Using pattern: $PATTERN"
echo "---------------------------------------------"

found=0
for file in "$DUMP_DIR"/$PATTERN; do
  if [ -f "$file" ]; then
    found=1
    FILENAME=$(basename "$file")
    # Example: DRM_EB37_20250413_01.dmp => DRM_EB37_20250413_%U.dmp
    NORMALIZED_NAME=$(echo "$FILENAME" | sed -E 's/_[0-9]+\.dmp$/_%U.dmp/')
    echo "$NORMALIZED_NAME"
    break  # Print only once
  fi
done

if [ $found -eq 0 ]; then
  echo "No matching dump files found in $DUMP_DIR"
fi
