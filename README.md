
DECLARE
    CURSOR user_cursor IS 
        SELECT username FROM dba_users 
        WHERE username IN ('C071919', 'C070159', 'C072222');

    v_username dba_users.username%TYPE;
    v_account_status dba_users.account_status%TYPE;
BEGIN
    FOR user_rec IN user_cursor LOOP
        v_username := user_rec.username;

        -- Lock the user account
        EXECUTE IMMEDIATE 'ALTER USER ' || v_username || ' ACCOUNT LOCK';

        -- Check if the user is locked
        SELECT account_status INTO v_account_status 
        FROM dba_users 
        WHERE username = v_username;

        -- Confirm the result
        IF v_account_status LIKE '%LOCK%' THEN
            DBMS_OUTPUT.PUT_LINE('User ' || v_username || ' has been successfully locked.');
        ELSE
            DBMS_OUTPUT.PUT_LINE('Failed to lock user ' || v_username || '.');
        END IF;
    END LOOP;
END;
/
