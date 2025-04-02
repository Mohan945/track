#!/bin/bash
echo "Checking ORACLE_HOME for all running databases..."

for SID in $(ps -ef | grep pmon | grep -v grep | awk -F_ '{print $3}'); do
    # Find ORACLE_HOME using the process list
    ORA_HOME=$(ps -ef | grep $SID | grep -v grep | awk '{for(i=1;i<=NF;i++) if($i ~ /oracle\/product/) print $i}')

    # If ORACLE_HOME is empty, try finding it from /etc/oratab
    if [[ -z "$ORA_HOME" ]]; then
        ORA_HOME=$(grep -E "^$SID:" /etc/oratab | cut -d: -f2)
    fi

    # If still empty, print an error message
    if [[ -z "$ORA_HOME" ]]; then
        echo "Database: $SID  -->  ORACLE_HOME: NOT FOUND"
    else
        echo "Database: $SID  -->  ORACLE_HOME: $ORA_HOME"
    fi
done
