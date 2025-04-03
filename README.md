#!/bin/ksh
####################################
# Locking Process with Dynamic Env #
# SCRIPT : Leavers_Automated
# VERSION : 001
# AUTHOR : Mohan Tippasamudram (CO70989)
# DATE : 3rd April 2025
# PURPOSE : Lock users as part of daily Leavers/Movers process
####################################

# Define file paths
INPUT_FILE="/mount/PRODDBA/oracle_scripts/leavers/employee_ids_csv.txt"
EMPLOYEE_FILE_COL1="/mount/PRODDBA/oracle_scripts/leavers/employee_ids.txt"
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/leavers/filtered_ids.txt"
LOG_FILE="/mount/PRODDBA/oracle_scripts/leavers/lock_users.log"
SQL_SCRIPT="/mount/PRODDBA/oracle_scripts/leavers/lock.sql"
TNS_FILE="/mount/PRODDBA/oracle_scripts/leavers/tnsnames.ora"
DB_ENV_FILE="/mount/PRODDBA/oracle_scripts/leavers/all_inst_nonDataGuard_Y"
DB_CRED_FILE="/mount/PRODDBA/oracle_scripts/leavers/db_credentials.txt"

# Temporary files for column extraction
FILTERED_COL1="/mount/PRODDBA/oracle_scripts/leavers/filtered_ids_col1.txt"
FILTERED_COL2="/mount/PRODDBA/oracle_scripts/leavers/filtered_ids_col2.txt"

# Database credentials
DB_USER="SYSTEM"

# Securely read the database password from an external file
if [ ! -f "$DB_CRED_FILE" ]; then
    echo "[$(date)] ERROR: Credential file not found: $DB_CRED_FILE" | tee -a "$LOG_FILE"
    exit 1
fi

DB_PASS=$(cat "$DB_CRED_FILE" | tr -d '[:space:]')  # Remove any accidental spaces/newlines

# List of databases to process
DB_NAMES=("QP25PROD" "AU42PROD" "BM23PROD" "CN03PROD" "CN23PROD" "CX21PROD" "DL21PROD" "FF01PROD" "GX21PROD" "MI22PROD" "OA21PROD" "RI22PROD" "TZ21PROD" "QP24PROD" "AU43PROD" "BM24PROD" "CD21PROD" "EB23PROD" "MI21PROD" "TZ23PROD" "WT21PROD" "WT22PROD" "BM41PROD" "AU41PROD" "BM21PROD" "CI01PROD" "GP22PROD" "IN05PROD" "KW21PROD" "RD02PROD" "RD23PROD" "SQ01PROD" "TZ02PROD" "WT23PROD" "RD24PROD" "EB21PROD" "EB37PROD")

# Clear previous log file
> "$LOG_FILE"

# Extract Employee IDs from Column 1 (First Column)
awk '/Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT/ {print $1}' "$EMPLOYEE_FILE_COL1" > "$FILTERED_COL1"

# Extract Employee IDs from Column 2 (Second Column)
awk '/Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT/ {print $2}' "$INPUT_FILE" > "$FILTERED_COL2"

# Validate employee file for column 1
if [ ! -f "$EMPLOYEE_FILE_COL1" ]; then
    echo "[$(date)] ERROR: First column employee file not found: $EMPLOYEE_FILE_COL1" | tee -a "$LOG_FILE"
    exit 1
fi

# Filter first column employees by matching with employee_ids_col1.txt
grep -F -f "$EMPLOYEE_FILE_COL1" "$FILTERED_COL1" > "$FILTERED_COL1.tmp"
mv "$FILTERED_COL1.tmp" "$FILTERED_COL1"

# Merge filtered first and second column employee IDs, remove duplicates
cat "$FILTERED_COL1" "$FILTERED_COL2" | grep -v '^$' | sort | uniq > "$SQL_INPUT_FILE"


[oracle@exa5dbadm01 leavers]$ cat employee_ids_csv.txt
SLAESL  CO72417 2023565 Madhu Kumar     113103 Delivery Integration     Termination
SLAESL  CO72590 2024230 Maqbul  Shaik   371101 Application Services     Termination
SLAESL  CO72983 2025503 Pooja Pawar     113103 Delivery Integration     Termination
SLAESL  CO70166 2013979 Clare Simpson   101555 Migration Hub Contractors        Termination
SLAESL  CO68555 1006074 Chris Clark     113103 IT Testing and Release Management        Termination
[oracle@exa5dbadm01 leavers]$ cat employee_ids.txt
CO66258    Praneeth P            ITS : Solutions Supp ACTION - EMPLOYEE HAS LEFT
CO67240    Annejanette Dickson   Platform Contact Cen ACTION - EMPLOYEE HAS LEFT
[oracle@exa5dbadm01 leavers]$
