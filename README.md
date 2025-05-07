SHARABLE_MEM EXECUTIONS PARSE_CALLS      LOADS SQL_ID        SQL_TEXT
------------ ---------- ----------- ---------- ------------- ----------------------------------------------------------------------------------------------------
   7,715,014        714         530        143 56ww4chtcmh8w select  EPV_VAR_ID,VERSION_ID,NAME,VAR_NAME,EXT_DESCRIPTION,DEFAULT_VALUE,GUID,LAST_MODIFIED,EPV_ID,
   4,193,739        104          16          9 0za9fv0j1vgkk WITH MONITOR_DATA AS (SELECT * FROM TABLE(GV$(CURSOR( SELECT USERENV('instance') AS INST_ID, KEY, NV
   3,189,352          2           2          1 fu9z5zkznfmqu        with disk_names as (      select /*+ materialize */           dbid, snap_id, cell_hash, disk_
   3,188,315         15          14          2 bpu5fw867n8un SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   3,178,384          3           3          2 fdqntbsat1whq SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by taskDueDate, instanceId, taskPri
   2,738,972          2           2          1 frxyp4xsu8pqy        with disk_names as (      select /*+ materialize */           dbid, snap_id, cell_hash, disk_
   2,734,602          7           7          2 bkzr5bfhw00bv select count (*)  from lsw_task t  left join lsw_bpd_instance i on t.bpd_instance_id = i.bpd_instanc
   2,380,555          6           6          2 0k11tw6fz62dr SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   2,345,891         25          24         73 2skdztawzw38c SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   2,278,571          2           2          2 3txcyyqx86ybc SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   2,148,238          9           9          4 fxhkyu7yt01tr SELECT NULL AS table_cat,        o.owner AS table_schem,        o.object_name AS table_name,
   2,107,031          3           3          2 ajkbcndvq3j8a SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by taskSubject, instanceId, taskDue
   2,059,980        104         100         10 atwuyuvqkf27w SELECT /*+ OPT_PARAM('_fix_control' '16391176:1') */ GROUP_TYPE, BUCKET_START, BUCKET_END, TM_GROUP_
   1,988,690         28          27         87 0wnw3buj2d7yc select count (*)  from lsw_task t  left join lsw_bpd_instance i on t.bpd_instance_id = i.bpd_instanc
   1,937,371         22          19          6 9cfu83d8pq5tw SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,904,931       3694        3492        227 agv7mzbamrrmr SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,767,779       2280        2207        170 fn5q6q4m3pyhb SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,731,938          4           4          2 c9f7gctu3xmtv select count (*)  from lsw_task t  left join lsw_bpd_instance i on t.bpd_instance_id = i.bpd_instanc
   1,655,556         11           4          3 fg11facbfjwvx            SELECT  /*+ no_monitor leading(mo_px) use_hash(mo_px) */           xmlelement(
   1,623,875         53          53         79 0xqac5mwzy4g7 SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,622,595          2           2          1 bcsw42s78r4x5        with disk_names as (      select /*+ materialize */           dbid, snap_id, cell_hash, disk_
   1,578,813         69          69         76 9vz29bndq93wf SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,557,019          1           1          1 fj6f4t8hj5rhx SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,527,219       4029        3937        191 5zvq1qp6akqf1 SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,495,211        231         209         66 2y1fn4qwvg0qw SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,460,986        356         232         33 7vp9vn5s6nh8u select po_type, po_id, po_root_id, po_guid, po_version_id, po_orig_version_id, version_summary_id, p
   1,393,431          2           2          1 36k6j4z6sx195        with disk_names as (      select /*+ materialize */           dbid, snap_id, cell_hash, disk_
   1,365,884          1           1          1 bty8rawav46df WITH binds as           (select :dbid                  as dbid                , :inst_id_low
   1,303,547          3           3          1 c3x7zr1a470zn SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,294,867       1111        1034         55 5v4ak0js08865 SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,252,467          4           4          2 f3ag3kusshzst SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,209,787       2725        2659        160 bcqdcmpdxsh9f SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,
   1,169,123          1           1          1 1y36rh6821tj4 WITH jd AS (SELECT end_time, input_bytes, session_key, session_recid, session_stamp, input_type, out
   1,146,783          1           1          1 92yjj5ydx1gmk WITH binds as           (select :dbid                  as dbid                , :inst_id_low
   1,114,427       1037        1009        185 aa702tw5qryc4 SELECT * FROM ( SELECT actualSearch.*, ROW_NUMBER() OVER ( order by PSSPECIFICDUEDATE, taskDueDate,

35 rows selected.
