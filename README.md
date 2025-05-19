[oracle@exa7dbadm01: trace]$ cat /u01/app/oracle/diag/rdbms/ci01gpro/CI01GPRO_1/trace/CI01GPRO_1_ora_221839.trc
Trace file /u01/app/oracle/diag/rdbms/ci01gpro/CI01GPRO_1/trace/CI01GPRO_1_ora_221839.trc
Oracle Database 19c Enterprise Edition Release 19.0.0.0.0 - Production
Version 19.10.0.0.0
Build label:    RDBMS_19.10.0.0.0DBRU_LINUX.X64_201227
ORACLE_HOME:    /u01/app/oracle/product/19.0.0.0/dbhome_3
System name:    Linux
Node name:      exa7dbadm01.corp.standardlife.com
Release:        4.1.12-124.42.4.el7uek.x86_64
Version:        #2 SMP Thu Sep 3 16:14:48 PDT 2020
Machine:        x86_64
Storage:        Exadata
Instance name: CI01GPRO_1
Redo thread mounted by this instance: 1
Oracle process number: 203
Unix process pid: 221839, image: oracle@exa7dbadm01.corp.standardlife.com


*** 2025-05-19T11:35:01.541886+01:00
*** SESSION ID:(403.38882) 2025-05-19T11:35:01.541905+01:00
*** CLIENT ID:() 2025-05-19T11:35:01.541914+01:00
*** SERVICE NAME:(CI01PROD) 2025-05-19T11:35:01.541918+01:00
*** MODULE NAME:(SQL Developer) 2025-05-19T11:35:01.541922+01:00
*** ACTION NAME:() 2025-05-19T11:35:01.541925+01:00
*** CLIENT DRIVER:(jdbcthin) 2025-05-19T11:35:01.541928+01:00

--- PROTOCOL VIOLATION DETECTED ---
----- Dump Cursor sql_id=3m7p1331w3m5h xsc=0x7f3b8d92d988 cur=0x7f3b946b0800 -----

LibraryHandle:  Address=0x71800440 Hash=c3c1ccb0 LockMode=N PinMode=0 LoadLockMode=0 Status=VALD
  ObjectName:  Name=SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :1  and table_name = :2
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :3  and table_name = :4
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_t
    FullHashValue=69c750e79fb349cf399ea118c3c1ccb0 Namespace=SQL AREA(00) Type=CURSOR(00) ContainerId=0 ContainerUid=0 Identifier=3284257968 OwnerIdn=1186
  Statistics:  InvalidationCount=0 ExecutionCount=2 LoadCount=2 ActiveLocks=1 TotalLockCount=2 TotalPinCount=1
  Counters:  BrokenCount=1 RevocablePointer=1 KeepDependency=1 Version=0 BucketInUse=1 HandleInUse=1 HandleReferenceCount=0
  Concurrency:  DependencyMutex=0x718004f0(0, 1, 0, 0) Mutex=0x71800590(0, 26, 0, 0)
  Flags=RON/PIN/TIM/PN0/DBN/[10012841] Flags2=[0000]
  WaitersLists:
    Lock=0x718004d0[0x718004d0,0x718004d0]
    Pin=0x718004b0[0x718004b0,0x718004b0]
    LoadLock=0x71800528[0x71800528,0x71800528]
  Timestamp:  Current=05-19-2025 11:34:08
  HandleReference:  Address=0x71801070 Handle=(nil) Flags=[00]
  LibraryObject:  Address=0xb319e290 HeapMask=0000-0001-0001-0000 Flags=EXS[0000] Flags2=[0000] Flags3=[0000] PublicFlags=[0000]
    ChildTable:  size='16'
      Child:  id='0' Table=0xb319f0f8 Reference=0xb319ec80 Handle=0x725fdaf0
    Children:
      Child:  childNum='0'
        LibraryHandle:  Address=0x725fdaf0 Hash=0 LockMode=N PinMode=S LoadLockMode=0 Status=VALD
          Name:  Namespace=SQL AREA(00) Type=CURSOR(00) ContainerId=0
          Statistics:  InvalidationCount=0 ExecutionCount=2 LoadCount=1 ActiveLocks=1 TotalLockCount=2 TotalPinCount=3
          Counters:  BrokenCount=1 RevocablePointer=1 KeepDependency=0 Version=0 BucketInUse=0 HandleInUse=0 HandleReferenceCount=0
          Concurrency:  DependencyMutex=0x725fdba0(0, 0, 0, 0) Mutex=0x71800590(0, 26, 0, 0)
          Flags=RON/PIN/PN0/EXP/CHD/[10012111] Flags2=[0000]
          WaitersLists:
            Lock=0x725fdb80[0x725fdb80,0x725fdb80]
            Pin=0x725fdb60[0x725fdb60,0x725fdb60]
            LoadLock=0x725fdbd8[0x725fdbd8,0x725fdbd8]
          LibraryObject:  Address=0x69d20288 HeapMask=0000-0001-0001-0000 Flags=EXS[0000] Flags2=[80000] Flags3=[0000] PublicFlags=[0000]
            DataBlocks:
              Block:  #='0' name=KGLH0^c3c1ccb0 pins=0 Change=NONE
                Heap=0xcdb20850 Pointer=0x69d20368 Extent=0x69d201c8 Flags=I/-/P/A/-/-/-
                FreedLocation=0 Alloc=40.539062 Size=43.742188 LoadTime=6274694241
              Block:  #='6' name=SQLA^c3c1ccb0 pins=0 Change=NONE
                Heap=0xb319ea00 Pointer=0xbc544268 Extent=0xbc543690 Flags=I/-/P/A/-/-/E
                FreedLocation=0 Alloc=2746.148438 Size=2753.257812 LoadTime=0
          NamespaceDump:
            Child Cursor:  Heap0=0x69d20368 Heap6=0xbc544268 Heap0 Load Time=05-19-2025 11:34:08 Heap6 Load Time=05-19-2025 11:34:08
  NamespaceDump:
    Parent Cursor:  sql_id=3m7p1331w3m5h parent=0xb319e370 maxchild=1 plk=y ppn=n piflg=82 pflg=10008100 oct=03 psn=1 app(hash)=SQL Developer(3c543292) act(hash)=(0) caller obj#=0 line#=0 prsfcnt=0 obscnt=0    kkscs=0xb319e8b0 nxt=(nil) flg=18 cld=0 hd=0x725fdaf0 par=0xb319e370 pdbuid=0 pdb=0
   Mutex 0xb319e8b0(0, 0) idn 7fff00000000
   ct=0 hsh=0 unp=(nil) unn=0 hvl=b319f190 nhv=0 ses=(nil)
   hep=0xb319e958 flg=80 ld=1 ob=0x69d20288 ptr=0xbc544268 fex=0xbc543690
cursor instantiation=0x7f3b8d92d988 used=1747650901 exec_id=16777217 exec=1
 child#0(0x725fdaf0) pcs=0xb319e8b0
  clk=0x8c8703a0 ci=0x69d20368 pn=0xa3c14440 ctx=0xbc544268
  ctxflg=0x10008310 xfl=0x8002040 yfl=0x604280 xyfl=0x0 xzfl=0x20410002 rtflg=0x2114
 kgsccflg=0x0 llk[0x7f3b8d92d990,0x7f3b8d92d990] idx=0x0
 xscflg=0xc0110676 fl2=0xd000008 fl3=0x42222008 fl4=0x100 fl5=0x800
 xscSecFlg=0x0

----- Bind Info (kkscoacd) -----
 Bind#0
  oacdty=01 mxl=128(64) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=2368 off=0
  kxsbbbfp=7f3b933d1fa0  bln=128  avl=16  flg=05
  value="CUSTOMERPLATFORM"
 Bind#1
  oacdty=01 mxl=128(92) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=128
  kxsbbbfp=7f3b933d2020  bln=128  avl=23  flg=01
  value="CUSTOMERVULNERABILITY_B"
 Bind#2
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=256
  kxsbbbfp=7f3b933d20a0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#3
  oacdty=01 mxl=128(52) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=288
  kxsbbbfp=7f3b933d20c0  bln=128  avl=13  flg=01
  value="RG_ORIGINATOR"
 Bind#4
  oacdty=01 mxl=128(64) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=416
  kxsbbbfp=7f3b933d2140  bln=128  avl=16  flg=01
  value="CUSTOMERPLATFORM"
 Bind#5
  oacdty=01 mxl=128(44) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=544
  kxsbbbfp=7f3b933d21c0  bln=128  avl=11  flg=01
  value="CONTRACTS_A"
 Bind#6
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=672
  kxsbbbfp=7f3b933d2240  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#7
  oacdty=01 mxl=128(52) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=704
  kxsbbbfp=7f3b933d2260  bln=128  avl=13  flg=01
  value="RG_DEPARTMENT"
 Bind#8
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=832
  kxsbbbfp=7f3b933d22e0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#9
  oacdty=01 mxl=128(88) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=864
  kxsbbbfp=7f3b933d2300  bln=128  avl=22  flg=01
  value="RG_DISSATISFIED_REASON"
 Bind#10
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=992
  kxsbbbfp=7f3b933d2380  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#11
  oacdty=01 mxl=128(72) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1024
  kxsbbbfp=7f3b933d23a0  bln=128  avl=18  flg=01
  value="RG_WRAP_UP_OUTCOME"
 Bind#12
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1152
  kxsbbbfp=7f3b933d2420  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#13
  oacdty=01 mxl=128(100) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1184
  kxsbbbfp=7f3b933d2440  bln=128  avl=25  flg=01
  value="RG_WRAP_UP_PRIMARY_DEMAND"
 Bind#14
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1312
  kxsbbbfp=7f3b933d24c0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#15
  oacdty=01 mxl=128(48) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1344
  kxsbbbfp=7f3b933d24e0  bln=128  avl=12  flg=01
  value="RG_REQUESTOR"
 Bind#16
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1472
  kxsbbbfp=7f3b933d2560  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#17
  oacdty=01 mxl=128(84) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1504
  kxsbbbfp=7f3b933d2580  bln=128  avl=21  flg=01
  value="RG_REASON_FOR_CONTACT"
 Bind#18
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1632
  kxsbbbfp=7f3b933d2600  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#19
  oacdty=01 mxl=128(108) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1664
  kxsbbbfp=7f3b933d2620  bln=128  avl=27  flg=01
  value="RG_WRAP_UP_SECONDARY_DEMAND"
 Bind#20
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1792
  kxsbbbfp=7f3b933d26a0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#21
  oacdty=01 mxl=128(112) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1824
  kxsbbbfp=7f3b933d26c0  bln=128  avl=28  flg=01
  value="RG_WRAP_UP_SECONDARY_OUTCOME"
 Bind#22
  oacdty=01 mxl=128(48) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1952
  kxsbbbfp=7f3b933d2740  bln=128  avl=12  flg=01
  value="CIA_IADS_TBL"
 Bind#23
  oacdty=01 mxl=128(76) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2080
  kxsbbbfp=7f3b933d27c0  bln=128  avl=19  flg=01
  value="RPT_RAR_SURVEY_DATA"
 Bind#24
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2208
  kxsbbbfp=7f3b933d2840  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#25
  oacdty=01 mxl=128(40) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2240
  kxsbbbfp=7f3b933d2860  bln=128  avl=10  flg=01
  value="RG_WRAP_UP"
 Incarnation curinc=1 instinc=1 curpdbinc=1 pdbinc=1
 Frames pfr 0x7f3b8d92d930 siz=1962280 efr 0x7f3b92e61368 siz=1962248
 kxscphp=0x7f3b945beb08 siz=15832 inu=13968 nps=8536
 kxscbhp=0x7f3b945bee58 siz=8136 inu=5616 nps=2392
 kxscwhp=0x7f3b945be960 siz=4056 inu=192 nps=0
