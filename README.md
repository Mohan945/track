Job&nbsp;name=PROD_EXA_BAU_DB_ARCH_BACKUP_GROUP_C 
Job&nbsp;owner=TPS_ORACLE_SCHEDULER 
Job&nbsp;type=RMAN Script 
Host=exa7dbadm01.corp.standardlife.com 
Target type=Cluster Database 
Target name=BM21GPRO.world 
Categories=Jobs 
Message=Job PROD_EXA_BAU_DB_ARCH_BACKUP_GROUP_C failed. 
Action message=Retry the execution. 
Severity=Critical 
Event reported time=02-Jul-2025 15:26:01 BST 
Target Lifecycle Status=Production 
Comment=M71Operating System=LinuxPlatform=x86_64 Event Type=Job Status Change 
Event name=JobStatus 
Job Status=ErrorExecution Log=Step aborted because agent went down
-----
Successfully established repository database connection.
Validating runtime parameters.
[Info] Target Name: BM21GPRO.world
[Info] Target Type: rac_database
[Info] Job Name: null
[Info] Dont Calculate RCVCAT: false
[Info] Do Blackout: NO
[Info] Backup Strategy: null
[Info] Original Script Name: rman_perl_script
[Info] Modified Script Name: modified_rman_perl_script
Successfully validated runtime parameters.
Checking RMAN repository settings...
Trying to establish a connection to the database BM21GPRO.world:rac_database.
Successfully established the connection to the database BM21GPRO.world:rac_database.
[DEBUG] Initializing RMAN Object
[INFO] Is PDB: false
[INFO] Honor Shared Recovery Catalog Settings: true
[INFO] Honor Local Recovery Catalog Settings: false
[INFO] Use Recovery Catalog (Before validating Enterprise Manager Users access Privilege to the Recovery Catalog Database and Credentials): {0}
[INFO] Use Recovery Catalog (After validating Enterprise Manager Users Access Privilege to the Recovery Catalog Database and Credentials): {0}
[INFO] Database Version: 19.0.0.0.0
[INFO] Database Unique Name: BM21GPRO
INFO: Preferred Connect String .
INFO: Final Connect String (DESCRIPTION = (LOAD_BALANCE = ON)(ADDRESS = (PROTOCOL = TCP)(HOST = exa7-scan1)(PORT = 1521))(CONNECT_DATA = (SERVICE_NAME = BM21GPRO.world))).
[INFO] Media Management Vendor Settings for database BM21GPRO.world:rac_database: null
[INFO] Is Connect String Going To Be Used For RMAN Operation: YES
Updating the database connect string parameter, Value: (DESCRIPTION = (LOAD_BALANCE = ON)(ADDRESS = (PROTOCOL = TCP)(HOST = exa7-scan1)(PORT = 1521))(CONNECT_DATA = (SERVICE_NAME = BM21GPRO.world))).
[INFO] SID and Host Info of Cluster Database Instances: BM21GPRO_1#exa7dbadm01.corp.standardlife.com
[DEBUG] Updating large input parameters in the job
[INFO] Job ID: 00ab26e0acde9dc5e0638b2a14ac23fc
[INFO] Execution ID: 38f24f1e05e886cde0638b2a14ac870d
Obtaining the large input parameter value from the Management Server Repository.
Successfully retrieved the large input parameter from the Management Repository.
Job ID: 00ab26e0acde9dc5e0638b2a14ac23fc
Execution ID: 38f24f1e05e886cde0638b2a14ac870d
Comparing the original script with the modified script.
 Rule Name=SL Production Rule Set,Job Scheduling Ruleset 
Rule Owner=SYSMAN 
Update Details:Job PROD_EXA_BAU_DB_ARCH_BACKUP_GROUP_C failed.
