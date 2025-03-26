#!/bin/bash

# Define file paths (NFS location for shared access)
INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"   # File containing all employee data
SQL_INPUT_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt"  # Filtered list of Employee IDs
LOG_FILE="/mount/PRODDBA/oracle_scripts/recert/leavers/test/lock_users.log"  # Log file for tracking

# Create/clear log file
> "$LOG_FILE"

# Extract Employee IDs from input file where Details contain specific keywords
grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "$INPUT_FILE" | awk -F'\t' '{print $2}' > "$SQL_INPUT_FILE"

# Check if the SQL input file has Employee IDs
if [ ! -s "$SQL_INPUT_FILE" ]; then
    echo "[$(date)] No matching Employee IDs found. Exiting." | tee -a "$LOG_FILE"
    exit 1
fi

# Execute the SQL script using OEM-provided credentials
sqlplus -s system/"cowboy_1"@CI01SYST  @/mount/PRODDBA/oracle_scripts/recert/leavers/test/test1.sql "$SQL_INPUT_FILE" >> "$LOG_FILE" 2>&1

# Check for errors
if [ $? -eq 0 ]; then
    echo "[$(date)] User locking process completed successfully." | tee -a "$LOG_FILE"
else
    echo "[$(date)] User locking process encountered errors." | tee -a "$LOG_FILE"
fi

SET SERVEROUTPUT ON;
SET FEEDBACK OFF;
SET VERIFY OFF;
SET HEADING OFF;
SET LINESIZE 300;
SET PAGESIZE 1000;

DECLARE
    v_employee_id VARCHAR2(100);
    v_count NUMBER;
    v_username VARCHAR2(100);
    file UTL_FILE.FILE_TYPE;
    file_path VARCHAR2(500) := '/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt';  -- Path to Employee IDs file

BEGIN
    DBMS_OUTPUT.PUT_LINE('--- Starting User Lock Process ---');

    -- Open the Employee IDs file from NFS
    file := UTL_FILE.FOPEN('DBA_TEST', 'filtered_ids.txt', 'R');

    LOOP
        BEGIN
            UTL_FILE.GET_LINE(file, v_employee_id);

            -- Check if the user exists in DBA_USERS
            SELECT COUNT(*) INTO v_count
            FROM DBA_USERS
            WHERE USERNAME LIKE '%' || v_employee_id || '%';

            -- If the user exists, lock the account
            IF v_count > 0 THEN
                FOR user_rec IN (SELECT USERNAME FROM DBA_USERS
                                 WHERE USERNAME LIKE '%' || v_employee_id || '%')
                LOOP
                    v_username := user_rec.USERNAME;

                    -- Lock the Oracle user account
                    EXECUTE IMMEDIATE 'ALTER USER ' || v_username || ' ACCOUNT LOCK';

                    -- Print success message
                    DBMS_OUTPUT.PUT_LINE('User ' || v_username || ' locked successfully.');
                END LOOP;
            ELSE
                DBMS_OUTPUT.PUT_LINE('No matching user found for Employee ID: ' || v_employee_id);
            END IF;
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                EXIT;
        END;
    END LOOP;

    -- Close file
    UTL_FILE.FCLOSE(file);

    DBMS_OUTPUT.PUT_LINE('--- User Lock Process Completed ---');
END;
/
