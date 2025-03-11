SET SERVEROUTPUT ON
DECLARE
    v_count   NUMBER;
    v_db_name VARCHAR2(100);
    v_log     UTL_FILE.FILE_TYPE;
    
    -- List of users to check and lock
    TYPE user_list IS TABLE OF VARCHAR2(100);
    v_users user_list := user_list('USER1', 'USER2', 'USER3'); -- Modify this list as needed

BEGIN
    -- Get the database name
    SELECT name INTO v_db_name FROM v$database;

    -- Open the log file
    v_log := UTL_FILE.FOPEN('LOG_DIR', 'user_lock_log.txt', 'w');

    -- Write header to log file
    UTL_FILE.PUT_LINE(v_log, 'Database Name: ' || v_db_name);
    UTL_FILE.PUT_LINE(v_log, '-----------------------------------');

    -- Loop through users
    FOR i IN 1 .. v_users.COUNT LOOP
        -- Check if the user exists
        SELECT COUNT(*) INTO v_count FROM dba_users WHERE username = UPPER(v_users(i));

        IF v_count > 0 THEN
            -- Lock the user
            EXECUTE IMMEDIATE 'ALTER USER ' || v_users(i) || ' ACCOUNT LOCK';

            -- Log success
            UTL_FILE.PUT_LINE(v_log, 'User: ' || v_users(i) || ' - LOCKED');
            DBMS_OUTPUT.PUT_LINE('User: ' || v_users(i) || ' - LOCKED');
        ELSE
            -- Log user not found
            UTL_FILE.PUT_LINE(v_log, 'User: ' || v_users(i) || ' - NOT FOUND');
            DBMS_OUTPUT.PUT_LINE('User: ' || v_users(i) || ' - NOT FOUND');
        END IF;
    END LOOP;

    -- Close log file
    UTL_FILE.FCLOSE(v_log);

EXCEPTION
    WHEN OTHERS THEN
        -- Handle errors
        DBMS_OUTPUT.PUT_LINE('Error: ' || SQLERRM);
        IF UTL_FILE.IS_OPEN(v_log) THEN
            UTL_FILE.PUT_LINE(v_log, 'Error: ' || SQLERRM);
            UTL_FILE.FCLOSE(v_log);
        END IF;
END;
/
