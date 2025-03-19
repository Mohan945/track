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
    file := UTL_FILE.FOPEN('DBA_TEST', 'file_path', 'R');

    LOOP
        BEGIN
            UTL_FILE.GET_LINE(file, v_employee_id);

            -- Check if the user exists in DBA_USERS
            SELECT COUNT(*) INTO v_count
            FROM DBA_USERS
            WHERE USERNAME LIKE '%' || v_employee_id || '%'
               OR USERNAME LIKE 'OPS$%' || v_employee_id || '%'
               OR USERNAME LIKE 'DBA%' || v_employee_id || '%'
               OR USERNAME LIKE 'DBD%' || v_employee_id || '%';

            -- If the user exists, lock the account
            IF v_count > 0 THEN
                FOR user_rec IN (SELECT USERNAME FROM DBA_USERS
                                 WHERE USERNAME LIKE '%' || v_employee_id || '%'
                                    OR USERNAME LIKE 'OPS$%' || v_employee_id || '%'
                                    OR USERNAME LIKE 'DBA%' || v_employee_id || '%'
                                    OR USERNAME LIKE 'DBD%' || v_employee_id || '%')
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

--- Starting User Lock Process ---
DECLARE
*
ERROR at line 1:
ORA-29283: invalid file operation: nonexistent file or path [29434]
ORA-06512: at "SYS.UTL_FILE", line 536
ORA-06512: at "SYS.UTL_FILE", line 41
ORA-06512: at "SYS.UTL_FILE", line 478
ORA-06512: at line 12
