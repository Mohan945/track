#!/bin/bash

# === Step 0: Setup Logging ===
LOGFILE="/mount/PRODDBA/oracle_scripts/leavers/schema_refresh_$(date +%Y%m%d_%H%M%S).log"
exec > >(tee -a "$LOGFILE") 2>&1

echo "===== Starting Schema Refresh Process ====="
START_TIME=$(date)

# === Step 1: Read Config File ===
CONFIG_FILE="/mount/PRODDBA/oracle_scripts/leavers/schema_refresh.cfg"
# Set the DB name (the Oracle instance name) here.
DB_NAME="EB37GDRQ"

# Read a config line with the format: DB_NAME:SOURCE_SCHEMA:TARGET_SCHEMA
LINE=$(grep "^${DB_NAME}:" "$CONFIG_FILE")
if [ -z "$LINE" ]; then
  echo "ERROR: Config not found for DB $DB_NAME"
  exit 1
fi

# Parse fields from the config file
SRC_SCHEMA=$(echo "$LINE" | cut -d':' -f2)
TARGET_SCHEMA=$(echo "$LINE" | cut -d':' -f3)

echo "Database (ORACLE_SID): $DB_NAME"
echo "Source Schema (for import remap): $SRC_SCHEMA"
echo "Target Schema (to be refreshed): $TARGET_SCHEMA"

# === Step 2: Set Oracle Environment ===
export ORACLE_SID=${DB_NAME}
export ORAENV_ASK=NO
. /usr/local/bin/oraenv > /dev/null 2>&1

# === Step 3: Verify Environment ===
echo "Verifying Oracle Environment:"
echo "ORACLE_SID = $ORACLE_SID"
echo "ORACLE_HOME = $ORACLE_HOME"
echo "PATH = $PATH"
echo "SQL*Plus location: $(which sqlplus)"
echo "Environment check complete."
echo "--------------------------"


# === Step 5: Generate & Run Drop Queries ===
DROP_SCRIPT="/tmp/drop_${TARGET_SCHEMA}_objects.sql"
echo "Generating DROP statements for $TARGET_SCHEMA..."
sqlplus -s system/"cowboy_1" <<EOF
set pages 0
set lines 300
set heading off
spool $DROP_SCRIPT
select 'drop table '||owner||'.'||table_name||' cascade constraints purge;'
  from dba_tables
  where owner = upper('$TARGET_SCHEMA')
union all
select 'drop '||object_type||' '||owner||'.'||object_name||';'
  from dba_objects
  where object_type not in ('TABLE','INDEX','PACKAGE BODY','TRIGGER','LOB')
    and object_type not like '%LINK%'
    and object_type not like '%PARTITION%'
    and owner = upper('$TARGET_SCHEMA')
order by 1;
spool off
EOF

echo "-----Running DROP statements..."
sqlplus -s system/"cowboy_1" @"$DROP_SCRIPT" <<EOF
EOF
echo "Drop complete."
echo "--------------------------"

# === Step 6: Import from Prod SCP Dump ===
# The dump files should be named as per production exports, for example: DRM_EB37_*.dmp
IMP_DUMP_DIR=DATA_PUMP_DRMQ
DUMP_DIR_PATH=/mount/PRODDBA/oracle/EB37DRMQ
DUMP_PATTERN=DRM_EB37_*%U.dmp
IMP_LOGFILE=imp_${TARGET_SCHEMA}_$(date +%Y%m%d).log


# If DUMP_DATE is not set, default to yesterday's date.
: ${DUMP_DATE:=$(date -d "yesterday" +%Y%m%d)}

impdp system/"cowboy_1" \
  directory=$IMP_DUMP_DIR \
  dumpfile=DRM_EB37_${DUMP_DATE}_%U.dmp \
  logfile=imp_${TARGET_SCHEMA}_${DUMP_DATE}.log \
  remap_schema=${SRC_SCHEMA}:${TARGET_SCHEMA} 

echo "Import completed: $IMP_LOGFILE"
echo "--------------------------"

# === Step 7: Completion ===
END_TIME=$(date)
echo "===== Schema Refresh Completed ====="
echo "Start Time: $START_TIME"
echo "End Time  : $END_TIME"
echo "Log File  : $LOGFILE"

# === Step 8: Email Notification (Added) ===
# Change the email address to your desired recipient.
MAIL_TO="gouk_oracle@standardlife.com"
MAIL_SUBJECT="Schema Refresh Completed for $TARGET_SCHEMA on $(date +%Y%m%d)"
mail -s "$MAIL_SUBJECT" "$MAIL_TO" < "$LOGFILE"

getting below error but manually it is working

ORA-39001: invalid argument value
ORA-39000: bad dump file specification
ORA-31640: unable to open dump file "/mount/PRODDBA/oracle/EB37DRMQ/DRM_EB37_20250413_01.dmp" for read
ORA-27037: unable to obtain file status
Linux-x86_64 Error: 2: No such file or directory

