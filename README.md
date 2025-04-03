#!/bin/sh
######################## Function ContYN - process control ####################################
ContYN () {
echo "Continue Y/N"
read Var_CONTYN
if [ -z ${Var_CONTYN} ]
then
   Var_CONTYN="N"
fi
if [ ${Var_CONTYN} != "Y" -a ${Var_CONTYN} != "y" ]
then
   echo "Stopping"
   exit
fi
}

########## Function extract_cloning_candidates: only show clone candidates from this compute node ##########
extract_cloning_candidates () {
# run function to select the target DB - in this case also ONLY the EBS databases via the egrep
  CURRENT_COMPUTE_NODE=`hostname -s`
  export CONFIGG="/mount/PRODDBA/oracle_scripts/general/clone_config.txt"
  export CONFIGF="/tmp/cloning_candidates"

# now pick out only the 'exa' entries running on this host from the config file (and include the header line via exaXdb0X)
#Instance; Target ; Source ;Oracle_Home                                 ;Lsnr_Port;Directory Name;  Source Directory;RAC_Instance;zfssa ID; Backup Destination PostClone;PRI;CLO
#EB21FSYS;exa5dbadm02;exa7dbadm02;/u01/app/oracle/product/11.2.0.3.28/ebsfsys;1542;fsys;/zfssa/EBS_CLONE/backup1/EB21PROD;EB21FSYS1;EXA5_EB21FSYS;/zfssa/EXA5_EB21FSYS/backup1;LEG7;LEG5

  egrep "Instance|$CURRENT_COMPUTE_NODE;e" ${CONFIGG}|egrep 'Instance|^EB01|^EB21' > ${CONFIGF}
  echo "Original Config file: " ${CONFIGG}
  echo "Reading Target configuration details from file: " ${CONFIGF}
}

####################### Function get_APPS_pwd - prompt for APPS password ################
get_APPS_pwd () {
echo " "
#echo "EB01TEST apps Gr4ss_H0pp3r"
echo "EB21PROD apps stdl1f3_prd"
#echo "EB21FSYS apps stdl1f3_fsys1"
echo "EB21SYST apps syst_appsus3r"
echo "EB21SYIT apps stdl1f3_syit"
echo "EB01DEV2 apps dev2_appsus3r"
echo "EB21BSYS apps bsys_appsus3r"
echo "EB21MIGR apps stdl1f3_mig"
echo "EB21DEVT apps stdl1f3_dev"
echo "EB21PATC apps stdl1f3_patc"
echo "This password is mandatory for many of the steps in the following menu."
echo "Now enter the APPS password for the database named in the following path: ${CLONE_SRC_DIR} (case sensitive):"
read APPS_PASS
echo " "
if [ -z ${APPS_PASS} ]
   then
     echo "Need a APPS password, exiting ....."
     exit
fi
export APPS_CONN_STR="APPS/"${APPS_PASS}
#echoVars
}

