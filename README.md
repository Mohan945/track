# track
-- Dynamic script to lock users if they exist in the database

DECLARE
    -- Declare a cursor to fetch the usernames that exist in the database
    CURSOR user_cursor IS
        SELECT username
        FROM dba_users
        WHERE username IN ('USER1', 'USER2', 'USER3')  -- Replace with the dynamic list of users or modify as needed
        AND account_status = 'OPEN';  -- Only lock users with 'OPEN' account status
    
    -- Declare a variable to store the username
    v_username dba_users.username%TYPE;
BEGIN
    -- Loop through all the users in the cursor
    FOR user_rec IN user_cursor LOOP
        -- Store the username in the variable
        v_username := user_rec.username;

        -- Dynamically lock the user account
        EXECUTE IMMEDIATE 'ALTER USER ' || v_username || ' ACCOUNT LOCK';
        DBMS_OUTPUT.PUT_LINE('User ' || v_username || ' has been locked.');
    END LOOP;
END;
/