Starting SQL statement dump
SQL Information
user_id=1186 user_name=SLBR73 module=SQL Developer action=
sql_id=3m7p1331w3m5h plan_hash_value=-1732438384 problem_type=4 command_type=3
----- Current SQL Statement for this session (sql_id=3m7p1331w3m5h) -----
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :1  and table_name = :2
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :3  and table_name = :4
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :5  and table_name = :6
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :7  and table_name = :8
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :9  and table_name = :10
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :11  and table_name = :12
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :13  and table_name = :14
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :15  and table_name = :16
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :17  and table_name = :18
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :19  and table_name = :20
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :21  and table_name = :22
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :23  and table_name = :24
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :25  and table_name = :26
sql_text_length=2670
sql=SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :1  and table_name = :2
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :3  and table_name = :4
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :5  and table_name = :6
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :7  and table_name = :8
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :9  and table_name = :10
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :11  and table_name = :12
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :13  and table_name = :14
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :15  and table_name = :16
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :17  and table_name = :18
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :19  and table_name = :20
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :21  and table_name = :22
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :23  and table_name = :24
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :25  and table_name = :26
Compilation Environment Dump
optimizer_mode_hinted               = false
optimizer_features_hinted           = 0.0.0
parallel_execution_enabled          = false
parallel_query_forced_dop           = 0
parallel_dml_forced_dop             = 0
parallel_ddl_forced_degree          = 0
parallel_ddl_forced_instances       = 0
_query_rewrite_fudge                = 90
optimizer_features_enable           = 11.2.0.3
_optimizer_search_limit             = 5
cpu_count                           = 8
active_instance_count               = 1
parallel_threads_per_cpu            = 1
hash_area_size                      = 131072
bitmap_merge_area_size              = 1048576
sort_area_size                      = 65536
sort_area_retained_size             = 0
_sort_elimination_cost_ratio        = 0
_optimizer_block_size               = 8192
_sort_multiblock_read_count         = 2
_hash_multiblock_io_count           = 0
_db_file_optimizer_read_count       = 128
_optimizer_max_permutations         = 2000
pga_aggregate_target                = 8388608 KB
_pga_max_size                       = 1677720 KB
_query_rewrite_maxdisjunct          = 257
_smm_auto_min_io_size               = 120 KB
_smm_auto_max_io_size               = 1016 KB
_smm_min_size                       = 1024 KB
_smm_max_size_static                = 838860 KB
_smm_px_max_size_static             = 4194304 KB
_cpu_to_io                          = 0
_optimizer_undo_cost_change         = 11.2.0.3
parallel_query_mode                 = enabled
_enable_parallel_dml                = disabled
parallel_ddl_mode                   = enabled
optimizer_mode                      = choose
sqlstat_enabled                     = false
_optimizer_percent_parallel         = 101
_always_anti_join                   = choose
_always_semi_join                   = choose
_optimizer_mode_force               = true
_partition_view_enabled             = true
_always_star_transformation         = false
_query_rewrite_or_error             = false
_hash_join_enabled                  = true
cursor_sharing                      = exact
_b_tree_bitmap_plans                = true
star_transformation_enabled         = false
_optimizer_cost_model               = choose
_new_sort_cost_estimate             = true
_complex_view_merging               = true
_unnest_subquery                    = true
_eliminate_common_subexpr           = true
_pred_move_around                   = true
_convert_set_to_join                = false
_push_join_predicate                = true
_push_join_union_view               = true
_fast_full_scan_enabled             = true
_optim_enhance_nnull_detection      = true
_parallel_broadcast_enabled         = true
_px_broadcast_fudge_factor          = 100
_ordered_nested_loop                = true
_no_or_expansion                    = false
optimizer_index_cost_adj            = 100
optimizer_index_caching             = 0
_system_index_caching               = 0
_disable_datalayer_sampling         = false
query_rewrite_enabled               = true
query_rewrite_integrity             = trusted
_query_cost_rewrite                 = true
_query_rewrite_2                    = true
_query_rewrite_1                    = true
_query_rewrite_expression           = true
_query_rewrite_jgmigrate            = true
_query_rewrite_fpc                  = true
_query_rewrite_drj                  = false
_full_pwise_join_enabled            = true
_partial_pwise_join_enabled         = true
_left_nested_loops_random           = true
_improved_row_length_enabled        = true
_index_join_enabled                 = true
_enable_type_dep_selectivity        = true
_improved_outerjoin_card            = true
_optimizer_adjust_for_nulls         = true
_optimizer_degree                   = 0
_use_column_stats_for_function      = true
_subquery_pruning_enabled           = true
_subquery_pruning_mv_enabled        = false
_or_expand_nvl_predicate            = true
_like_with_bind_as_equality         = false
_table_scan_cost_plus_one           = true
_cost_equality_semi_join            = true
_default_non_equality_sel_check     = true
_new_initial_join_orders            = true
_oneside_colstat_for_equijoins      = true
_optim_peek_user_binds              = true
_minimal_stats_aggregation          = true
_force_temptables_for_gsets         = false
workarea_size_policy                = auto
_smm_auto_cost_enabled              = true
_gs_anti_semi_join_allowed          = true
_optim_new_default_join_sel         = true
optimizer_dynamic_sampling          = 2
_pre_rewrite_push_pred              = true
_optimizer_new_join_card_computation = true
_union_rewrite_for_gs               = yes_gset_mvs
_generalized_pruning_enabled        = true
_optim_adjust_for_part_skews        = true
_force_datefold_trunc               = false
statistics_level                    = typical
_optimizer_system_stats_usage       = true
skip_unusable_indexes               = true
_remove_aggr_subquery               = true
_optimizer_push_down_distinct       = 0
_dml_monitoring_enabled             = true
_optimizer_undo_changes             = false
_predicate_elimination_enabled      = true
_nested_loop_fudge                  = 100
_project_view_columns               = true
_local_communication_costing_enabled = true
_local_communication_ratio          = 50
_query_rewrite_vop_cleanup          = true
_slave_mapping_enabled              = true
_optimizer_cost_based_transformation = linear
_optimizer_mjc_enabled              = true
_right_outer_hash_enable            = true
_spr_push_pred_refspr               = true
_optimizer_cache_stats              = false
_optimizer_cbqt_factor              = 50
_optimizer_squ_bottomup             = true
_fic_area_size                      = 131072
_optimizer_skip_scan_enabled        = true
_optimizer_cost_filter_pred         = false
_optimizer_sortmerge_join_enabled   = true
_optimizer_join_sel_sanity_check    = true
_mmv_query_rewrite_enabled          = true
_bt_mmv_query_rewrite_enabled       = true
_add_stale_mv_to_dependency_list    = true
_distinct_view_unnesting            = false
_optimizer_dim_subq_join_sel        = true
_optimizer_disable_strans_sanity_checks = 0
_optimizer_compute_index_stats      = true
_push_join_union_view2              = true
optimizer_ignore_hints              = false
_optimizer_random_plan              = 0
_query_rewrite_setopgrw_enable      = true
_optimizer_correct_sq_selectivity   = true
_disable_function_based_index       = false
_optimizer_join_order_control       = 3
_optimizer_cartesian_enabled        = true
_optimizer_starplan_enabled         = true
_extended_pruning_enabled           = true
_optimizer_push_pred_cost_based     = true
_optimizer_null_aware_antijoin      = true
_optimizer_extend_jppd_view_types   = true
_sql_model_unfold_forloops          = run_time
_enable_dml_lock_escalation         = false
_bloom_filter_enabled               = true
_update_bji_ipdml_enabled           = 0
_optimizer_extended_cursor_sharing  = udo
_dm_max_shared_pool_pct             = 1
_optimizer_cost_hjsmj_multimatch    = true
_optimizer_transitivity_retain      = true
_px_pwg_enabled                     = true
optimizer_secure_view_merging       = true
_optimizer_join_elimination_enabled = true
flashback_table_rpi                 = non_fbt
_optimizer_cbqt_no_size_restriction = true
_optimizer_enhanced_filter_push     = true
_optimizer_filter_pred_pullup       = true
_rowsrc_trace_level                 = 0
_simple_view_merging                = true
_optimizer_rownum_pred_based_fkr    = true
_optimizer_better_inlist_costing    = all
_optimizer_self_induced_cache_cost  = false
_optimizer_min_cache_blocks         = 10
_optimizer_or_expansion             = depth
_optimizer_order_by_elimination_enabled = true
_optimizer_outer_to_anti_enabled    = true
_selfjoin_mv_duplicates             = true
_dimension_skip_null                = true
_force_rewrite_enable               = false
_optimizer_star_tran_in_with_clause = true
_optimizer_complex_pred_selectivity = true
_optimizer_connect_by_cost_based    = true
_gby_hash_aggregation_enabled       = true
_globalindex_pnum_filter_enabled    = true
_px_minus_intersect                 = true
_fix_control_key                    = -1614931837
_force_slave_mapping_intra_part_loads = false
_force_tmp_segment_loads            = false
_query_mmvrewrite_maxpreds          = 10
_query_mmvrewrite_maxintervals      = 5
_query_mmvrewrite_maxinlists        = 5
_query_mmvrewrite_maxdmaps          = 10
_query_mmvrewrite_maxcmaps          = 20
_query_mmvrewrite_maxregperm        = 512
_query_mmvrewrite_maxqryinlistvals  = 500
_disable_parallel_conventional_load = false
_trace_virtual_columns              = false
_replace_virtual_columns            = true
_virtual_column_overload_allowed    = true
_kdt_buffering                      = true
_first_k_rows_dynamic_proration     = true
_optimizer_sortmerge_join_inequality = true
_optimizer_aw_stats_enabled         = true
_bloom_pruning_enabled              = true
result_cache_mode                   = MANUAL
_px_ual_serial_input                = true
_optimizer_skip_scan_guess          = false
_enable_row_shipping                = true
_row_shipping_threshold             = 100
_row_shipping_explain               = false
transaction_isolation_level         = read_commited
_optimizer_distinct_elimination     = true
_optimizer_multi_level_push_pred    = true
_optimizer_group_by_placement       = true
_optimizer_rownum_bind_default      = 10
_enable_query_rewrite_on_remote_objs = true
_optimizer_extended_cursor_sharing_rel = simple
_optimizer_adaptive_cursor_sharing  = true
_direct_path_insert_features        = 0
_optimizer_improve_selectivity      = true
optimizer_use_pending_statistics    = false
_optimizer_enable_density_improvements = true
_optimizer_aw_join_push_enabled     = true
_optimizer_connect_by_combine_sw    = true
_enable_pmo_ctas                    = 0
_optimizer_native_full_outer_join   = force
_bloom_predicate_enabled            = true
_optimizer_enable_extended_stats    = true
_is_lock_table_for_ddl_wait_lock    = 0
_pivot_implementation_method        = choose
optimizer_capture_sql_plan_baselines = false
optimizer_use_sql_plan_baselines    = true
_optimizer_star_trans_min_cost      = 0
_optimizer_star_trans_min_ratio     = 0
_with_subquery                      = OPTIMIZER
_optimizer_fkr_index_cost_bias      = 10
_optimizer_use_subheap              = true
parallel_degree_policy              = manual
parallel_degree                     = 0
parallel_min_time_threshold         = 10
_parallel_time_unit                 = 10
_optimizer_or_expansion_subheap     = true
_optimizer_free_transformation_heap = true
_optimizer_reuse_cost_annotations   = true
_result_cache_auto_size_threshold   = 100
_result_cache_auto_time_threshold   = 1000
_optimizer_nested_rollup_for_gset   = 100
_nlj_batching_enabled               = 1
parallel_query_default_dop          = 0
is_recur_flags                      = 0
optimizer_use_invisible_indexes     = false
flashback_data_archive_internal_cursor = 0
_optimizer_extended_stats_usage_control = 192
_parallel_syspls_obey_force         = true
cell_offload_processing             = true
_rdbms_internal_fplib_enabled       = false
db_file_multiblock_read_count       = 128
_bloom_folding_enabled              = true
_mv_generalized_oj_refresh_opt      = true
cell_offload_compaction             = ADAPTIVE
cell_offload_plan_display           = AUTO
_bloom_predicate_offload            = true
_bloom_filter_size                  = 0
_bloom_pushing_max                  = 512
parallel_degree_limit               = 65535
parallel_force_local                = false
parallel_max_degree                 = 8
total_cpu_count                     = 8
_optimizer_coalesce_subqueries      = true
_optimizer_fast_pred_transitivity   = true
_optimizer_fast_access_pred_analysis = true
_optimizer_unnest_disjunctive_subq  = true
_optimizer_unnest_corr_set_subq     = true
_optimizer_distinct_agg_transform   = true
_aggregation_optimization_settings  = 0
_optimizer_connect_by_elim_dups     = true
_optimizer_eliminate_filtering_join = true
_connect_by_use_union_all           = true
dst_upgrade_insert_conv             = true
advanced_queuing_internal_cursor    = 0
_optimizer_unnest_all_subqueries    = true
parallel_autodop                    = 0
parallel_ddldml                     = 0
_parallel_cluster_cache_policy      = adaptive
_parallel_scalability               = 50
iot_internal_cursor                 = 0
_optimizer_instance_count           = 0
_optimizer_connect_by_cb_whr_only   = false
_suppress_scn_chk_for_cqn           = nosuppress_1466
_optimizer_join_factorization       = true
_optimizer_use_cbqt_star_transformation = true
_optimizer_table_expansion          = true
_and_pruning_enabled                = true
_deferred_constant_folding_mode     = DEFAULT
_optimizer_distinct_placement       = true
partition_pruning_internal_cursor   = 0
parallel_hinted                     = none
_sql_compatibility                  = 0
_optimizer_use_feedback             = true
_optimizer_try_st_before_jppd       = true
_dml_frequency_tracking             = false
_optimizer_interleave_jppd          = true
kkb_drop_empty_segments             = 0
_px_partition_scan_enabled          = true
_px_partition_scan_threshold        = 64
_optimizer_false_filter_pred_pullup = true
_bloom_minmax_enabled               = true
only_move_row                       = 0
_optimizer_enable_table_lookup_by_nl = true
parallel_execution_message_size     = 16384
_px_loc_msg_cost                    = 1000
_px_net_msg_cost                    = 10000
_optimizer_generate_transitive_pred = true
_optimizer_cube_join_enabled        = false
_optimizer_filter_pushdown          = true
deferred_segment_creation           = true
_optimizer_outer_join_to_inner      = true
_allow_level_without_connect_by     = false
_max_rwgs_groupings                 = 8192
_optimizer_hybrid_fpwj_enabled      = false
_px_replication_enabled             = false
ilm_filter                          = 0
_optimizer_partial_join_eval        = false
_px_concurrent                      = false
_px_object_sampling_enabled         = false
_px_back_to_parallel                = OFF
_optimizer_unnest_scalar_sq         = false
_optimizer_full_outer_join_to_outer = true
_px_filter_parallelized             = false
_px_filter_skew_handling            = false
_zonemap_use_enabled                = true
_zonemap_control                    = 0
_optimizer_multi_table_outerjoin    = false
_px_groupby_pushdown                = choose
_partition_advisor_srs_active       = true
_optimizer_ansi_join_lateral_enhance = false
_px_parallelize_expression          = false
_fast_index_maintenance             = true
_optimizer_ansi_rearchitecture      = false
_optimizer_gather_stats_on_load     = false
ilm_access_tracking                 = 0
ilm_dml_timestamp                   = 0
_px_adaptive_dist_method            = off
_px_adaptive_dist_method_threshold  = 0
_optimizer_batch_table_access_by_rowid = false
optimizer_adaptive_reporting_only   = false
_optimizer_ads_max_table_count      = 0
_optimizer_ads_time_limit           = 0
_optimizer_ads_use_result_cache     = true
_px_wif_dfo_declumping              = off
_px_wif_extend_distribution_keys    = false
_px_join_skew_handling              = false
_px_join_skew_ratio                 = 10
_px_join_skew_minfreq               = 30
CLI_internal_cursor                 = 0
_parallel_fault_tolerance_enabled   = false
_parallel_fault_tolerance_threshold = 3
_px_partial_rollup_pushdown         = off
_px_single_server_enabled           = false
_optimizer_dsdir_usage_control      = 0
_px_cpu_autodop_enabled             = false
_px_cpu_process_bandwidth           = 200
_sql_hvshare_threshold              = 0
_px_tq_rowhvs                       = true
_optimizer_use_gtt_session_stats    = false
_optimizer_proc_rate_level          = off
_px_hybrid_TSM_HWMB_load            = true
_optimizer_use_histograms           = true
PMO_altidx_rebuild                  = 0
_cell_offload_expressions           = true
_cell_materialize_virtual_columns   = true
_cell_materialize_all_expressions   = false
_rowsets_enabled                    = true
_rowsets_target_maxsize             = 524288
_rowsets_max_rows                   = 256
_use_hidden_partitions              = 0
_px_load_monitor_threshold          = 10000
_px_numa_support_enabled            = true
total_processor_group_count         = 2
_adaptive_window_consolidator_enabled = false
_optimizer_strans_adaptive_pruning  = false
_bloom_rm_filter                    = false
_optimizer_null_accepting_semijoin  = false
_long_varchar_allow_IOT             = 0
_parallel_ctas_enabled              = true
_cell_offload_complex_processing    = true
_optimizer_performance_feedback     = off
_optimizer_proc_rate_source         = DEFAULT
_hashops_prefetch_size              = 4
_cell_offload_sys_context           = true
_multi_commit_global_index_maint    = 0
_stat_aggs_one_pass_algorithm       = false
_dbg_scan                           = 0
_oltp_comp_dbg_scan                 = 0
_arch_comp_dbg_scan                 = 0
_optimizer_gather_feedback          = true
_upddel_dba_hash_mask_bits          = 0
_px_pwmr_enabled                    = true
_px_cdb_view_enabled                = true
_bloom_sm_enabled                   = true
_optimizer_cluster_by_rowid         = false
_optimizer_cluster_by_rowid_control = 3
_partition_cdb_view_enabled         = true
_common_data_view_enabled           = true
_pred_push_cdb_view_enabled         = true
_rowsets_cdb_view_enabled           = true
_distinct_agg_optimization_gsets    = off
_array_cdb_view_enabled             = true
_optimizer_adaptive_plan_control    = 0
_optimizer_adaptive_random_seed     = 0
_optimizer_cbqt_or_expansion        = off
_inmemory_dbg_scan                  = 0
_gby_vector_aggregation_enabled     = false
_optimizer_vector_transformation    = false
_optimizer_vector_fact_dim_ratio    = 10
_key_vector_max_size                = 0
_key_vector_predicate_enabled       = true
_key_vector_predicate_threshold     = 0
_vector_operations_control          = 0
_optimizer_vector_min_fact_rows     = 10000000
parallel_dblink                     = 0
_px_scalable_invdist                = false
_key_vector_offload                 = predicate
_optimizer_aggr_groupby_elim        = false
_optimizer_reduce_groupby_key       = false
_vector_serialize_temp_threshold    = 1000
_always_vector_transformation       = false
_optimizer_cluster_by_rowid_batched = false
_object_link_fixed_enabled          = true
optimizer_inmemory_aware            = true
_optimizer_inmemory_table_expansion = false
_optimizer_inmemory_gen_pushable_preds = false
_optimizer_inmemory_autodop         = false
_optimizer_inmemory_access_path     = false
_optimizer_inmemory_bloom_filter    = false
_parallel_inmemory_min_time_threshold = 1
_parallel_inmemory_time_unit        = 1
_rc_sys_obj_enabled                 = true
_optimizer_nlj_hj_adaptive_join     = false
_indexable_con_id                   = true
_bloom_serial_filter                = off
inmemory_force                      = default
inmemory_query                      = enable
_inmemory_query_scan                = true
_inmemory_query_fetch_by_rowid      = false
_inmemory_pruning                   = on
_px_autodop_pq_overhead             = true
_px_external_table_default_stats    = false
_optimizer_key_vector_aggr_factor   = 75
_optimizer_vector_cost_adj          = 100
_cdb_cross_container                = 65535
_cdb_view_parallel_degree           = 65535
_optimizer_hll_entry                = 4096
_px_cdb_view_join_enabled           = true
inmemory_size                       = 0
_external_table_smart_scan          = hadoop_only
_optimizer_inmemory_minmax_pruning  = false
_approx_cnt_distinct_gby_pushdown   = choose
_approx_cnt_distinct_optimization   = 0
_optimizer_ads_use_partial_results  = false
_query_execution_time_limit         = 0
_optimizer_inmemory_cluster_aware_dop = false
_optimizer_db_blocks_buffers        = 0 KB
_query_rewrite_use_on_query_computation = false
_px_scalable_invdist_mcol           = false
_optimizer_bushy_join               = off
_optimizer_bushy_fact_dim_ratio     = 20
_optimizer_bushy_fact_min_size      = 100000
_optimizer_bushy_cost_factor        = 100
_rmt_for_table_redef_enabled        = true
_optimizer_eliminate_subquery       = false
_sqlexec_cache_aware_hash_join_enabled = true
_zonemap_usage_tracking             = true
_sqlexec_hash_based_distagg_enabled = false
_sqlexec_disable_hash_based_distagg_tiv = false
_sqlexec_hash_based_distagg_ssf_enabled = false
_sqlexec_distagg_optimization_settings = 0
approx_for_aggregation              = false
approx_for_count_distinct           = false
_optimizer_union_all_gsets          = false
_rowsets_use_encoding               = true
_rowsets_max_enc_rows               = 64
_optimizer_enhanced_join_elimination = false
_optimizer_multicol_join_elimination = false
_key_vector_create_pushdown_threshold = 0
_optimizer_enable_plsql_stats       = false
_recursive_with_parallel            = false
_recursive_with_branch_iterations   = 1
_px_dist_agg_partial_rollup_pushdown = off
_px_adaptive_dist_bypass_enabled    = true
_enable_view_pdb                    = true
_optimizer_key_vector_pruning_enabled = false
_pwise_distinct_enabled             = false
_recursive_with_using_temp_table    = false
_partition_read_only                = true
_sql_alias_scope                    = true
_cdb_view_no_skip_migrate           = false
_approx_perc_sampling_err_rate      = 2
_sqlexec_encoding_aware_hj_enabled  = true
rim_node_exist                      = 0
_enable_containers_subquery         = true
_force_containers_subquery          = false
_cell_offload_vector_groupby        = true
_vector_encoding_mode               = off
_ds_xt_split_count                  = 0
_ds_sampling_method                 = NO_QUALITY_METRIC
_optimizer_ads_use_spd_cache        = false
_optimizer_use_table_scanrate       = OFF
_optimizer_use_xt_rowid             = false
_xt_sampling_scan_granules          = off
_recursive_with_control             = 0
_sqlexec_use_rwo_aware_expr_analysis = true
_optimizer_band_join_aware          = false
_optimizer_vector_base_dim_fact_factor = 0
approx_for_percentile               = none
_approx_percentile_optimization     = 0
_projection_pushdown                = true
_px_object_sampling                 = 200
_optimizer_adaptive_plans_continuous = false
_optimizer_adaptive_plans_iterative = false
_ds_enable_view_sampling            = false
_optimizer_generate_ptf_implied_preds = true
_optimizer_inmemory_use_stored_stats = NEVER
_cdb_special_old_xplan              = true
uniaud_internal_cursor              = 0
_kd_dbg_control                     = 0
_mv_access_compute_fresh_data       = off
_bloom_filter_ratio                 = 30
_optimizer_control_shard_qry_processing = 65529
containers_parallel_degree          = 65535
sql_from_coordinator                = 0
_xt_sampling_scan_granules_min_granules = 1
_in_memory_cdt                      = LIMITED
_in_memory_cdt_maxpx                = 4
_px_partition_load_dist_threshold   = 64
_px_adaptive_dist_bypass_optimization = 1
_optimizer_interleave_or_expansion  = false
optimizer_adaptive_plans            = true
optimizer_adaptive_statistics       = false
_optimizer_use_feedback_for_join    = false
_optimizer_ads_for_pq               = false
_px_join_skewed_values_count        = 0
optimizer_ignore_parallel_hints     = false
parallel_min_degree                 = 1
_px_nlj_bcast_rr_threshold          = 65535
_optimizer_gather_stats_on_load_all = false
_optimizer_gather_stats_on_load_hist = false
_optimizer_allow_all_access_paths   = true
_key_vector_double_enabled          = false
_key_vector_timestamp_enabled       = false
_optimizer_answering_query_using_stats = false
_disable_dblink_optim               = true
_cell_offload_hybrid_processing     = true
_read_optimized_table_lookup        = true
_optimizer_key_vector_payload       = false
_optimizer_vector_fact_payload_ratio = 20
_bloom_pruning_setops_enabled       = false
_bloom_filter_setops_enabled        = false
_key_vector_shared_dgk_ht           = true
_px_pwise_wif_enabled               = false
_sqlexec_reorder_wif_enabled        = false
_px_partition_skew_threshold        = 0
_sqlexec_pwiseops_with_sqlfuncs_enabled = false
_sqlexec_pwiseops_with_binds_enabled = false
_px_shared_hash_join                = false
_px_reuse_server_groups             = multi
_px_join_skew_null_handling         = false
_px_join_skew_use_histogram         = false
_px_join_skew_sampling_time_limit   = 0
_px_join_skew_sampling_percent      = 0
_cdb_view_no_skip_restricted        = false
_inmemory_external_table            = true
_sqlexec_use_kgghash3               = true
_cell_offload_vector_groupby_force  = false
_hcs_enable_pred_push               = false
parallel_dop_doubled                = 0
_optimizer_gather_stats_on_load_index = false
_con_map_sql_enforcement            = true
_uniq_cons_sql_enforcement          = true
_ref_cons_sql_enforcement           = true
_is_lock_table_for_split_merge      = 0
_cell_offload_vector_groupby_fact_key = false
_px_scalable_gby_invdist            = false
_px_dynamic_granules                = false
_px_dynamic_granules_adjust         = 0
_px_hybrid_partition_execution_enabled = false
_px_hybrid_partition_skew_threshold = 255
_optimizer_key_vector_use_max_size  = 1048576
_cell_offload_vector_groupby_withnojoin = false
_key_vector_join_pushdown_enabled   = false
_cell_offload_grand_total           = false
_optimizer_key_vector_payload_dim_aggs = false
_optimizer_use_auto_indexes         = OFF
_optimizer_use_stats_on_conventional_dml = false
_optimizer_gather_stats_on_conventional_dml = false
_optimizer_use_stats_on_conventional_config = 0
_skip_pset_col_chk                  = 0
_nls_binary                         = true
_optimizer_quarantine_sql           = false
_optimizer_gather_stats_on_conventional_config = 0
_nls_forced_binary                  = 0
_utl32k_mv_query                    = false
optimizer_session_type              = NORMAL
BlockChain_ledger_infrastructure    = 0
_cell_offload_vector_groupby_external = true
_bigdata_offload_flag               = false
container_data                      = ALL
optimizer_real_time_statistics      = false

