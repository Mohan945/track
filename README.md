SET SERVEROUTPUT ON;
SET FEEDBACK OFF;
SET VERIFY OFF;
SET HEADING OFF;
SET LINESIZE 300;
SET PAGESIZE 1000;

DECLARE
    v_employee_id VARCHAR2(100) := '&1';
    v_count NUMBER;
    v_username VARCHAR2(100);
BEGIN
    DBMS_OUTPUT.PUT_LINE('--- Processing Employee ID: ' || v_employee_id || ' ---');

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

    DBMS_OUTPUT.PUT_LINE('--- Completed Processing for Employee ID: ' || v_employee_id || ' ---');
END;
/
