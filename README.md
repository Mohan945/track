SET DEFINE OFF;
SET SERVEROUTPUT ON SIZE UNLIMITED;

DECLARE
    -- Table to store multiple employee IDs
    TYPE emp_id_table IS TABLE OF VARCHAR2(50);
    emp_ids emp_id_table := emp_id_table('CO70989', 'CO70999'); -- Add more employee IDs

    -- Cursor to fetch users matching each employee ID pattern
    CURSOR user_cursor(emp_id VARCHAR2) IS 
        SELECT username 
        FROM dba_users 
        WHERE username LIKE '%' || emp_id || '%';  -- Find all variations of the employee ID in usernames

    v_username dba_users.username%TYPE;
    v_account_status dba_users.account_status%TYPE;

BEGIN
    -- Loop through each employee ID
    FOR i IN 1..emp_ids.COUNT LOOP
        FOR user_rec IN user_cursor(emp_ids(i)) LOOP
            v_username := user_rec.username;
            
            -- Lock the user account
            EXECUTE IMMEDIATE 'ALTER USER ' || v_username || ' ACCOUNT LOCK';

            -- Short delay to ensure DBA_USERS reflects the update
            DBMS_LOCK.SLEEP(2);

            -- Check if the user is locked
            SELECT account_status 
            INTO v_account_status
            FROM dba_users
            WHERE username = v_username;

            -- Confirm the result
            IF v_account_status IN ('LOCKED', 'EXPIRED & LOCKED', 'LOCKED(TIMED)') THEN
                DBMS_OUTPUT.PUT_LINE('User ' || v_username || ' has been successfully locked.');
            ELSE
                DBMS_OUTPUT.PUT_LINE('Failed to lock user ' || v_username || '. Current Status: ' || v_account_status);
            END IF;

        END LOOP;
    END LOOP;
END;
/