Fine-Grained Invalidation state dump:
fine-grained dependencies:
  [0] typ=6 (INDEX TBL), objn=425, pnum=[0,0], flags=0x00000000
  [1] typ=1 (INDEX), objn=427, pnum=[0,0], flags=0x00000000
  [2] typ=1 (INDEX), objn=1260008, pnum=[0,0], flags=0x00000000
  [3] typ=1 (INDEX), objn=1260010, pnum=[0,0], flags=0x00000000
  [4] typ=1 (INDEX), objn=1260049, pnum=[0,0], flags=0x00000000
  [5] typ=1 (INDEX), objn=1260006, pnum=[0,0], flags=0x00000000
  [6] typ=6 (INDEX TBL), objn=61, pnum=[0,0], flags=0x00000000
  [7] typ=1 (INDEX), objn=62, pnum=[0,0], flags=0x00000000
  [8] typ=6 (INDEX TBL), objn=1260017, pnum=[0,0], flags=0x00000000
  [9] typ=1 (INDEX), objn=1260050, pnum=[0,0], flags=0x00000000
  [10] typ=6 (INDEX TBL), objn=1260005, pnum=[0,0], flags=0x00000000
  [11] typ=1 (INDEX), objn=1260009, pnum=[0,0], flags=0x00000000