####################### Function get_target_db - menu to select db for cloning ###############
get_target_db () {
#Instance; Target ; Source ;Oracle_Home                                 ;Lsnr_Port;Directory Name;  Source Directory;RAC_Instance;zfssa ID; Backup Destination PostClone;PRI;CLO
#EB21SYIT;exa5dbadm02;exa7dbadm02;/u01/app/oracle/product/11.2.0.3.28/ebssyit;1529;syit;/zfssa/EBS_CLONE/backup1/EB21PROD;EB21SYIT;EXA5_EB21SYIT;/zfssa/EXA5_EB21SYIT/backup1;LEG7;LEG5

# list the options for the target DB from the config file so we can pick it
echo " "
echo "Select TARGET database:"
echo "-----------------------"
awk 'BEGIN { FS = ";"; seq=-1 }
{seq=seq+1}
seq == 0 {print $1 " " $2 " " $7 }
seq != 0 {print seq ". " $1 " " $2 " " $7}
END {}' $CONFIGF
echo ""

TGT_NO=0
export MAX_TGT_NO=`wc -l ${CONFIGF}|awk '{print $1}'`

while [ ${TGT_NO} -le 0 -o ${TGT_NO} -ge ${MAX_TGT_NO} ]
do
   echo "Input the Target number for new DB 1 -" `expr ${MAX_TGT_NO} - 1`
   read TGT_NO

#  empty string input so try again
   if [ -z ${TGT_NO} ]
   then
     TGT_NO=0
   fi
   # if it has alphas then message numeric only set to 0 so we loop again.
   if [[ ! -z $(echo ${TGT_NO} | sed 's/[0-9]//g') ]]
   then
     echo "integer only"
     TGT_NO=0
   fi
done

# fix the number as the config file has a header line.
TGT_NO=`expr ${TGT_NO} + 1`
TGT_LINE=`head -${TGT_NO} ${CONFIGF}|tail -1`
# Now extract and display the four values from the config file
CLONE_TGT=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $1} END {}'`
CLONE_LC_TGT=`echo ${CLONE_TGT} | tr '[:upper:]' '[:lower:]'`
CLONE_UC4_TGT=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print substr($1,5,4)} END {}'`
CLONE_LC4_TGT=`echo $CLONE_UC4_TGT |tr '[:upper:]' '[:lower:]'`
HOSTNAM_TGT=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $2} END {}'`
HOSTNAM_SRC=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $3} END {}'`
HOST_UC4_TGT=`echo ${HOSTNAM_TGT} | awk 'BEGIN { FS=";" } {print substr($1,1,4)} END {}' | tr '[:lower:]' '[:upper:]'`
HOST_EXA_NUM_TGT=`echo ${HOSTNAM_TGT} | awk '{print substr($1,4,1)}'`
HOST_EXA_NUM_SRC=`echo ${HOSTNAM_SRC} | awk '{print substr($1,4,1)}'`
NODE_TGT=`echo ${HOSTNAM_TGT} | awk '{print substr($1,8,1)}'`
LSNR_PORT=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $5} END {}'`
CLONE_TGT_DBA_DIR=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $6} END {}'`
CLONE_SRC_DIR=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $7} END {}'`
CLONE_SRC=`echo ${CLONE_SRC_DIR} | awk 'BEGIN { FS="/" } {print $5} END {}'`
CLONE_LC_SRC=`echo ${CLONE_SRC} | awk 'BEGIN { FS="/" } {print $1} END {}' | tr '[:upper:]' '[:lower:]'`
CLONE_TGT_RAC_INST=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $8} END {}'`
CLONE_ZFSSA_ID=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $9} END {}'`
CLONE_BACKUP_TARGET=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $10} END {}'`
PRIMARY_DATA=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $11} END {}'`
CLONE_DATA=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $12} END {}'`
PRIMARY_DATA_LC=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $11} END {}' | tr '[:upper:]' '[:lower:]'`
CLONE_DATA_LC=`echo ${TGT_LINE} | awk 'BEGIN { FS=";" } {print $12} END {}' | tr '[:upper:]' '[:lower:]'`
FORMATTED_DATE=`date +%Y%m%d_%H%M%S`

echo "You have selected the following TARGET system. Is this correct?"
echo "CLONE_TGT="$CLONE_TGT  $CLONE_LC4_TGT
echo "HOST_TGT="$HOSTNAM_TGT
echo "HOST_EXA_NUM_TGT="$HOST_EXA_NUM_TGT
echo "HOST_EXA_NUM_SRC="$HOST_EXA_NUM_SRC
echo "LSNR_PORT="$LSNR_PORT
echo "CLONE_TGT_DBA_DIR="$CLONE_TGT_DBA_DIR
echo "CLONE_SRC_DIR="$CLONE_SRC_DIR "CLONE_SRC=" $CLONE_SRC
echo "CLONE_TGT_RAC_INST="$CLONE_TGT_RAC_INST
echo "CLONE_ZFSSA_ID="$CLONE_ZFSSA_ID
echo "CLONE_BACKUP_TARGET="$CLONE_BACKUP_TARGET
echo "PRIMARY_DATA Group="$PRIMARY_DATA  "CLONE_DATA Group="$CLONE_DATA
echo "FORMATTED_DATE="$FORMATTED_DATE
echo " "
}

####################### Function f02 - store original target settings for reset later ################
f02 () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
spool /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f02_setting_${FORMATTED_DATE}.log

create pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}_f02_preREFRESH.ora' from spfile;

show parameter service
show parameter domain
show parameter utl_file_dir
show parameter undo
set linesize 300
set pagesize 400

select substr(name,1,30)||' '||substr(value,1,100) CHANNEL from v\$rman_configuration where name='CHANNEL';

