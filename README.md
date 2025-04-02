[oracle@exa7dbadm01: test]$ vi lock_users.log
[Wed Apr  2 11:38:43 BST 2025] Extracted CONN_STR: EB21
[Wed Apr  2 11:38:43 BST 2025] Extracted ORACLE_HOME: /u01/app/oracle/product/11.2.0.3.28/ebsprod
[Wed Apr  2 11:38:43 BST 2025] Set ORACLE_SID: EB21PROD2

TNS Ping Utility for Linux: Version 11.2.0.3.0 - Production on 02-APR-2025 11:38:43

Copyright (c) 1997, 2011, Oracle.  All rights reserved.

Used parameter files:


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION=(CONNECT_TIMEOUT=5)(TRANSPORT_CONNECT_TIMEOUT=3)(RETRY_COUNT=2)(ADDRESS_LIST=(LOAD_BALANCE=OFF)(FAILOVER=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=obiee_ebs_src_prod.corp.standardlife.com)(PORT=1521))(ADDRESS=(PROTOCOL=TCP)(HOST=exa8-scan1.corp.standardlife.com)(PORT=1521))(ADDRESS=(PROTOCOL=TCP)(HOST=exa7-scan1.corp.standardlife.com)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=EB21PROD.world)))
OK (10 msec)
No matching users found to lock.
No matching users found to lock.
[Wed Apr  2 11:38:43 BST 2025] Successfully processed database: EB21PROD

[Wed Apr  2 11:38:43 BST 2025] Processing database: WT23PROD
[Wed Apr  2 11:38:43 BST 2025] Extracted CONN_STR: WT23
[Wed Apr  2 11:38:43 BST 2025] Extracted ORACLE_HOME: /u01/app/oracle/product/12.2.0.1/dbhome_1
[Wed Apr  2 11:38:43 BST 2025] Set ORACLE_SID:

TNS Ping Utility for Linux: Version 12.2.0.1.0 - Production on 02-APR-2025 11:38:43

Copyright (c) 1997, 2016, Oracle.  All rights reserved.

Used parameter files:
/u01/app/oracle/product/12.2.0.1/dbhome_1/network/admin/sqlnet.ora


Used TNSNAMES adapter to resolve the alias
Attempting to contact (DESCRIPTION=(CONNECT_TIMEOUT=5)(TRANSPORT_CONNECT_TIMEOUT=3)(RETRY_COUNT=2)(ADDRESS_LIST=(LOAD_BALANCE=OFF)(FAILOVER=ON)(ADDRESS=(PROTOCOL=TCP)(HOST=WT23_Watchlist_PROD.corp.standardlife.com)(PORT=1521))(ADDRESS=(PROTOCOL=TCP)(HOST=exa8-scan1.corp.standardlife.com)(PORT=1521))(ADDRESS=(PROTOCOL=TCP)(HOST=exa7-scan1.corp.standardlife.com)(PORT=1521)))(CONNECT_DATA=(SERVER=DEDICATED)(SERVICE_NAME=WT23PROD.world)))
OK (0 msec)
No matching users found to lock.
No matching users found to lock.
[Wed Apr  2 11:38:43 BST 2025] Successfully processed database: WT23PROD

[Wed Apr  2 11:38:43 BST 2025] Locking process completed.

