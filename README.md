[oracle@exa5dbadm01 ~]$ cd /mount/PRODDBA/oracle_scripts/recert/leavers/test
[oracle@exa5dbadm01 test]$ ./main.sh
[Wed Apr  2 08:21:25 BST 2025] Processing database: CI01SYST
find: warning: Unix filenames usually don't contain slashes (though pathnames do).  That means that '-name ‘/u01/app/oracle/product/19.0.0.0/dbhome_7’' will probably evaluate to false all the time on this system.  You might find the '-wholename' test more useful, or perhaps '-samefile'.  Alternatively, if you are using GNU grep, you could use 'find ... -print0 | grep -FzZ ‘/u01/app/oracle/product/19.0.0.0/dbhome_7’'.
[Wed Apr  2 08:21:25 BST 2025] ORACLE_HOME not found for version /u01/app/oracle/product/19.0.0.0/dbhome_7. Skipping database: CI01SYST.
[Wed Apr  2 08:21:25 BST 2025] Processing database: OA21SYST
[Wed Apr  2 08:21:25 BST 2025] Unknown Oracle version for OA21SYST in /mount/PRODDBA/oracle_scripts/recert/leavers/test/all_inst_nonDataGuard_Y. Skipping.
[Wed Apr  2 08:21:25 BST 2025] Processing database: MI22SYST
find: warning: Unix filenames usually don't contain slashes (though pathnames do).  That means that '-name ‘/u01/app/oracle/product/19.0.0.0/dbhome_7’' will probably evaluate to false all the time on this system.  You might find the '-wholename' test more useful, or perhaps '-samefile'.  Alternatively, if you are using GNU grep, you could use 'find ... -print0 | grep -FzZ ‘/u01/app/oracle/product/19.0.0.0/dbhome_7’'.
[Wed Apr  2 08:21:25 BST 2025] ORACLE_HOME not found for version /u01/app/oracle/product/19.0.0.0/dbhome_7. Skipping database: MI22SYST.
[Wed Apr  2 08:21:25 BST 2025] Locking process completed.
[oracle@exa5dbadm01 test]$