cursor refresh state:
  maCtxInfo_ctxdef=0x6c674570
  papRefresh: infoMut=0, cursor=0
  tidRefresh: infoMut=0, cursor=0
  FGIRolling: infoMut=0, cursor=0
    FGIValid: infoMut=0, cursor=0
  refreshCnt:  ctxmut=0, cursor=0
       valid: 1
End of Fine-Grained Invalidation state dump
====================== END SQL Statement Dump ======================
2025-05-19T11:35:01.631105+01:00
Incident 242011 created, dump file: /u01/app/oracle/diag/rdbms/ci01gpro/CI01GPRO_1/incident/incdir_242011/CI01GPRO_1_ora_221839_i242011.trc
ORA-03137: malformed TTC packet from client rejected: [3146] [4] [] [] [] [] [] []


*** 2025-05-19T11:35:33.319026+01:00
2025-05-19T11:35:33.319005+01:00
Incident 242012 created, dump file: /u01/app/oracle/diag/rdbms/ci01gpro/CI01GPRO_1/incident/incdir_242012/CI01GPRO_1_ora_221839_i242012.trc
ORA-03137: malformed TTC packet from client rejected: [3146] [94] [] [] [] [] [] []


*** 2025-05-19T14:09:05.612689+01:00
OSSIPC:SKGXP:[19863f80.1192]{0}: (221839 <- 32238)SKGXPDOAINVALCON: connection 0x19874c20 admno 0x47743e3a scoono 0x55801581 acconn 0x1690e5da getting closed. inactive: 0
Surviving request 0x198c6480
Device Reopen async completion request 0x198cb570, device handle 0x19879820, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1987a050, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198833b0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19887d80, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198885b0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19889610, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19889e40, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988aea0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988dfc0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1989fb50, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198ad640, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198b12b0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198b8b90, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198bc800, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d3cf0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d4d50, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d5db0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d65e0, error code 0
OSSIPC:SKGXP:[19863f80.1209]{0}: (221839 <- 39277)SKGXPDOAINVALCON: connection 0x1987ba10 admno 0x47743e3a scoono 0x55801582 acconn 0x300aaf46 getting closed. inactive: 0
Surviving request 0x198c6480
Device Reopen async completion request 0x198cb570, device handle 0x19882350, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19883be0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988a670, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988b6d0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988bf00, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988cf60, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988d790, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988e7f0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198908e0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1989f320, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198a0380, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198a0bb0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198a5d60, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198c0470, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d2c90, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d34c0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d4520, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d5580, error code 0
OSSIPC:SKGXP:[19863f80.1227]{0}: (221839 <- 43940)SKGXPDOAINVALCON: connection 0x19869c80 admno 0x47743e3a scoono 0x5580157a acconn 0x321cd63b getting closed. inactive: 1
Surviving request 0x198c6480
Device Reopen async completion request 0x198cb570, device handle 0x198705c0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19871d50, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1987a880, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19882b80, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19884410, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19884c40, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19887550, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x19888de0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x1988c730, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198a20f0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198a99d0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198b4f20, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198c40e0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198c70e0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198ca0e0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198cdfd0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d0fd0, error code 0
Device Reopen async completion request 0x198cb570, device handle 0x198d2460, error code 0
OSSIPC:SKGXP:[19863f80.1245]{0}: (221839 <- 32238)SKGXPDOAINVALCON: connection 0x19895460 admno 0x47743e3a scoono 0x55801594 acconn 0x3c1d5a7e getting closed. inactive: 0

