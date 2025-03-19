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
    log_file UTL_FILE.FILE_TYPE;
    file_path VARCHAR2(500) := '/mnt/nfs/filtered_ids.txt'; -- Update this with the actual NFS path
    log_path VARCHAR2(500) := '/mnt/nfs/lock_users_db.log'; -- Update this with the actual NFS path

BEGIN
    DBMS_OUTPUT.PUT_LINE('--- Starting User Lock Process ---');

    -- Open the input file from NFS mount point
    file := UTL_FILE.FOPEN('NFS_DIR', file_path, 'R');
    log_file := UTL_FILE.FOPEN('NFS_DIR', log_path, 'W');

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

                    -- Log success message
                    UTL_FILE.PUT_LINE(log_file, 'User ' || v_username || ' locked successfully.');
                END LOOP;
            ELSE
                UTL_FILE.PUT_LINE(log_file, 'No matching user found for Employee ID: ' || v_employee_id);
            END IF;
        EXCEPTION
            WHEN NO_DATA_FOUND THEN
                EXIT;
        END;
    END LOOP;

    -- Close files
    UTL_FILE.FCLOSE(file);
    UTL_FILE.FCLOSE(log_file);
    
    DBMS_OUTPUT.PUT_LINE('--- User Lock Process Completed ---');
END;
/
