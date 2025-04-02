for SID in $(ps -ef | grep pmon | grep -v grep | awk -F_ '{print $3}'); do
    export ORACLE_SID=$SID
    ORA_HOME=$(sqlplus -s / as sysdba <<EOF
SET PAGESIZE 0 FEEDBACK OFF VERIFY OFF HEADING OFF ECHO OFF;
SELECT SYS_CONTEXT('USERENV', 'ORACLE_HOME') FROM DUAL;
EXIT;
EOF
)
    echo "Database: $SID  -->  ORACLE_HOME: $ORA_HOME"
done