select 'CREATE OR REPLACE DIRECTORY '||directory_name||' AS '''||directory_path||''';' from dba_directories;
SQLEOF

echo " .. retrieved dba_directories from before clone capture file"
egrep '^CREATE' /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f02_setting_${FORMATTED_DATE}.log > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f02_directory_set.sql

export f02_flag=`date`
}

####################### Function f03 - derive db_file_name_convert AND log_file_name_convert parameters to be added to "original" pfile ################
f03 () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

echo "SID "$ORACLE_SID "  HOME" $ORACLE_HOME

if [ "${HOST_EXA_NUM_SRC}" = "${HOST_EXA_NUM_TGT}" ]; then
#  echo "nowt if"
 echo "*.db_file_name_convert='+DATA_"${PRIMARY_DATA}"/"${CLONE_SRC}"','+DATA_"${CLONE_DATA}"/"${CLONE_TGT}"','+RECO_"${PRIMARY_DATA}"/"${CLONE_SRC}"','+RECO_"${CLONE_DATA}"/"${CLONE_TGT}"','+data_"${PRIMARY_DATA_LC}"/"${CLONE_LC_SRC}"','+data_"${CLONE_DATA_LC}"/"${CLONE_LC_TGT}"','+reco_"${PRIMARY_DATA_LC}"/"${CLONE_LC_SRC}"','+reco_"${CLONE_DATA_LC}"/"${CLONE_LC_TGT}"'"  > $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt
  echo "*.log_file_name_convert='+DATA_"${PRIMARY_DATA}"/"${CLONE_SRC}"','+DATA_"${CLONE_DATA}"/"${CLONE_TGT}"','+RECO_"${PRIMARY_DATA}"/"${CLONE_SRC}"','+RECO_"${CLONE_DATA}"/"${CLONE_TGT}"','+data_"${PRIMARY_DATA_LC}"/"${CLONE_LC_SRC}"','+data_"${CLONE_DATA_LC}"/"${CLONE_LC_TGT}"','+reco_"${PRIMARY_DATA_LC}"/"${CLONE_LC_SRC}"','+reco_"${CLONE_DATA_LC}"/"${CLONE_LC_TGT}"'" >> $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt
else
#  echo "nowt else"
  echo "*.db_file_name_convert='+DATA_"${PRIMARY_DATA}"','+DATA_"${CLONE_DATA}"','+RECO_"${PRIMARY_DATA}"','+RECO_"${CLONE_DATA}"','+data_"${PRIMARY_DATA_LC}"','+data_"${CLONE_DATA_LC}"','+reco_"${PRIMARY_DATA_LC}"','+reco_"${CLONE_DATA_LC}"'" > $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt
  echo "*.log_file_name_convert='+DATA_"${PRIMARY_DATA}"','+DATA_"${CLONE_DATA}"','+RECO_"${PRIMARY_DATA}"','+RECO_"${CLONE_DATA}"','+data_"${PRIMARY_DATA_LC}"','+data_"${CLONE_DATA_LC}"','+reco_"${PRIMARY_DATA_LC}"','+reco_"${CLONE_DATA_LC}"'" >> $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt
fi
echo "*.cluster_database=false" >> $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt
echo " "
echo " parameters created for db_file_name_convert and log_file_name_convert"
echo " "
cat $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
create pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_pfile' from spfile;
quit
SQLEOF

mv $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_spfile
cp $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_spfile $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_ASM_keep

echo " copy initialisation parameters from ${CLONE_TGT_RAC_INST} aside to provide starter pfile for new instance ${CLONE_TGT_RAC_INST}"
cat $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_pfile $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt > $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora

echo " "
echo " five files .."
echo "  init${CLONE_TGT_RAC_INST}.ora"
echo "  init${CLONE_TGT_RAC_INST}.ora_f03_pfile"
echo "  ${CLONE_TGT_RAC_INST}_f03_filename_convert.txt"
echo "  init${CLONE_TGT_RAC_INST}.ora_f03_spfile"
echo "  init${CLONE_TGT_RAC_INST}.ora_f03_ASM_keep"
ls -l $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_pfile $ORACLE_HOME/dbs/${CLONE_TGT_RAC_INST}_f03_filename_convert.txt $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_spfile $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_ASM_keep

export f03_flag=`date`
}

####################### Function f04 - delete backups via RMAN ############################
f04 () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
echo "SID "$ORACLE_SID "  HOME" $ORACLE_HOME

echo "Deleting Backups - why not just rm from zfs"
echo "----------------"
export RMAN_TGT_CONNECT="sys/R4nd0m_s"
#export RMAN_TGT_CONNECT="sys/R4nd0m_s@"${CLONE_TGT_RAC_INST}
export RMAN_DEL_COMMAND="delete noprompt backup;"
export RMAN_DEL_COMMAND2="delete noprompt archivelog all;"

rman << RMANEOF
connect target $RMAN_TGT_CONNECT
$RMAN_DEL_COMMAND
$RMAN_DEL_COMMAND2
RMANEOF

export f04_flag=`date`
}

###################### Function f05 - stop listener ######################################
f05 () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
echo "SID "$ORACLE_SID "  HOME" $ORACLE_HOME

lsnrctl stop ${CLONE_TGT}

export f05_flag=`date`
}

##################### Function f06 - removing any left-overs in zfs backup1 directory ######################
f06 () {
echo "Directory content to be removed - remaining after RMAN delete - removal of content of /zfssa/${CLONE_ZFSSA_ID}/backup1 and beyond"
echo "Current contents:"
ls -lotr /zfssa/${CLONE_ZFSSA_ID}/backup1
ContYN

cd /zfssa/${CLONE_ZFSSA_ID}/backup1;rm -R *

export f06_flag=`date`
}

##################### Function f06a - shutdown immediate database ####################
f06a () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
echo "SID "$ORACLE_SID "  HOME" $ORACLE_HOME

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
create spfile='$ORACLE_HOME/dbs/spfile${CLONE_TGT_RAC_INST}.ora' from pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora';
shutdown immediate;
quit
SQLEOF

echo "Duplicate spfile created here:"
ls -lrt $ORACLE_HOME/dbs/spfile${CLONE_TGT_RAC_INST}.ora

export f06a_flag=`date`
}

#################### Function f06b - prompt to delete database files manually from ASM via another session ##########
f06b () {

echo " "
echo " PROMPT: delete the datafiles from ASM manually in another session"
echo " "
ContYN

export f06b_flag=`date`
}

#################### Function f07 - the RMAN duplicate database step ############################
f07 () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
echo "SID "$ORACLE_SID "  HOME" $ORACLE_HOME

export NLS_DATE_FORMAT="DD/MM/YYYY HH24:MI:SS"
export DATE_SUFFIX=`date +%Y%m%d%H%M`

echo " if the last line shows cluster_database=true or the convert lines look wrong"
echo " .. you have five seconds to press  Ctrl-C to stop the imminent failure"
echo "    of the restore (after it has done all the work!)"
echo " "
egrep 'cluster_database|file_name_convert' $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora
sleep 5

echo " "
echo " commands being executed:"
#echo "   startup nomount pfile=$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora;"
echo "   startup nomount ..... and picking up the default $ORACLE_HOME/dbs/spfile${CLONE_TGT_RAC_INST}.ora;"
echo "   duplicate database to '${CLONE_TGT}' backup location '${CLONE_SRC_DIR}';"
echo " use tail -f $ORACLE_HOME/dbs/$DATE_SUFFIX_duplicate_${CLONE_TGT_RAC_INST}.log on separate session to see progress"
echo " log='$ORACLE_HOME/dbs/srdc_rman_dup_output_$DATE_SUFFIX.log'"
echo " "

#export RMAN_START_COMMAND="startup nomount pfile=$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora;"
export RMAN_START_COMMAND="startup nomount"
#export RMAN_LOG_COMMAND="log='$ORACLE_HOME/dbs/`date +%Y%m%d`_duplicate_${CLONE_TGT_RAC_INST}.log'"
export RMAN_LOG_COMMAND="trace='$ORACLE_HOME/dbs/srdc_rman_dup_debug_$DATE_SUFFIX.trc' log='$ORACLE_HOME/dbs/srdc_rman_dup_output_$DATE_SUFFIX.log'"
export RMAN_DUP_COMMAND="duplicate database to '${CLONE_TGT}' backup location '${CLONE_SRC_DIR}';"

rman << RMANEOF
connect target /
$RMAN_START_COMMAND
RMANEOF

rman $RMAN_LOG_COMMAND  << RMANEOF
connect auxiliary /

set echo on;
debug on;

run {
ALLOCATE AUXILIARY CHANNEL AUX_DISK_1 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_2 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_3 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_4 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_5 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_6 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_7 TYPE DISK ;
ALLOCATE AUXILIARY CHANNEL AUX_DISK_8 TYPE DISK ;

$RMAN_DUP_COMMAND
}
debug off;
RMANEOF

echo "Proof of the Pudding - tail of restore log"
tail -30 $ORACLE_HOME/dbs/srdc_rman_dup_output_$DATA_SUFFIX.log

export f07_flag=`date`
}
#################### Function f07a - recreate block change tracking ############################
f07a () {
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
echo "SID "$ORACLE_SID "  HOME" $ORACLE_HOME

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
ALTER DATABASE ENABLE BLOCK CHANGE TRACKING;
quit
SQLEOF

export f07a_flag=`date`
}

## Following BUG where spfile is deleted mid-duplicate - this step no longer required ##
################### Function f08 - retrieve from ASM control file names recovered during database restore ###############
f08 () {

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

export ASM_SID="+ASM${NODE_TGT}"

sudo su - grid -c "export ORAENV_ASK=NO; export ORACLE_SID=${ASM_SID};. oraenv; asmcmd ls -l +data_exa${HOST_EXA_NUM_TGT}/${CLONE_TGT}/controlfile > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_data_control.txt"
sudo su - grid -c "export ORAENV_ASK=NO; export ORACLE_SID=${ASM_SID};. oraenv; asmcmd ls -l +reco_exa${HOST_EXA_NUM_TGT}/${CLONE_TGT}/controlfile > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_reco_control.txt"

echo "*.control_files='+data_exa${HOST_EXA_NUM_TGT}/${CLONE_TGT}/controlfile/`egrep Current /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_data_control.txt | awk 'BEGIN { FS=";" } {print substr($1,53,30)} END {}'`','+reco_exa${HOST_EXA_NUM_TGT}/${CLONE_TGT}/controlfile/`egrep Current /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_reco_control.txt | awk 'BEGIN { FS=";" } {print substr($1,53,30)} END {}'`'" > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_control_file.txt

cp $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f08_b4_control
cat $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f08_b4_control /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_control_file.txt > $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora

echo " "
echo " three files"
echo "  $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f08_b4_control /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_control_file.txt $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora"
ls -lrt $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f08_b4_control /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_control_file.txt $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora
echo " "
export f08_flag=`date`
}

## Following BUG where spfile is deleted mid-duplicate - this variant of this step is no longer required ##
################### Function f08a - shutdown immediate and restart with new control file names added #############
f08a () {

echo " shutdown immediate - then restart with new control file names added"
cat /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f08_control_file.txt

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
shutdown immediate
startup pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora'
quit
SQLEOF

export f08a_flag=`date`
}

################### Function f08b - create pfile on disk from memory/disk spfile, shutdown immediate and restart with new control file names added #############
f08b () {

echo " create pfile on disk from memory/disk spfile, shutdown immediate - then restart with new control file names added"

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
create pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora' from spfile;
shutdown immediate
startup pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora'
quit
SQLEOF

export f08b_flag=`date`
}

################### Function f09 - create ASM spfile - shutdown immediate - move f03_ASM_keep file into .ora file - startup using ASM spfile ###################
f09 () {

echo " "
echo " create spfile - shutdown immediate - move f03_ASM_keep file into .ora file - startup using ASM spfile"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
create spfile='+DATA_LEG${HOST_EXA_NUM_TGT}/${CLONE_TGT}/spfile${CLONE_TGT}.ora' from pfile='$ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora';
shutdown immediate
quit
SQLEOF

# Now put back into the init.ora file the entry that points to the ASM spfile alias
mv $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f09_pfile_keep
mv $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora_f03_ASM_keep $ORACLE_HOME/dbs/init${CLONE_TGT_RAC_INST}.ora

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
startup
quit
SQLEOF

cd $ORACLE_HOME/dbs/
ls -l init${CLONE_TGT_RAC_INST}.ora_pfile ${CLONE_TGT_RAC_INST}_f08_filename_convert.txt init${CLONE_TGT_RAC_INST}.ora init${CLONE_TGT_RAC_INST}.ora_spfile ${CLONE_TGT_RAC_INST}*

export f09_flag=`date`
}

################## Function f10 - drop production database startup trigger - stop/start database ###############
f10 () {

echo " "
echo " drop production startup trigger - stop / start database"
echo " "

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
drop trigger sys.check_eb21user_start;
shutdown immediate
quit
SQLEOF

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
startup
quit
SQLEOF

export f10_flag=`date`
}

################# Function f11: execute fnd_conc_clone.setup_clean #################
f11 () {

echo " "
echo " execute fnd_conc_clone.setup_clean"
echo " "

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

#SQL_CONN_STR="APPS/"${APPS_PASS}"@"${CLONE_TGT}
SQL_CONN_STR="APPS/"${APPS_PASS}

${ORACLE_HOME}/bin/sqlplus ${SQL_CONN_STR} << SQLEOF
select NAME,CREATED,host_name from v\$database, v\$instance;
exec fnd_conc_clone.setup_clean
quit
SQLEOF

export f11_flag=`date`
}

################ Function f12: restart listener so that adautocfg works ##############
f12 () {

echo " "
echo " start listener in preparation for adautocfg execution"
echo " "

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

lsnrctl start ${CLONE_TGT}

export f12_flag=`date`
}

############### Function f13: verify APPS connects can connect via listener then execute adautocfg ###############
f13 () {

echo "APPS_PWD" $APPS_PASS
echo " verifying apps user still works via listener"

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

SQL_CONN_STR="APPS/"${APPS_PASS}"@"${CLONE_TGT}

$ORACLE_HOME/bin/sqlplus ${SQL_CONN_STR} <<SQLEOF
select NAME,CREATED,host_name from v\$database, v\$instance;
SQLEOF

echo " "
echo " execute adautocfg - cut and paste APPS password when prompted - look for AutoConfig completed successfully."
echo " "

# Executing adautocfg
$ORACLE_HOME/appsutil/scripts/${CLONE_TGT_RAC_INST}_${HOSTNAM_TGT}/adautocfg.sh

export f13_flag=`date`
}

############## Function f14: update sys/system/dbsnmp passwords on restored database ##################
f14 () {

echo " "
echo " update sys/system/dbsnmp passwords on restored database"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
export NEW_SYSTEM="${CLONE_LC_TGT}_eb21"

echo 'alter user sys identified by "R4nd0m_s" account unlock;'            > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f14_passwords.sql
echo 'alter user system identified by "'${NEW_SYSTEM}'" account unlock;' >> /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f14_passwords.sql
echo 'alter user dbsnmp identified by "Y0u_full" account unlock;'        >> /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f14_passwords.sql
echo 'grant sysdba to dbsnmp;'                                           >> /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f14_passwords.sql

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
@/mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f14_passwords.sql
SQLEOF

export f14_flag=`date`
}

############## Function f16: drop extra RAC-specific UNDO tablespaces UNDOTBS2 UNDOTBS3 UNDOTBS4 ############
f16 () {

echo " "
echo " drop extra RAC-specific UNDO tablespaces UNDOTBS2 UNDOTBS3 UNDOTBS4"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
drop tablespace UNDOTBS2 including contents and datafiles;
drop tablespace UNDOTBS3 including contents and datafiles;
drop tablespace UNDOTBS4 including contents and datafiles;
SQLEOF

export f16_flag=`date`
}

############## Function f17 - reduce TEMP tablespaces in size to 4M AUTOEXTEND ON  NEXT 336M  MAXSIZE 32592M  ##############
f17 () {

echo " "
echo " reduce TEMP tablespaces"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
set pages 999
set lines 200
set feedback off
set heading off
spool /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17_temp_resize_A.sql
select 'alter DATABASE tempfile '''||file_name||''' resize 4m;' from dba_temp_files;
select 'alter DATABASE tempfile '''||file_name||''' AUTOEXTEND ON  NEXT 336M  MAXSIZE 32592M;' from dba_temp_files;
SQLEOF

egrep '^alter' /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17_temp_resize_A.sql > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17_temp_resize.sql
rm /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17_temp_resize_A.sql

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
spool /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17_temp_resize.log
@/mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17_temp_resize.sql
SQLEOF

export f17_flag=`date`
}

############## Function f17a - reduce UNDO tablespaces in size to 4M AUTOEXTEND ON  NEXT 336M  MAXSIZE 32592M  ##############
f17a () {
echo " currently does nothing - may in future"

#echo " "
#echo " reduce UNDO tablespaces"
#echo " "
#export ORACLE_SID="${CLONE_TGT}"
#export ORAENV_ASK="NO";. /usr/local/bin/oraenv

#$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
#connect / as sysdba
#set pages 999
#set lines 200
#set feedback off
#set heading off
#spool /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17a_undo_resize_A.sql
#select 'alter DATABASE datafile '''||name||''' resize 4m;' from v\$datafile where name like '%undo%';
#select 'alter DATABASE datafile '''||name||''' AUTOEXTEND ON  NEXT 336M  MAXSIZE 32592M;' from v\$datafile where name like '%undo%';
#SQLEOF

#egrep '^alter' /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17a_undo_resize_A.sql > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17a_undo_resize.sql
#rm /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17a_undo_resize_A.sql

#$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
#connect / as sysdba
#spool /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17a_undo_resize.log
#@/mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f17a_undo_resize.sql
#SQLEOF

export f17a_flag=`date`
}

############## Function f18 - remove RAC threads and associated LOGFILE Groups ##############
f18() {

echo " "
echo " remove RAC threads and associated LOGFILE Groups"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba

spool ${CLONE_TGT}_f18_thread_logfile_drop.log
ALTER DATABASE DISABLE THREAD 2;
ALTER DATABASE DISABLE THREAD 3;
ALTER DATABASE DISABLE THREAD 4;

ALTER DATABASE DROP LOGFILE GROUP 45;
ALTER DATABASE DROP LOGFILE GROUP 46;
ALTER DATABASE DROP LOGFILE GROUP 47;
ALTER DATABASE DROP LOGFILE GROUP 48;
ALTER DATABASE DROP LOGFILE GROUP 49;
ALTER DATABASE DROP LOGFILE GROUP 56;
ALTER DATABASE DROP LOGFILE GROUP 57;
ALTER DATABASE DROP LOGFILE GROUP 58;
ALTER DATABASE DROP LOGFILE GROUP 59;
ALTER DATABASE DROP LOGFILE GROUP 60;
ALTER DATABASE DROP LOGFILE GROUP 67;
ALTER DATABASE DROP LOGFILE GROUP 68;
ALTER DATABASE DROP LOGFILE GROUP 69;
ALTER DATABASE DROP LOGFILE GROUP 70;
ALTER DATABASE DROP LOGFILE GROUP 71;

SQLEOF

export f18_flag=`date`
}

############## Function f19 - cancel e-mail and workflow - approximately 10 minutes of apparent inactivity" ############
f19() {

echo " "
echo " cancel e-mail and workflow"
echo " this will sit at the SQL> SQL> SQL> prompt for approximately 10 minutes .. it IS executing"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

SQL_CONN_STR="APPS/"${APPS_PASS}

${ORACLE_HOME}/bin/sqlplus ${SQL_CONN_STR} << SQLEOF

spool ${CLONE_TGT}_f19_cancel_email_workflow.log
@/mount/PRODDBA/oracle_scripts/cloning/disable_emails.sql
@/mount/PRODDBA/oracle_scripts/cloning/cancel_wf_notifications.sql

SQLEOF

export f19_flag=`date`
}

############## Function f20 - rename datafile restored from Prod database ###########
f20() {

echo " "
echo " current PRODUCTION database file#=34 is named 'oddly' - using RMAN to fix the clone"
echo " "

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

export RMAN_COPY_COMMAND="copy datafile 34 to '+DATA_EXA${HOST_EXA_NUM_TGT}';"

echo $RMAN_COPY_COMMAND                                   > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_1

echo "sql \"alter tablespace XXCUSD_ARCHIVE offline\"; "  > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_2

echo "switch datafile 34 to copy;"                        > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_3
echo "recover tablespace XXCUSD_ARCHIVE;"                >> /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_3
echo "sql \"alter tablespace xxcusd_archive online\"; "  >> /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_3

echo "delete copy of datafile 34;"                        > /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_4

rman target / cmdfile /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_1
rman target / cmdfile /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_2
rman target / cmdfile /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_3
rman target / cmdfile /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f20_rman.rcv_4

ContYN
#############

export f20_flag=`date`
}

############## Function f21 - setup masking - logs in as SYSTEM using password from step 14 ###############
f21() {

echo " "
echo " setup masking - uses same password for SYSTEM set in step f14 above"
echo " "
export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv
export NEW_SYSTEM="${CLONE_LC_TGT}_eb21"

SQL_CONN_STR="SYSTEM/"${NEW_SYSTEM}

${ORACLE_HOME}/bin/sqlplus ${SQL_CONN_STR} << SQLEOF

spool ${CLONE_TGT}_f21_initialise_masking.log
@/mount/PRODDBA/oracle_scripts/cloning/dm_fmtlib_pkgdef.sql
@/mount/PRODDBA/oracle_scripts/cloning/dm_fmtlib_pkgbody.sql

-- #3. Check for GRP, flashback should be off
select flashback_on from v$database;

SQLEOF

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba

-- #4. Additional grants for ODM for system/apps
spool  ${CLONE_TGT}_f21_grants_masking.log

grant create any procedure to apps;
grant GRANT ANY OBJECT PRIVILEGE to apps;
grant GRANT ANY OBJECT PRIVILEGE to system;
grant execute any procedure to apps;
grant create any procedure to system;
grant execute any procedure to system;
grant exempt access policy to SYSTEM WITH ADMIN OPTION;
grant execute on sys.DBMS_CRYPTO to system;
grant execute on sys.DBMS_CRYPTO to APPS;
grant all on APPS.MGMT$MASK_DISABLEFLASHBACK to system;
grant Grant any object privilege to apps;

--#Note:  grant on APPS.MGMT$MASK_DISABLEFLASHBACK fails ORA-00942: table or view does not exist

-- #5. Additional grants for ODM for Poonam & Nitya database accounts
grant  execute on sys.DBMS_CRYPTO to CO51670,CO56391;
grant  execute on sys.DBMS_RANDOM to CO51670,CO56391;

SQLEOF

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
spool ${CLONE_TGT}_f21_ODM_password.log

--#6. Run password change script  This is for ODM .. Request from Poonam
@/mount/PRODDBA/oracle_scripts/cloning/change_passwd_after_clone.sql
SQLEOF

export f21_flag=`date`
}

############## Function f22 - reset the dba_directories from before clone capture file ##############
f22() {
echo " "
echo " .. reset the dba_directories from before clone capture file"
echo "    /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f02_directory_set.sql"
echo " "

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

$ORACLE_HOME/bin/sqlplus /nolog << SQLEOF
connect / as sysdba
spool /mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f22_directory_set.log
@/mount/PRODDBA/oracle_scripts/cloning/${CLONE_TGT}_f02_directory_set.sql
SQLEOF

export f22_flag=`date`
}

############## Function f23 - reset RMAN persistent parameters, run l0 backup, and prompt to "reset" OEM ############
f23() {
echo " "
echo " Reset RMAN persistent DEV parameters .. and increase for this run only to 8 channels"
echo " "

export ORACLE_SID="${CLONE_TGT}"
export ORAENV_ASK="NO";. /usr/local/bin/oraenv

### New RECOVERY WINDOW for DEV
### /mount/PRODDBA/oracle_scripts/backups/setup_rman.sh ${CLONE_ZFSSA_ID} single 1
/mount/PRODDBA/oracle_scripts/backups/setup_rman_dev.sh ${CLONE_ZFSSA_ID} single 1
ContYN

echo " - and now run a level 0 keep 21 days archive backup"
execution_date=`date +%Y%m%d`
export NLS_DATE_FORMAT="DD/MM/YYYY HH24:MI:SS"
echo " Enter Keep_Tag_Prefix for RMAN backup"
read KEEP_TAG_PREFIX
export keep_tag="${KEEP_TAG_PREFIX}_${execution_date}"
date > $keep_tag.txt
echo $keep_tag >> $keep_tag.txt
cat  $keep_tag.txt
echo " "
echo " backup log file in $ORACLE_HOME/dbs/${keep_tag}_backup.log"
echo " backup placed into $CLONE_BACKUP_TARGET"

rman target / log=${ORACLE_HOME}/dbs/${keep_tag}_backup.log <<RMANEOF

CONFIGURE DEVICE TYPE DISK PARALLELISM 8 BACKUP TYPE TO COMPRESSED BACKUPSET;

run {
ALLOCATE CHANNEL c1 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c2 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c3 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c4 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c5 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c6 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c7 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;
ALLOCATE CHANNEL c8 DEVICE TYPE DISK FORMAT   '$CLONE_BACKUP_TARGET/%T_%U' MAXPIECESIZE 32 G;


show all;

set command id to 'Database Backup L0 KEEP';

backup as compressed backupset filesperset 1 incremental level 0 database TAG "$keep_tag" keep until time 'SYSDATE+28' force;
}

CONFIGURE DEVICE TYPE DISK PARALLELISM 4 BACKUP TYPE TO COMPRESSED BACKUPSET;
RMANEOF

tail -1 $ORACLE_HOME/dbs/${keep_tag}_backup.log >> $keep_tag.txt
date >> $keep_tag.txt
cat $keep_tag.txt

echo "PROMPT - edit OEM Monitoring configuration - and Test connection for dbsnmp as sysdba - then Save - setup should be retained from 'old' database"
export f23_flag=`date`
}

############## Function f23a - put out passwords ############
f23a() {
echo " "
echo " The passwords below should be forwarded to the EBS Support team"
echo " APPS password  : " ${APPS_PASS}
echo " SYSTEM Password: " ${CLONE_LC_TGT}"_eb21"
echo " "
export f23a_flag=`date`
}

###################### Main Logic ######################################

# check that we are running this script from oracle so that the su will work.
WHOAMI=`whoami`
if [ ${WHOAMI} != oracle ]
then
   echo "need to run this script from oracle"
   exit
fi

# Retrieve cloning candidates on this node
extract_cloning_candidates

# Retrieve variables from clone_config.txt extracted by above function
get_target_db

# Prompt User to supply the APPS password - this is mandatory
get_APPS_pwd

CHOOSE="G"
while [ ${CHOOSE} != "Q" -a ${CHOOSE} != "q" ]
do
  echo " "
  echo "Script: $0"
  echo "Config File:" $CONFIGG
  echo "Source:" $HOSTNAM_SRC $CLONE_SRC_DIR
  echo "Target:" $HOSTNAM_TGT $CLONE_TGT
  echo "-------------------------------------------------"
  echo "work thru these in sequence:"
  echo "2.   capture service/domain/dba_dir      * " $f02_flag
  echo "3.   create pfile / add convert parms    * " $f03_flag
  echo "4.   delete RMAN backups                 * " $f04_flag
  echo "5.   stop listener ${CLONE_TGT}              * " $f05_flag
  echo "6.   clear zfssa directory               * " $f06_flag
  echo "6a.  shutdown database ${CLONE_TGT}          * " $f06a_flag
  echo "6b.  delete database files from ASM      * " $f06b_flag
  echo "7.   duplicate database                  * " $f07_flag
  echo "7a.  recreate block change tracking      * " $f07a_flag
  echo "8b.  recreate pfile and restart with control file * " $f08b_flag
  echo "9.   create another spfile & restart     * " $f09_flag
  echo "10.  drop trigger & restart              * " $f10_flag
  echo "11.  fnd_node_clean via APPS user        * " $f11_flag
  echo "12.  start listener ${CLONE_TGT}             * " $f12_flag
  echo "13.  execute adautocfg                   * " $f13_flag
  echo "14.  reset passwords                     * " $f14_flag
  echo "      ## Perform steps 16-18 when cloning into NON-RAC DB ##"
  echo "16.  drop UNDO tbspaces for RAC Inst 2-4 * " $f16_flag
  echo "17.  resize TEMP tablespace              * " $f17_flag
# echo "17a. resize UNDO tablespace              * " $f17a_flag;
  echo "18.  remove RAC threads and LOG Groups   * " $f18_flag
  echo "      ## For ALL Production refreshes ##       "
  echo "19.  cancel e-mail and workflow          * " $f19_flag
  echo "      ## Optional for Masking ##         "
  echo "21.  setup masking                       * " $f21_flag
  echo "      ## For ALL refreshes ## "
  echo "22.  reset dba_directories for $CLONE_TGT  * " $f22_flag
  echo "23.  setup rman parms & take backup      * " $f23_flag
  echo "23a. echo passwords for passing to EBS Support * " $f23a_flag

  echo " ${CLONE_TGT}: Last Option (${LAST_CHOICE}) - Select option Q or q to quit"
  read CHOOSE
  LAST_CHOICE=$CHOOSE
# empty string input so try again
  if [ -z ${CHOOSE} ]
  then
    CHOOSE=1
  fi
  case  $CHOOSE in
          2) f02;;
          3) f03;;
          4) f04;;
          5) f05;;
          6) f06;;
         6a) f06a;;
         6b) f06b;;
          7) f07;;
         7a) f07a;;
         8b) f08b;;
          9) f09;;
         10) f10;;
         11) f11;;
         12) f12;;
         13) f13;;
         14) f14;;
         16) f16;;
         17) f17;;
#       17a) f17a;;
         18) f18;;
         19) f19;;
         21) f21;;
         22) f22;;
         23) f23;;
        23a) f23a;;
          Q) echo "";;
          q) echo "";;
          *) echo "Invalid option"
  esac
done

[oracle@exa5dbadm03 ~]$
