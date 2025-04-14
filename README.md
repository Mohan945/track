#!/bin/bash

DUMP_DIR="/mount/PRODDBA/oracle/EB37DRMQ"
PATTERN="DRM_EB37_*.dmp"

echo "Normalized dump file name(s) using %U format:"
echo "---------------------------------------------"

for file in "$DUMP_DIR"/$PATTERN; do
  if [ -e "$file" ]; then
    # Extract date portion from filename
    FILENAME=$(basename "$file")
    
    # Example: DRM_EB37_20250413_01.dmp => DRM_EB37_20250413_%U.dmp
    NORMALIZED_NAME=$(echo "$FILENAME" | sed -E 's/_[0-9]+\.dmp$/_%U.dmp/')
    
    echo "$NORMALIZED_NAME"
    break  # Only need to print once since pattern is the same for all
  fi
done