*** 2025-05-19T15:18:31.341241+01:00
Surviving request 0x198cb570
Device Reopen async completion request 0x19891ed0, device handle 0x19879820, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1987a050, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198833b0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19887d80, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198885b0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19889610, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19889e40, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988aea0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988dfc0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1989fb50, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198ad640, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198b12b0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198b8b90, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198bc800, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d3cf0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d4d50, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d5db0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d65e0, error code 0
OSSIPC:SKGXP:[19863f80.1263]{0}: (221839 <- 39277)SKGXPDOAINVALCON: connection 0x19876960 admno 0x47743e3a scoono 0x558015a3 acconn 0x404722b3 getting closed. inactive: 0
Surviving request 0x198cb570
Device Reopen async completion request 0x19891ed0, device handle 0x19882350, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19883be0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988a670, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988b6d0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988bf00, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988cf60, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988d790, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988e7f0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198908e0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1989f320, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198a0380, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198a0bb0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198a5d60, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198c0470, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d2c90, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d34c0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d4520, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d5580, error code 0
OSSIPC:SKGXP:[19863f80.1282]{0}: (221839 <- 43940)SKGXPDOAINVALCON: connection 0x1989a6b0 admno 0x47743e3a scoono 0x558015a5 acconn 0x29b7b42f getting closed. inactive: 0
Surviving request 0x198cb570
Device Reopen async completion request 0x19891ed0, device handle 0x198705c0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19871d50, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1987a880, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19882b80, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19884410, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19884c40, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19887550, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x19888de0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x1988c730, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198a20f0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198a99d0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198b4f20, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198c40e0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198c70e0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198ca0e0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198cdfd0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d0fd0, error code 0
Device Reopen async completion request 0x19891ed0, device handle 0x198d2460, error code 0
--- PROTOCOL VIOLATION DETECTED ---
----- Dump Cursor sql_id=4vd4x866hyjh3 xsc=0x7f3b8f7a1328 cur=0x7f3b946b0800 -----

LibraryHandle:  Address=0x848ec920 Hash=8d0f4603 LockMode=N PinMode=0 LoadLockMode=0 Status=VALD
  ObjectName:  Name=SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :1  and table_name = :2
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :3  and table_name = :4
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_t
    FullHashValue=fb836c344e5b00654db49d418d0f4603 Namespace=SQL AREA(00) Type=CURSOR(00) ContainerId=0 ContainerUid=0 Identifier=2366588419 OwnerIdn=1186
  Statistics:  InvalidationCount=0 ExecutionCount=1 LoadCount=2 ActiveLocks=1 TotalLockCount=1 TotalPinCount=1
  Counters:  BrokenCount=1 RevocablePointer=1 KeepDependency=1 Version=0 BucketInUse=0 HandleInUse=0 HandleReferenceCount=0
  Concurrency:  DependencyMutex=0x848ec9d0(0, 1, 0, 0) Mutex=0x848eca70(0, 20, 0, 0)
  Flags=RON/PIN/TIM/PN0/DBN/[10012841] Flags2=[0000]
  WaitersLists:
    Lock=0x848ec9b0[0x848ec9b0,0x848ec9b0]
    Pin=0x848ec990[0x848ec990,0x848ec990]
    LoadLock=0x848eca08[0x848eca08,0x848eca08]
  Timestamp:  Current=05-19-2025 15:18:31
  HandleReference:  Address=0x848ed620 Handle=(nil) Flags=[00]
  LibraryObject:  Address=0xc9cbfa50 HeapMask=0000-0001-0001-0000 Flags=EXS[0000] Flags2=[0000] Flags3=[0000] PublicFlags=[0000]
    ChildTable:  size='16'
      Child:  id='0' Table=0xc9cc08b8 Reference=0xc9cc0440 Handle=0x7221bb18
    Children:
      Child:  childNum='0'
        LibraryHandle:  Address=0x7221bb18 Hash=0 LockMode=N PinMode=S LoadLockMode=0 Status=VALD
          Name:  Namespace=SQL AREA(00) Type=CURSOR(00) ContainerId=0
          Statistics:  InvalidationCount=0 ExecutionCount=1 LoadCount=1 ActiveLocks=1 TotalLockCount=1 TotalPinCount=2
          Counters:  BrokenCount=1 RevocablePointer=1 KeepDependency=0 Version=0 BucketInUse=0 HandleInUse=0 HandleReferenceCount=0
          Concurrency:  DependencyMutex=0x7221bbc8(0, 0, 0, 0) Mutex=0x848eca70(0, 20, 0, 0)
          Flags=RON/PIN/PN0/EXP/CHD/[10012111] Flags2=[0000]
          WaitersLists:
            Lock=0x7221bba8[0x7221bba8,0x7221bba8]
            Pin=0x7221bb88[0x7221bb88,0x7221bb88]
            LoadLock=0x7221bc00[0x7221bc00,0x7221bc00]
          LibraryObject:  Address=0xc9653810 HeapMask=0000-0001-0001-0000 Flags=EXS[0000] Flags2=[80000] Flags3=[0000] PublicFlags=[0000]
            DataBlocks:
              Block:  #='0' name=KGLH0^8d0f4603 pins=0 Change=NONE
                Heap=0xca6f8838 Pointer=0xc96538f0 Extent=0xc9653750 Flags=I/-/P/A/-/-/-
                FreedLocation=0 Alloc=42.921875 Size=43.742188 LoadTime=6288156846
              Block:  #='6' name=SQLA^8d0f4603 pins=0 Change=NONE
                Heap=0xc9cc01c0 Pointer=0x945aed00 Extent=0x945ae128 Flags=I/-/P/A/-/-/E
                FreedLocation=0 Alloc=2956.867188 Size=2962.851562 LoadTime=0
          NamespaceDump:
            Child Cursor:  Heap0=0xc96538f0 Heap6=0x945aed00 Heap0 Load Time=05-19-2025 15:18:31 Heap6 Load Time=05-19-2025 15:18:31
  NamespaceDump:
    Parent Cursor:  sql_id=4vd4x866hyjh3 parent=0xc9cbfb30 maxchild=1 plk=y ppn=n piflg=82 pflg=10008100 oct=03 psn=1 app(hash)=SQL Developer(3c543292) act(hash)=(0) caller obj#=0 line#=0 prsfcnt=0 obscnt=0    kkscs=0xc9cc0070 nxt=(nil) flg=18 cld=0 hd=0x7221bb18 par=0xc9cbfb30 pdbuid=0 pdb=0
   Mutex 0xc9cc0070(0, 0) idn 7fff00000000
   ct=0 hsh=0 unp=(nil) unn=0 hvl=c9cc0950 nhv=0 ses=(nil)
   hep=0xc9cc0118 flg=80 ld=1 ob=0xc9653810 ptr=0x945aed00 fex=0x945ae128
cursor instantiation=0x7f3b8f7a1328 used=1747664311 exec_id=16777216 exec=1
 child#0(0x7221bb18) pcs=0xc9cc0070
  clk=0xadc72428 ci=0xc96538f0 pn=0xb3c1b218 ctx=0x945aed00
  ctxflg=0x10008310 xfl=0x8002040 yfl=0x604280 xyfl=0x0 xzfl=0x20410002 rtflg=0x2114
 kgsccflg=0x0 llk[0x7f3b8f7a1330,0x7f3b8f7a1330] idx=0x0
 xscflg=0xc0110676 fl2=0x1d000008 fl3=0x42222008 fl4=0x100 fl5=0x100800
 xscSecFlg=0x0

----- Bind Info (kkscoacd) -----
 Bind#0
  oacdty=01 mxl=128(64) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=2528 off=0
  kxsbbbfp=7f3b946ad620  bln=128  avl=16  flg=05
  value="CUSTOMERPLATFORM"
 Bind#1
  oacdty=01 mxl=128(92) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=128
  kxsbbbfp=7f3b946ad6a0  bln=128  avl=23  flg=01
  value="CUSTOMERVULNERABILITY_B"
 Bind#2
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=256
  kxsbbbfp=7f3b946ad720  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#3
  oacdty=01 mxl=128(52) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=288
  kxsbbbfp=7f3b946ad740  bln=128  avl=13  flg=01
  value="RG_ORIGINATOR"
 Bind#4
  oacdty=01 mxl=128(64) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=416
  kxsbbbfp=7f3b946ad7c0  bln=128  avl=16  flg=01
  value="CUSTOMERPLATFORM"
 Bind#5
  oacdty=01 mxl=128(44) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=544
  kxsbbbfp=7f3b946ad840  bln=128  avl=11  flg=01
  value="CONTRACTS_A"
 Bind#6
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=672
  kxsbbbfp=7f3b946ad8c0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#7
  oacdty=01 mxl=128(52) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=704
  kxsbbbfp=7f3b946ad8e0  bln=128  avl=13  flg=01
  value="RG_DEPARTMENT"
 Bind#8
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=832
  kxsbbbfp=7f3b946ad960  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#9
  oacdty=01 mxl=128(88) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=864
  kxsbbbfp=7f3b946ad980  bln=128  avl=22  flg=01
  value="RG_DISSATISFIED_REASON"
 Bind#10
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=992
  kxsbbbfp=7f3b946ada00  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#11
  oacdty=01 mxl=128(92) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1024
  kxsbbbfp=7f3b946ada20  bln=128  avl=23  flg=01
  value="RG_DISSATISFIED_OUTCOME"
 Bind#12
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1152
  kxsbbbfp=7f3b946adaa0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#13
  oacdty=01 mxl=128(72) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1184
  kxsbbbfp=7f3b946adac0  bln=128  avl=18  flg=01
  value="RG_WRAP_UP_OUTCOME"
 Bind#14
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1312
  kxsbbbfp=7f3b946adb40  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#15
  oacdty=01 mxl=128(100) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1344
  kxsbbbfp=7f3b946adb60  bln=128  avl=25  flg=01
  value="RG_WRAP_UP_PRIMARY_DEMAND"
 Bind#16
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1472
  kxsbbbfp=7f3b946adbe0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#17
  oacdty=01 mxl=128(48) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1504
  kxsbbbfp=7f3b946adc00  bln=128  avl=12  flg=01
  value="RG_REQUESTOR"
 Bind#18
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1632
  kxsbbbfp=7f3b946adc80  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#19
  oacdty=01 mxl=128(84) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1664
  kxsbbbfp=7f3b946adca0  bln=128  avl=21  flg=01
  value="RG_REASON_FOR_CONTACT"
 Bind#20
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1792
  kxsbbbfp=7f3b946add20  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#21
  oacdty=01 mxl=128(108) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1824
  kxsbbbfp=7f3b946add40  bln=128  avl=27  flg=01
  value="RG_WRAP_UP_SECONDARY_DEMAND"
 Bind#22
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1952
  kxsbbbfp=7f3b946addc0  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#23
  oacdty=01 mxl=128(112) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=1984
  kxsbbbfp=7f3b946adde0  bln=128  avl=28  flg=01
  value="RG_WRAP_UP_SECONDARY_OUTCOME"
 Bind#24
  oacdty=01 mxl=128(48) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2112
  kxsbbbfp=7f3b946ade60  bln=128  avl=12  flg=01
  value="CIA_IADS_TBL"
 Bind#25
  oacdty=01 mxl=128(76) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2240
  kxsbbbfp=7f3b946adee0  bln=128  avl=19  flg=01
  value="RPT_RAR_SURVEY_DATA"
 Bind#26
  oacdty=01 mxl=32(20) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2368
  kxsbbbfp=7f3b946adf60  bln=32  avl=05  flg=01
  value="RGTBL"
 Bind#27
  oacdty=01 mxl=128(40) mxlc=00 mal=00 scl=00 pre=00
  oacflg=03 fl2=1000010 frm=01 csi=873 siz=0 off=2400
  kxsbbbfp=7f3b946adf80  bln=128  avl=10  flg=01
  value="RG_WRAP_UP"
 Incarnation curinc=1 instinc=1 curpdbinc=1 pdbinc=1
 Frames pfr 0x7f3b93dec578 siz=2113048 efr 0x7f3b8a3b6500 siz=2113016
 kxscphp=0x7f3b93dda720 siz=16960 inu=15120 nps=9192
 kxscbhp=0x7f3b93dd1c78 siz=8136 inu=6016 nps=2552
 kxscwhp=0x7f3b93dd1928 siz=4056 inu=200 nps=0
