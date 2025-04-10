1st step, I dont want to give schema name in the script it should refer a config file
expdp system/"cowboy_1" directory=DATA_PUMP_DRMQ schemas=**** dumpfile=DRM_date_28_%U.dmp logfile=DRM_date_28.log parallel=5

once export is done it should go to second step and should only use below queey
2nd step which will generate drop quries for a schema, but when i run this i dont want to enter the schema name manually I wwant to create a config file it should take from that config file
set pages 0
set lines 300
set heading off
spool /tmp/dropobj.sql
select 'drop table '||owner||'.'||table_name||' cascade constraints purge;'
from dba_tables
where owner = upper('&&owner')
union all
select 'drop '||object_type||' '||owner||'.'||object_name||';'
from dba_objects
where object_type not in ('TABLE','INDEX','PACKAGE BODY','TRIGGER','LOB')
and object_type not like '%LINK%'
and object_type not like '%PARTITION%'
and owner = upper('&&owner')
order by 1;

3rd step impdp of scp prod dumps
DATA_PUMP_DRMQ
/mount/DBDESIGN/exports/eb37drmq
REMAP_SCHEMA=DRM_PROD:***** I dont want to give remapschema=DRM_PROD:**** it should refer a config file
impdp system/"cowboy_1" DIRECTORY=DATA_PUMP_DRMQ DUMPFILE=DRM_EB37_2309_%U.dmp logfile=imp_EB37DRM2309_.log REMAP_SCHEMA=DRM_PROD:*****
