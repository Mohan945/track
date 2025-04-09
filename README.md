DATA_PUMP_DIR -- PROD directory
/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD

expdp system/"******* " directory=DATA_PUMP_DIR schemas=DRM_PROD dumpfile=DRM_EB37_28_%U.dmp logfile=DRM_EB37_28.log parallel=5
