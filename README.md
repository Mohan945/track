SET DEFINE OFF; -- Prevents variable substitution issues
SET SERVEROUTPUT ON SIZE UNLIMITED;

DECLARE
    v_db_name VARCHAR2(100);
    CURSOR user_cursor IS 
        SELECT username FROM dba_users 
        WHERE username IN ('C071919', 'C070159', 'C072222');

    v_username dba_users.username%TYPE;
    v_account_status dba_users.account_status%TYPE;
BEGIN
    -- Get the database name
    SELECT SYS_CONTEXT('USERENV', 'DB_NAME') INTO v_db_name FROM DUAL;

    DBMS_OUTPUT.PUT_LINE('📌 Database Name: ' || v_db_name);
    DBMS_OUTPUT.PUT_LINE('----------------------------------');

    FOR user_rec IN user_cursor LOOP
        v_username := user_rec.username;

        -- Lock the user account
        EXECUTE IMMEDIATE 'ALTER USER ' || v_username || ' ACCOUNT LOCK';

        -- Short delay to ensure DBA_USERS updates
        DBMS_LOCK.SLEEP(2); 

        -- Check if the user is locked
        SELECT account_status INTO v_account_status 
        FROM dba_users 
        WHERE username = v_username;

        -- Confirm the result
        IF v_account_status IN ('LOCKED', 'EXPIRED & LOCKED', 'LOCKED(TIMED)') THEN
            DBMS_OUTPUT.PUT_LINE('✅ [' || v_db_name || '] User ' || v_username || ' has been successfully locked.');
        ELSE
            DBMS_OUTPUT.PUT_LINE('❌ [' || v_db_name || '] Failed to lock user ' || v_username || '. Current Status: ' || v_account_status);
        END IF;
    END LOOP;
END;
/
