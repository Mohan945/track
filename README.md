#!/bin/bash
echo "Checking ORACLE_HOME for all running databases..."
for SID in $(ps -ef | grep pmon | grep -v grep | awk -F_ '{print $3}'); do
    ORA_HOME=$(ps -ef | grep $SID | grep -v grep | awk '{for(i=1;i<=NF;i++) if($i ~ /oracle\/product/) print $i}')
    echo "Database: $SID  -->  ORACLE_HOME: $ORA_HOME"
done
