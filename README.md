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
    file_path VARCHAR2(500) := '/mount/PRODDBA/oracle_scripts/recert/leavers/test/filtered_ids.txt'; -- File path
BEGIN
    DBMS_OUTPUT.PUT_LINE('--- Starting User Lock Process ---');

    -- Open the file for reading
    file := UTL_FILE.FOPEN('DBA_TEST', 'filtered_ids.txt', 'R');

    -- Loop through the file line by line
    LOOP
        BEGIN
            -- Read employee ID from file
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
                EXIT; -- This should NOT be here. It was causing the script to exit after one user.
            WHEN OTHERS THEN
                DBMS_OUTPUT.PUT_LINE('Error processing Employee ID: ' || v_employee_id || ' - ' || SQLERRM);
        END;
    END LOOP;

    -- Close file
    UTL_FILE.FCLOSE(file);

    DBMS_OUTPUT.PUT_LINE('--- User Lock Process Completed ---');

EXCEPTION
    WHEN OTHERS THEN
        DBMS_OUTPUT.PUT_LINE('Unexpected error: ' || SQLERRM);
        IF UTL_FILE.IS_OPEN(file) THEN
            UTL_FILE.FCLOSE(file);
        END IF;
END;
/
EXIT;