Starting SQL statement dump
SQL Information
user_id=1186 user_name=SLBR73 module=SQL Developer action=
sql_id=4vd4x866hyjh3 plan_hash_value=1783381835 problem_type=4 command_type=3
----- Current SQL Statement for this session (sql_id=4vd4x866hyjh3) -----
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :1  and table_name = :2
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :3  and table_name = :4
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :5  and table_name = :6
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :7  and table_name = :8
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :9  and table_name = :10
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :11  and table_name = :12
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :13  and table_name = :14
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :15  and table_name = :16
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :17  and table_name = :18
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :19  and table_name = :20
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :21  and table_name = :22
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :23  and table_name = :24
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :25  and table_name = :26
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :27  and table_name = :28
sql_text_length=2877
sql=SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :1  and table_name = :2
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :3  and table_name = :4
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :5  and table_name = :6
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :7  and table_name = :8
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :9  and table_name = :10
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :11  and table_name = :12
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :13  and table_name = :14
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :15  and table_name = :16
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :17  and table_name = :18
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :19  and table_name = :20
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :21  and table_name = :22
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :23  and table_name = :24
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :25  and table_name = :26
 union all
SELECT 'COLUMN' type, owner, table_name object_name, column_name, column_id, data_type
FROM sys.all_tab_cols
WHERE hidden_column = 'NO' and rownum <=999 and owner = :27  and table_name = :28
Compilation Environment Dump
optimizer_mode_hinted               = false
optimizer_features_hinted           = 0.0.0
parallel_execution_enabled          = false
parallel_query_forced_dop           = 0
parallel_dml_forced_dop             = 0
parallel_ddl_forced_degree          = 0
parallel_ddl_forced_instances       = 0
_query_rewrite_fudge                = 90
optimizer_features_enable           = 11.2.0.3
_optimizer_search_limit             = 5
cpu_count                           = 8
active_instance_count               = 1
parallel_threads_per_cpu            = 1
hash_area_size                      = 131072
bitmap_merge_area_size              = 1048576
sort_area_size                      = 65536
sort_area_retained_size             = 0
_sort_elimination_cost_ratio        = 0
_optimizer_block_size               = 8192
_sort_multiblock_read_count         = 2
_hash_multiblock_io_count           = 0
_db_file_optimizer_read_count       = 128
_optimizer_max_permutations         = 2000
pga_aggregate_target                = 8388608 KB
_pga_max_size                       = 1677720 KB
_query_rewrite_maxdisjunct          = 257
_smm_auto_min_io_size               = 120 KB
_smm_auto_max_io_size               = 1016 KB
_smm_min_size                       = 1024 KB
_smm_max_size_static                = 838860 KB
_smm_px_max_size_static             = 4194304 KB
_cpu_to_io                          = 0
_optimizer_undo_cost_change         = 11.2.0.3
parallel_query_mode                 = enabled
_enable_parallel_dml                = disabled
parallel_ddl_mode                   = enabled
optimizer_mode                      = choose
sqlstat_enabled                     = false
_optimizer_percent_parallel         = 101
_always_anti_join                   = choose
_always_semi_join                   = choose
_optimizer_mode_force               = true
_partition_view_enabled             = true
_always_star_transformation         = false
_query_rewrite_or_error             = false
_hash_join_enabled                  = true
cursor_sharing                      = exact
_b_tree_bitmap_plans                = true
star_transformation_enabled         = false
_optimizer_cost_model               = choose
_new_sort_cost_estimate             = true
_complex_view_merging               = true
_unnest_subquery                    = true
_eliminate_common_subexpr           = true
_pred_move_around                   = true
_convert_set_to_join                = false
_push_join_predicate                = true
_push_join_union_view               = true
_fast_full_scan_enabled             = true
_optim_enhance_nnull_detection      = true
_parallel_broadcast_enabled         = true
_px_broadcast_fudge_factor          = 100
_ordered_nested_loop                = true
_no_or_expansion                    = false
optimizer_index_cost_adj            = 100
optimizer_index_caching             = 0
_system_index_caching               = 0
_disable_datalayer_sampling         = false
query_rewrite_enabled               = true
query_rewrite_integrity             = trusted
_query_cost_rewrite                 = true
_query_rewrite_2                    = true
_query_rewrite_1                    = true
_query_rewrite_expression           = true
_query_rewrite_jgmigrate            = true
_query_rewrite_fpc                  = true
_query_rewrite_drj                  = false
_full_pwise_join_enabled            = true
_partial_pwise_join_enabled         = true
_left_nested_loops_random           = true
_improved_row_length_enabled        = true
_index_join_enabled                 = true
_enable_type_dep_selectivity        = true
_improved_outerjoin_card            = true
_optimizer_adjust_for_nulls         = true
_optimizer_degree                   = 0
_use_column_stats_for_function      = true
_subquery_pruning_enabled           = true
_subquery_pruning_mv_enabled        = false
_or_expand_nvl_predicate            = true
_like_with_bind_as_equality         = false
_table_scan_cost_plus_one           = true
_cost_equality_semi_join            = true
_default_non_equality_sel_check     = true
_new_initial_join_orders            = true
_oneside_colstat_for_equijoins      = true
_optim_peek_user_binds              = true
_minimal_stats_aggregation          = true
_force_temptables_for_gsets         = false
workarea_size_policy                = auto
_smm_auto_cost_enabled              = true
_gs_anti_semi_join_allowed          = true
_optim_new_default_join_sel         = true
optimizer_dynamic_sampling          = 2
_pre_rewrite_push_pred              = true
_optimizer_new_join_card_computation = true
_union_rewrite_for_gs               = yes_gset_mvs
_generalized_pruning_enabled        = true
_optim_adjust_for_part_skews        = true
_force_datefold_trunc               = false
statistics_level                    = typical
_optimizer_system_stats_usage       = true
skip_unusable_indexes               = true
_remove_aggr_subquery               = true
_optimizer_push_down_distinct       = 0
_dml_monitoring_enabled             = true
_optimizer_undo_changes             = false
_predicate_elimination_enabled      = true
_nested_loop_fudge                  = 100
_project_view_columns               = true
_local_communication_costing_enabled = true
_local_communication_ratio          = 50
_query_rewrite_vop_cleanup          = true
_slave_mapping_enabled              = true
_optimizer_cost_based_transformation = linear
_optimizer_mjc_enabled              = true
_right_outer_hash_enable            = true
_spr_push_pred_refspr               = true
_optimizer_cache_stats              = false
_optimizer_cbqt_factor              = 50
_optimizer_squ_bottomup             = true
_fic_area_size                      = 131072
_optimizer_skip_scan_enabled        = true
_optimizer_cost_filter_pred         = false
_optimizer_sortmerge_join_enabled   = true
_optimizer_join_sel_sanity_check    = true
_mmv_query_rewrite_enabled          = true
_bt_mmv_query_rewrite_enabled       = true
_add_stale_mv_to_dependency_list    = true
_distinct_view_unnesting            = false
_optimizer_dim_subq_join_sel        = true
_optimizer_disable_strans_sanity_checks = 0
_optimizer_compute_index_stats      = true
_push_join_union_view2              = true
optimizer_ignore_hints              = false
_optimizer_random_plan              = 0
_query_rewrite_setopgrw_enable      = true
_optimizer_correct_sq_selectivity   = true
_disable_function_based_index       = false
_optimizer_join_order_control       = 3
_optimizer_cartesian_enabled        = true
_optimizer_starplan_enabled         = true
_extended_pruning_enabled           = true
_optimizer_push_pred_cost_based     = true
_optimizer_null_aware_antijoin      = true
_optimizer_extend_jppd_view_types   = true
_sql_model_unfold_forloops          = run_time
_enable_dml_lock_escalation         = false
_bloom_filter_enabled               = true
_update_bji_ipdml_enabled           = 0
_optimizer_extended_cursor_sharing  = udo
_dm_max_shared_pool_pct             = 1
_optimizer_cost_hjsmj_multimatch    = true
_optimizer_transitivity_retain      = true
_px_pwg_enabled                     = true
optimizer_secure_view_merging       = true
_optimizer_join_elimination_enabled = true
flashback_table_rpi                 = non_fbt
_optimizer_cbqt_no_size_restriction = true
_optimizer_enhanced_filter_push     = true
_optimizer_filter_pred_pullup       = true
_rowsrc_trace_level                 = 0
_simple_view_merging                = true
_optimizer_rownum_pred_based_fkr    = true
_optimizer_better_inlist_costing    = all
_optimizer_self_induced_cache_cost  = false
_optimizer_min_cache_blocks         = 10
_optimizer_or_expansion             = depth
_optimizer_order_by_elimination_enabled = true
_optimizer_outer_to_anti_enabled    = true
_selfjoin_mv_duplicates             = true
_dimension_skip_null                = true
_force_rewrite_enable               = false
_optimizer_star_tran_in_with_clause = true
_optimizer_complex_pred_selectivity = true
_optimizer_connect_by_cost_based    = true
_gby_hash_aggregation_enabled       = true
_globalindex_pnum_filter_enabled    = true
_px_minus_intersect                 = true
_fix_control_key                    = -1614931837
_force_slave_mapping_intra_part_loads = false
_force_tmp_segment_loads            = false
_query_mmvrewrite_maxpreds          = 10
_query_mmvrewrite_maxintervals      = 5
_query_mmvrewrite_maxinlists        = 5
_query_mmvrewrite_maxdmaps          = 10
_query_mmvrewrite_maxcmaps          = 20
_query_mmvrewrite_maxregperm        = 512
_query_mmvrewrite_maxqryinlistvals  = 500
_disable_parallel_conventional_load = false
_trace_virtual_columns              = false
_replace_virtual_columns            = true
_virtual_column_overload_allowed    = true
_kdt_buffering                      = true
_first_k_rows_dynamic_proration     = true
_optimizer_sortmerge_join_inequality = true
_optimizer_aw_stats_enabled         = true
_bloom_pruning_enabled              = true
result_cache_mode                   = MANUAL
_px_ual_serial_input                = true
_optimizer_skip_scan_guess          = false
_enable_row_shipping                = true
_row_shipping_threshold             = 100
_row_shipping_explain               = false
transaction_isolation_level         = read_commited
_optimizer_distinct_elimination     = true
_optimizer_multi_level_push_pred    = true
_optimizer_group_by_placement       = true
_optimizer_rownum_bind_default      = 10
_enable_query_rewrite_on_remote_objs = true
_optimizer_extended_cursor_sharing_rel = simple
_optimizer_adaptive_cursor_sharing  = true
_direct_path_insert_features        = 0
_optimizer_improve_selectivity      = true
optimizer_use_pending_statistics    = false
_optimizer_enable_density_improvements = true
_optimizer_aw_join_push_enabled     = true
_optimizer_connect_by_combine_sw    = true
_enable_pmo_ctas                    = 0
_optimizer_native_full_outer_join   = force
_bloom_predicate_enabled            = true
_optimizer_enable_extended_stats    = true
_is_lock_table_for_ddl_wait_lock    = 0
_pivot_implementation_method        = choose
optimizer_capture_sql_plan_baselines = false
optimizer_use_sql_plan_baselines    = true
_optimizer_star_trans_min_cost      = 0
_optimizer_star_trans_min_ratio     = 0
_with_subquery                      = OPTIMIZER
_optimizer_fkr_index_cost_bias      = 10
_optimizer_use_subheap              = true
parallel_degree_policy              = manual
parallel_degree                     = 0
parallel_min_time_threshold         = 10
_parallel_time_unit                 = 10
_optimizer_or_expansion_subheap     = true
_optimizer_free_transformation_heap = true
_optimizer_reuse_cost_annotations   = true
_result_cache_auto_size_threshold   = 100
_result_cache_auto_time_threshold   = 1000
_optimizer_nested_rollup_for_gset   = 100
_nlj_batching_enabled               = 1
parallel_query_default_dop          = 0
is_recur_flags                      = 0
optimizer_use_invisible_indexes     = false
flashback_data_archive_internal_cursor = 0
_optimizer_extended_stats_usage_control = 192
_parallel_syspls_obey_force         = true
cell_offload_processing             = true
_rdbms_internal_fplib_enabled       = false
db_file_multiblock_read_count       = 128
_bloom_folding_enabled              = true
_mv_generalized_oj_refresh_opt      = true
cell_offload_compaction             = ADAPTIVE
cell_offload_plan_display           = AUTO
_bloom_predicate_offload            = true
_bloom_filter_size                  = 0
_bloom_pushing_max                  = 512
parallel_degree_limit               = 65535
parallel_force_local                = false
parallel_max_degree                 = 8
total_cpu_count                     = 8
_optimizer_coalesce_subqueries      = true
_optimizer_fast_pred_transitivity   = true
_optimizer_fast_access_pred_analysis = true
_optimizer_unnest_disjunctive_subq  = true
_optimizer_unnest_corr_set_subq     = true
_optimizer_distinct_agg_transform   = true
_aggregation_optimization_settings  = 0
_optimizer_connect_by_elim_dups     = true
_optimizer_eliminate_filtering_join = true
_connect_by_use_union_all           = true
dst_upgrade_insert_conv             = true
advanced_queuing_internal_cursor    = 0
_optimizer_unnest_all_subqueries    = true
parallel_autodop                    = 0
parallel_ddldml                     = 0
_parallel_cluster_cache_policy      = adaptive
_parallel_scalability               = 50
iot_internal_cursor                 = 0
_optimizer_instance_count           = 0
_optimizer_connect_by_cb_whr_only   = false
_suppress_scn_chk_for_cqn           = nosuppress_1466
_optimizer_join_factorization       = true
_optimizer_use_cbqt_star_transformation = true
_optimizer_table_expansion          = true
_and_pruning_enabled                = true
_deferred_constant_folding_mode     = DEFAULT
_optimizer_distinct_placement       = true
partition_pruning_internal_cursor   = 0
parallel_hinted                     = none
_sql_compatibility                  = 0
_optimizer_use_feedback             = true
_optimizer_try_st_before_jppd       = true
_dml_frequency_tracking             = false
_optimizer_interleave_jppd          = true
kkb_drop_empty_segments             = 0
_px_partition_scan_enabled          = true
_px_partition_scan_threshold        = 64
_optimizer_false_filter_pred_pullup = true
_bloom_minmax_enabled               = true
only_move_row                       = 0
_optimizer_enable_table_lookup_by_nl = true
parallel_execution_message_size     = 16384
_px_loc_msg_cost                    = 1000
_px_net_msg_cost                    = 10000
_optimizer_generate_transitive_pred = true
_optimizer_cube_join_enabled        = false
_optimizer_filter_pushdown          = true
deferred_segment_creation           = true
_optimizer_outer_join_to_inner      = true
_allow_level_without_connect_by     = false
_max_rwgs_groupings                 = 8192
_optimizer_hybrid_fpwj_enabled      = false
_px_replication_enabled             = false
ilm_filter                          = 0
_optimizer_partial_join_eval        = false
_px_concurrent                      = false
_px_object_sampling_enabled         = false
_px_back_to_parallel                = OFF
_optimizer_unnest_scalar_sq         = false
_optimizer_full_outer_join_to_outer = true
_px_filter_parallelized             = false
_px_filter_skew_handling            = false
_zonemap_use_enabled                = true
_zonemap_control                    = 0
_optimizer_multi_table_outerjoin    = false
_px_groupby_pushdown                = choose
_partition_advisor_srs_active       = true
_optimizer_ansi_join_lateral_enhance = false
_px_parallelize_expression          = false
_fast_index_maintenance             = true
_optimizer_ansi_rearchitecture      = false
_optimizer_gather_stats_on_load     = false
ilm_access_tracking                 = 0
ilm_dml_timestamp                   = 0
_px_adaptive_dist_method            = off
_px_adaptive_dist_method_threshold  = 0
_optimizer_batch_table_access_by_rowid = false
optimizer_adaptive_reporting_only   = false
_optimizer_ads_max_table_count      = 0
_optimizer_ads_time_limit           = 0
_optimizer_ads_use_result_cache     = true
_px_wif_dfo_declumping              = off
_px_wif_extend_distribution_keys    = false
_px_join_skew_handling              = false
_px_join_skew_ratio                 = 10
_px_join_skew_minfreq               = 30
CLI_internal_cursor                 = 0
_parallel_fault_tolerance_enabled   = false
_parallel_fault_tolerance_threshold = 3
_px_partial_rollup_pushdown         = off
_px_single_server_enabled           = false
_optimizer_dsdir_usage_control      = 0
_px_cpu_autodop_enabled             = false
_px_cpu_process_bandwidth           = 200
_sql_hvshare_threshold              = 0
_px_tq_rowhvs                       = true
_optimizer_use_gtt_session_stats    = false
_optimizer_proc_rate_level          = off
_px_hybrid_TSM_HWMB_load            = true
_optimizer_use_histograms           = true
PMO_altidx_rebuild                  = 0
_cell_offload_expressions           = true
_cell_materialize_virtual_columns   = true
_cell_materialize_all_expressions   = false
_rowsets_enabled                    = true
_rowsets_target_maxsize             = 524288
_rowsets_max_rows                   = 256
_use_hidden_partitions              = 0
_px_load_monitor_threshold          = 10000
_px_numa_support_enabled            = true
total_processor_group_count         = 2
_adaptive_window_consolidator_enabled = false
_optimizer_strans_adaptive_pruning  = false
_bloom_rm_filter                    = false
_optimizer_null_accepting_semijoin  = false
_long_varchar_allow_IOT             = 0
_parallel_ctas_enabled              = true
_cell_offload_complex_processing    = true
_optimizer_performance_feedback     = off
_optimizer_proc_rate_source         = DEFAULT
_hashops_prefetch_size              = 4
_cell_offload_sys_context           = true
_multi_commit_global_index_maint    = 0
_stat_aggs_one_pass_algorithm       = false
_dbg_scan                           = 0
_oltp_comp_dbg_scan                 = 0
_arch_comp_dbg_scan                 = 0
_optimizer_gather_feedback          = true
_upddel_dba_hash_mask_bits          = 0
_px_pwmr_enabled                    = true
_px_cdb_view_enabled                = true
_bloom_sm_enabled                   = true
_optimizer_cluster_by_rowid         = false
_optimizer_cluster_by_rowid_control = 3
_partition_cdb_view_enabled         = true
_common_data_view_enabled           = true
_pred_push_cdb_view_enabled         = true
_rowsets_cdb_view_enabled           = true
_distinct_agg_optimization_gsets    = off
_array_cdb_view_enabled             = true
_optimizer_adaptive_plan_control    = 0
_optimizer_adaptive_random_seed     = 0
_optimizer_cbqt_or_expansion        = off
_inmemory_dbg_scan                  = 0
_gby_vector_aggregation_enabled     = false
_optimizer_vector_transformation    = false
_optimizer_vector_fact_dim_ratio    = 10
_key_vector_max_size                = 0
_key_vector_predicate_enabled       = true
_key_vector_predicate_threshold     = 0
_vector_operations_control          = 0
_optimizer_vector_min_fact_rows     = 10000000
parallel_dblink                     = 0
_px_scalable_invdist                = false
_key_vector_offload                 = predicate
_optimizer_aggr_groupby_elim        = false
_optimizer_reduce_groupby_key       = false
_vector_serialize_temp_threshold    = 1000
_always_vector_transformation       = false
_optimizer_cluster_by_rowid_batched = false
_object_link_fixed_enabled          = true
optimizer_inmemory_aware            = true
_optimizer_inmemory_table_expansion = false
_optimizer_inmemory_gen_pushable_preds = false
_optimizer_inmemory_autodop         = false
_optimizer_inmemory_access_path     = false
_optimizer_inmemory_bloom_filter    = false
_parallel_inmemory_min_time_threshold = 1
_parallel_inmemory_time_unit        = 1
_rc_sys_obj_enabled                 = true
_optimizer_nlj_hj_adaptive_join     = false
_indexable_con_id                   = true
_bloom_serial_filter                = off
inmemory_force                      = default
inmemory_query                      = enable
_inmemory_query_scan                = true
_inmemory_query_fetch_by_rowid      = false
_inmemory_pruning                   = on
_px_autodop_pq_overhead             = true
_px_external_table_default_stats    = false
_optimizer_key_vector_aggr_factor   = 75
_optimizer_vector_cost_adj          = 100
_cdb_cross_container                = 65535
_cdb_view_parallel_degree           = 65535
_optimizer_hll_entry                = 4096
_px_cdb_view_join_enabled           = true
inmemory_size                       = 0
_external_table_smart_scan          = hadoop_only
_optimizer_inmemory_minmax_pruning  = false
_approx_cnt_distinct_gby_pushdown   = choose
_approx_cnt_distinct_optimization   = 0
_optimizer_ads_use_partial_results  = false
_query_execution_time_limit         = 0
_optimizer_inmemory_cluster_aware_dop = false
_optimizer_db_blocks_buffers        = 0 KB
_query_rewrite_use_on_query_computation = false
_px_scalable_invdist_mcol           = false
_optimizer_bushy_join               = off
_optimizer_bushy_fact_dim_ratio     = 20
_optimizer_bushy_fact_min_size      = 100000
_optimizer_bushy_cost_factor        = 100
_rmt_for_table_redef_enabled        = true
_optimizer_eliminate_subquery       = false
_sqlexec_cache_aware_hash_join_enabled = true
_zonemap_usage_tracking             = true
_sqlexec_hash_based_distagg_enabled = false
_sqlexec_disable_hash_based_distagg_tiv = false
_sqlexec_hash_based_distagg_ssf_enabled = false
_sqlexec_distagg_optimization_settings = 0
approx_for_aggregation              = false
approx_for_count_distinct           = false
_optimizer_union_all_gsets          = false
_rowsets_use_encoding               = true
_rowsets_max_enc_rows               = 64
_optimizer_enhanced_join_elimination = false
_optimizer_multicol_join_elimination = false
_key_vector_create_pushdown_threshold = 0
_optimizer_enable_plsql_stats       = false
_recursive_with_parallel            = false
_recursive_with_branch_iterations   = 1
_px_dist_agg_partial_rollup_pushdown = off
_px_adaptive_dist_bypass_enabled    = true
_enable_view_pdb                    = true
_optimizer_key_vector_pruning_enabled = false
_pwise_distinct_enabled             = false
_recursive_with_using_temp_table    = false
_partition_read_only                = true
_sql_alias_scope                    = true
_cdb_view_no_skip_migrate           = false
_approx_perc_sampling_err_rate      = 2
_sqlexec_encoding_aware_hj_enabled  = true
rim_node_exist                      = 0
_enable_containers_subquery         = true
_force_containers_subquery          = false
_cell_offload_vector_groupby        = true
_vector_encoding_mode               = off
_ds_xt_split_count                  = 0
_ds_sampling_method                 = NO_QUALITY_METRIC
_optimizer_ads_use_spd_cache        = false
_optimizer_use_table_scanrate       = OFF
_optimizer_use_xt_rowid             = false
_xt_sampling_scan_granules          = off
_recursive_with_control             = 0
_sqlexec_use_rwo_aware_expr_analysis = true
_optimizer_band_join_aware          = false
_optimizer_vector_base_dim_fact_factor = 0
approx_for_percentile               = none
_approx_percentile_optimization     = 0
_projection_pushdown                = true
_px_object_sampling                 = 200
_optimizer_adaptive_plans_continuous = false
_optimizer_adaptive_plans_iterative = false
_ds_enable_view_sampling            = false
_optimizer_generate_ptf_implied_preds = true
_optimizer_inmemory_use_stored_stats = NEVER
_cdb_special_old_xplan              = true
uniaud_internal_cursor              = 0
_kd_dbg_control                     = 0
_mv_access_compute_fresh_data       = off
_bloom_filter_ratio                 = 30
_optimizer_control_shard_qry_processing = 65529
containers_parallel_degree          = 65535
sql_from_coordinator                = 0
_xt_sampling_scan_granules_min_granules = 1
_in_memory_cdt                      = LIMITED
_in_memory_cdt_maxpx                = 4
_px_partition_load_dist_threshold   = 64
_px_adaptive_dist_bypass_optimization = 1
_optimizer_interleave_or_expansion  = false
optimizer_adaptive_plans            = true
optimizer_adaptive_statistics       = false
_optimizer_use_feedback_for_join    = false
_optimizer_ads_for_pq               = false
_px_join_skewed_values_count        = 0
optimizer_ignore_parallel_hints     = false
parallel_min_degree                 = 1
_px_nlj_bcast_rr_threshold          = 65535
_optimizer_gather_stats_on_load_all = false
_optimizer_gather_stats_on_load_hist = false
_optimizer_allow_all_access_paths   = true
_key_vector_double_enabled          = false
_key_vector_timestamp_enabled       = false
_optimizer_answering_query_using_stats = false
_disable_dblink_optim               = true
_cell_offload_hybrid_processing     = true
_read_optimized_table_lookup        = true
_optimizer_key_vector_payload       = false
_optimizer_vector_fact_payload_ratio = 20
_bloom_pruning_setops_enabled       = false
_bloom_filter_setops_enabled        = false
_key_vector_shared_dgk_ht           = true
_px_pwise_wif_enabled               = false
_sqlexec_reorder_wif_enabled        = false
_px_partition_skew_threshold        = 0
_sqlexec_pwiseops_with_sqlfuncs_enabled = false
_sqlexec_pwiseops_with_binds_enabled = false
_px_shared_hash_join                = false
_px_reuse_server_groups             = multi
_px_join_skew_null_handling         = false
_px_join_skew_use_histogram         = false
_px_join_skew_sampling_time_limit   = 0
_px_join_skew_sampling_percent      = 0
_cdb_view_no_skip_restricted        = false
_inmemory_external_table            = true
_sqlexec_use_kgghash3               = true
_cell_offload_vector_groupby_force  = false
_hcs_enable_pred_push               = false
parallel_dop_doubled                = 0
_optimizer_gather_stats_on_load_index = false
_con_map_sql_enforcement            = true
_uniq_cons_sql_enforcement          = true
_ref_cons_sql_enforcement           = true
_is_lock_table_for_split_merge      = 0
_cell_offload_vector_groupby_fact_key = false
_px_scalable_gby_invdist            = false
_px_dynamic_granules                = false
_px_dynamic_granules_adjust         = 0
_px_hybrid_partition_execution_enabled = false
_px_hybrid_partition_skew_threshold = 255
_optimizer_key_vector_use_max_size  = 1048576
_cell_offload_vector_groupby_withnojoin = false
_key_vector_join_pushdown_enabled   = false
_cell_offload_grand_total           = false
_optimizer_key_vector_payload_dim_aggs = false
_optimizer_use_auto_indexes         = OFF
_optimizer_use_stats_on_conventional_dml = false
_optimizer_gather_stats_on_conventional_dml = false
_optimizer_use_stats_on_conventional_config = 0
_skip_pset_col_chk                  = 0
_nls_binary                         = true
_optimizer_quarantine_sql           = false
_optimizer_gather_stats_on_conventional_config = 0
_nls_forced_binary                  = 0
_utl32k_mv_query                    = false
optimizer_session_type              = NORMAL
BlockChain_ledger_infrastructure    = 0
_cell_offload_vector_groupby_external = true
_bigdata_offload_flag               = false
container_data                      = ALL
optimizer_real_time_statistics      = false

