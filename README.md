echo "Cleaning old files in ${REMOTE_DIR}..."
rm -f ${REMOTE_DIR}/DRM_EB37*.dmp ${REMOTE_DIR}/DRM_EB37*.log

echo "Copying dump files..."
find "${LOCAL_DIR}" -maxdepth 1 -type f -name "DRM_EB37PROD_${TODAY}_*.dmp" -exec cp {} "${REMOTE_DIR}/" \;

echo "Copying log file..."
cp "${LOCAL_DIR}/DRM_EB37PROD_${TODAY}.log" "${REMOTE_DIR}/"

echo "All files copied successfully."