Fine-Grained Invalidation state dump:
fine-grained dependencies:
  [0] typ=6 (INDEX TBL), objn=425, pnum=[0,0], flags=0x00000000
  [1] typ=1 (INDEX), objn=427, pnum=[0,0], flags=0x00000000
  [2] typ=1 (INDEX), objn=1260008, pnum=[0,0], flags=0x00000000
  [3] typ=1 (INDEX), objn=1260010, pnum=[0,0], flags=0x00000000
  [4] typ=1 (INDEX), objn=1260049, pnum=[0,0], flags=0x00000000
  [5] typ=1 (INDEX), objn=1260006, pnum=[0,0], flags=0x00000000
  [6] typ=6 (INDEX TBL), objn=61, pnum=[0,0], flags=0x00000000
  [7] typ=1 (INDEX), objn=62, pnum=[0,0], flags=0x00000000
  [8] typ=6 (INDEX TBL), objn=1260017, pnum=[0,0], flags=0x00000000
  [9] typ=1 (INDEX), objn=1260050, pnum=[0,0], flags=0x00000000
  [10] typ=6 (INDEX TBL), objn=1260005, pnum=[0,0], flags=0x00000000
  [11] typ=1 (INDEX), objn=1260009, pnum=[0,0], flags=0x00000000
cursor refresh state:
  maCtxInfo_ctxdef=0x92cdc8a8
  papRefresh: infoMut=0, cursor=0
  tidRefresh: infoMut=0, cursor=0
  FGIRolling: infoMut=0, cursor=0
    FGIValid: infoMut=0, cursor=0
  refreshCnt:  ctxmut=0, cursor=0
       valid: 1
End of Fine-Grained Invalidation state dump
====================== END SQL Statement Dump ======================
2025-05-19T15:18:32.180524+01:00
Incident 242013 created, dump file: /u01/app/oracle/diag/rdbms/ci01gpro/CI01GPRO_1/incident/incdir_242013/CI01GPRO_1_ora_221839_i242013.trc
ORA-03137: malformed TTC packet from client rejected: [3146] [4] [] [] [] [] [] []


*** 2025-05-19T15:19:21.864381+01:00
2025-05-19T15:19:21.864366+01:00
Incident 242014 created, dump file: /u01/app/oracle/diag/rdbms/ci01gpro/CI01GPRO_1/incident/incdir_242014/CI01GPRO_1_ora_221839_i242014.trc
ORA-03137: malformed TTC packet from client rejected: [3146] [94] [] [] [] [] [] []
ORA-03138: Connection terminated due to security policy violation
