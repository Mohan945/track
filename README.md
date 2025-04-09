[oracle@slglnbby ~]$ ls -lrt Clone_Jobs_Create_Next_EB21_TZ02_TZ21_EB37.sh Clone_Jobs_Create_EB21_TZ02_TZ21_EB37.sh /mount/PRODDBA/oracle_scripts/general/clone_generate_scp_gzip.txt
-rwxr-xr-x 1 oracle oinstall 12133 Dec 27 11:49 Clone_Jobs_Create_Next_EB21_TZ02_TZ21_EB37.sh
-rwxr-xr-x 1 oracle oinstall  2712 Dec 27 11:49 Clone_Jobs_Create_EB21_TZ02_TZ21_EB37.sh
-rwxr-xr-x 1 oracle ext_dba   1905 Apr  9 08:14 /mount/PRODDBA/oracle_scripts/general/clone_generate_scp_gzip.txt
[oracle@slglnbby ~]$ cat /mount/PRODDBA/oracle_scripts/general/clone_generate_scp_gzip.txt
Instance;BSN;BackUp Date Time;T; Source    ; Source Backup Directory                  ; Target Refresh Directory        ;SCP Date Time   ;Pri ;Clo ;Target Host
EB21PROD;BSN;2024-12-29 20:00;A;exa8dbadm02;/zfssa/EXA8_EB21NPRO/backup1/EBS_CLONE_CH ;/zfssa/EBS_CLONE/backup1/EB21PROD;2024-12-30 00:00;LEG8;LEG5;exa5dbadm01
TZ02NPRO;BSN;2024-12-29 20:00;A;exa8dbadm01;/zfssa/EXA8_TZ02NPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/TZ02PROD;2024-12-30 03:00;LEG8;LEG5;exa5dbadm01
EB21PROD;BSN;2025-03-30 20:00;A;exa7dbadm02;/zfssa/EXA7_EB21GPRO/backup1/EBS_CLONE_CH ;/zfssa/EBS_CLONE/backup1/EB21PROD;2025-03-31 00:00;LEG7;LEG5;exa5dbadm01
TZ02GPRO;BSN;2025-03-30 20:00;A;exa7dbadm01;/zfssa/EXA7_TZ02GPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/TZ02PROD;2025-03-31 03:00;LEG7;LEG5;exa5dbadm01
EB37NPRO;ESN;2021-01-24 20:00;A;exa8dbadm02;/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD ;n/a                              ;2021-01-25 00:00;LEG8;LEG5;exa5dbadm01
EB37GPRO;ESN;2025-04-13 15:00;A;exa7dbadm02;/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD ;n/a                              ;2025-04-14 00:00;LEG7;LEG5;exa5dbadm01
EB22SPRO;BSN;2019-12-15 22:15;2;exa4db02   ;/zfssa/EXA4_EB22SPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/EB22PROD;22:15;00:00;LEG7;LEG5;exa5dbadm01
EB25SPRO;BSN;2019-12-15 22:15;2;exa4db02   ;/zfssa/EXA4_EB25SPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/EB25PROD;22:15;00:00;LEG7;LEG5;exa5dbadm01
EB21SYIT;BGE;2018-01-09 22:15;2;exa1db03   ;/zfssa/EBS_CLONE/backup1/EB21SYIT/POSTMASK;/zfssa/EBS_CLONE/backup1/EB21SYIT;22:15;00:00;LEG5;LEG5;exa5dbadm01
TZ21NPRO;BSN;2021-03-05 20:00;A;exa8dbadm03;/zfssa/EXA8_TZ21NPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/TZ21PROD;2021-03-05 00:00;LEG8;LEG5;exa5dbadm01
CI01GPRO;BSN;2021-05-30 20:00;A;exa7dbadm01;/zfssa/EXA7_CI01GPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/CI01PROD;2021-05-31 03:00;LEG7;LEG5;exa5dbadm01
[oracle@slglnbby ~]$ cat Clone_Jobs_Create_Next_EB21_TZ02_TZ21_EB37.sh
#!/bin/sh
# Script : Clone_Jobs_Create_Next_EB21_TZ02_EB37.sh
# Author : David Green SL093R 26 March 2020
# Version: 1
# Amended: David Green SL093R 15 January 2021
#          Added code to generate the expdp for EB37
#
if [ $# -ne 2 ] ; then
  echo "$0:CHG Number and single clone selection parameter required to select record from /mount/PRODDBA/oracle_scripts/general/clone_generate_scp_gzip.txt"
  exit 8
fi
change_entered=$1
clone_entered=$2
TGT_NO=`expr ${clone_entered} + 1`

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

########## Function echoVars: display the current values retrieved/being used ##########
echoVars () {
  echo "TGT_LINE                   :$TGT_LINE"
  echo "SELECTED_CLONE             :$SELECTED_CLONE"
  echo "SELECTED_BACKUP_SOURCE     :$SELECTED_BACKUP_SOURCE"
  echo "SELECTED_SCP_SLOT1_HOUR    :$SELECTED_SCP_SLOT1_HOUR"
  echo "SELECTED_SCP_SLOT1_MINS    :$SELECTED_SCP_SLOT1_MINS"
}
########## Function generate_cloning_jobs_par_files: only show clone candidates from this compute node ##########
generate_cloning_jobs_par_files () {
  export CONFIGG="/mount/PRODDBA/oracle_scripts/general/clone_generate_scp_gzip.txt"
  echo "Original Config file: " ${CONFIGG}

#Instance;BSN;BackUp Date Time;T; Source    ; Source Backup Directory                  ; Target Refresh Directory        ;SCP Date Time   ;Pri ;Clo ;Target Host
#EB21PROD;BSN;2020-12-20 20:00;A;exa8dbadm02;/zfssa/EXA8_EB21NPRO/backup1/EBS_CLONE_CH ;/zfssa/EBS_CLONE/backup1/EB21PROD;2020-12-21 00:00;LEG8;LEG5;exa5dbadm01
#TZ02NPRO;BSN;2020-12-20 20:00;A;exa8dbadm01;/zfssa/EXA8_TZ02NPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/TZ02PROD;2020-12-21 03:00;LEG8;LEG5;exa5dbadm01
#EB21PROD;BSN;2020-12-06 20:00;A;exa7dbadm02;/zfssa/EXA7_EB21GPRO/backup1/EBS_CLONE_CH ;/zfssa/EBS_CLONE/backup1/EB21PROD;2020-12-07 00:00;LEG7;LEG5;exa5dbadm01
#TZ02GPRO;BSN;2020-12-06 20:00;A;exa7dbadm01;/zfssa/EXA7_TZ02GPRO/backup1/CLONE_CH     ;/zfssa/EBS_CLONE/backup1/TZ02PROD;2020-12-07 03:00;LEG7;LEG5;exa5dbadm01
#EB37NPRO;ESN;2021-01-24 20:00;A;exa8dbadm02;/mount/PRODDBA/oracle/DP_EXPORTS/EB37PROD ;n/a                              ;2021-01-25 00:00;LEG8;LEG5;exa5dbadm01

  TGT_LINE=`head -${TGT_NO} ${CONFIGG}|tail -1`
  SELECTED_CLONE=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $1} END {}'`
  SELECTED_UC4_DB=`echo ${SELECTED_CLONE} | awk 'BEGIN { FS=";" } {print substr($1,1,4)} END {}' | tr '[:lower:]' '[:upper:]'`
  SELECTED_BACKUP_SCP=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $2} END {}'`
  SELECTED_BACKUP_DATE_TIME=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $3} END {}'`
  SELECTED_BACKUP_SOURCE=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $5} END {}'`
  SELECTED_UC4_SRC=`echo ${SELECTED_BACKUP_SOURCE} | awk 'BEGIN { FS=";" } {print substr($1,1,4)} END {}' | tr '[:lower:]' '[:upper:]'`
  SELECTED_BACKUP_DIR=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print $6} END {}'`

  case $SELECTED_UC4_DB in
    EB21|TZ02|TZ21)
  SELECTED_SCP_DATE=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print substr($8,1,10)} END {}'`
  SELECTED_SCP_SLOT1_HOUR=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print substr($8,12,2)} END {}'`
  SELECTED_SCP_SLOT1_MINS=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print substr($8,15,2)} END {}'`
# echoVars

# calculate additional slots for scp execution
  SELECTED_SCP_SLOT2_HOUR=`printf %02d "$(($SELECTED_SCP_SLOT1_HOUR + 1))"`
  SELECTED_SCP_SLOT2_MINS=`printf %02d "$(($SELECTED_SCP_SLOT1_MINS + 30))"`
  SELECTED_SCP_SLOT3_HOUR=`printf %02d "$(($SELECTED_SCP_SLOT1_HOUR + 2))"`
  SELECTED_SCP_SLOT3_MINS=$SELECTED_SCP_SLOT1_MINS
  SELECTED_SCP_SLOT1="${SELECTED_SCP_DATE} ${SELECTED_SCP_SLOT1_HOUR}:${SELECTED_SCP_SLOT1_MINS}"
  SELECTED_SCP_SLOT2="${SELECTED_SCP_DATE} ${SELECTED_SCP_SLOT1_HOUR}:${SELECTED_SCP_SLOT2_MINS}"
  SELECTED_SCP_SLOT3="${SELECTED_SCP_DATE} ${SELECTED_SCP_SLOT2_HOUR}:${SELECTED_SCP_SLOT1_MINS}"
  SELECTED_SCP_SLOT4="${SELECTED_SCP_DATE} ${SELECTED_SCP_SLOT2_HOUR}:${SELECTED_SCP_SLOT2_MINS}"
  SELECTED_SCP_SLOT5="${SELECTED_SCP_DATE} ${SELECTED_SCP_SLOT3_HOUR}:${SELECTED_SCP_SLOT1_MINS}"
  SELECTED_SCP_SLOT6="${SELECTED_SCP_DATE} ${SELECTED_SCP_SLOT3_HOUR}:${SELECTED_SCP_SLOT2_MINS}"

# echo "slots " $SELECTED_SCP_SLOT1_HOUR $SELECTED_SCP_SLOT1_MINS $SELECTED_SCP_SLOT1_HOUR $SELECTED_SCP_SLOT2_MINS $SELECTED_SCP_SLOT2_HOUR $SELECTED_SCP_SLOT1_MINS $SELECTED_SCP_SLOT2_HOUR $SELECTED_SCP_SLOT2_MINS $SELECTED_SCP_SLOT3_HOUR $SELECTED_SCP_SLOT1_MINS $SELECTED_SCP_SLOT3_HOUR $SELECTED_SCP_SLOT2_MINS
# echo $SELECTED_SCP_SLOT1 $SELECTED_SCP_SLOT2 $SELECTED_SCP_SLOT3 $SELECTED_SCP_SLOT4 $SELECTED_SCP_SLOT5 $SELECTED_SCP_SLOT6

  SELECTED_CLONE_TARGET=`echo ${TGT_LINE} | awk 'BEGIN {FS=";"} {print substr($11,1,10)} END {}'`

  SELECTED_TARGET_LOW_1=$SELECTED_CLONE_TARGET"1"
  SELECTED_TARGET_LOW_2=$SELECTED_CLONE_TARGET"2"
  SELECTED_TARGET_LOW_3=$SELECTED_CLONE_TARGET"3"

  echo "s/SED_DB4_CAPS/$SELECTED_UC4_DB/g"                                                            > Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s-SED_DEF_BACK_CMD-/mount/PRODDBA/oracle_scripts/cloning/exa_clone_backup_gen_scp_X8.sh-g"   >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s-SED_DEF_SCP_CMD-/mount/PRODDBA/oracle_scripts/cloning/exa_clone_exec_scp_gunzip.sh-g"      >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CLONE_SELECTOR/$clone_entered/g"                                                       >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CHANGE/$change_entered/g"                                                              >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SRC4_CAPS/$SELECTED_UC4_SRC/g"                                                         >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_DB8_CAPS/$SELECTED_CLONE/g"                                                            >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SOURCE_LOW/$SELECTED_BACKUP_SOURCE/g"                                                  >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_BACKUP_START_TIME/$SELECTED_BACKUP_DATE_TIME/g"                                        >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_TARGET_LOW_1/$SELECTED_TARGET_LOW_1/g"                                                 >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_TARGET_LOW_2/$SELECTED_TARGET_LOW_2/g"                                                 >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_TARGET_LOW_3/$SELECTED_TARGET_LOW_3/g"                                                 >> Clone_EB21_TZ02_TZ21_Backup_sed_file

  echo "s/SED_CLONE_SLOT_1/51/g"                                                                     >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CLONE_SLOT_2/52/g"                                                                     >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CLONE_SLOT_3/53/g"                                                                     >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CLONE_SLOT_4/54/g"                                                                     >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CLONE_SLOT_5/55/g"                                                                     >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_CLONE_SLOT_6/56/g"                                                                     >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SCP_SLOT_1/$SELECTED_SCP_SLOT1/g"                                                      >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SCP_SLOT_2/$SELECTED_SCP_SLOT2/g"                                                      >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SCP_SLOT_3/$SELECTED_SCP_SLOT3/g"                                                      >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SCP_SLOT_4/$SELECTED_SCP_SLOT4/g"                                                      >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SCP_SLOT_5/$SELECTED_SCP_SLOT5/g"                                                      >> Clone_EB21_TZ02_TZ21_Backup_sed_file
  echo "s/SED_SCP_SLOT_6/$SELECTED_SCP_SLOT6/g"                                                      >> Clone_EB21_TZ02_TZ21_Backup_sed_file

# Generate the Clone_Backup_Job parameter file for parameter supplied
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_Backup_Job_EB21_TZ02_TZ21.sed > Clone_Backup_Job_${SELECTED_UC4_DB}.txt
# Generate the Clone_SCP_Job parameter files for parameter supplied
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_SCP_EB21_TZ02_TZ21_51.sed > Clone_SCP_${SELECTED_UC4_DB}_51.txt
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_SCP_EB21_TZ02_TZ21_52.sed > Clone_SCP_${SELECTED_UC4_DB}_52.txt
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_SCP_EB21_TZ02_TZ21_53.sed > Clone_SCP_${SELECTED_UC4_DB}_53.txt
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_SCP_EB21_TZ02_TZ21_54.sed > Clone_SCP_${SELECTED_UC4_DB}_54.txt
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_SCP_EB21_TZ02_TZ21_55.sed > Clone_SCP_${SELECTED_UC4_DB}_55.txt
  sed -f Clone_EB21_TZ02_TZ21_Backup_sed_file Clone_SCP_EB21_TZ02_TZ21_56.sed > Clone_SCP_${SELECTED_UC4_DB}_56.txt

  echo "+ + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + "
  echoVars
  echo "Be aware: Current Selected BSN Status : " $SELECTED_BACKUP_SCP " - ensure (B)ackup and (S)cp are set!!"
  echo "+ + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + "
  ContYN
      ;;
    EB37)
      echo "EB37 selected"
echo "expdp system FULL=N DIRECTORY=DATA_PUMP_DIR dumpfile=EB37PROD_20092020_release_1_%U.dmp schemas=DRM_PROD FLASHBACK_TIME=\"TO_TIMESTAMP\(TO_CHAR\(SYSDATE,\'YYYY-MM-DD HH24:MI:SS\'\),\'YYYY-MM-DD HH24:MI:SS\'\)\"  FILESIZE=15G PARALLEL=4  trace=480300 logfile=EB37PROD_20092020_release.log "
  echo "s/SED_DB4_CAPS/$SELECTED_UC4_DB/g"                                                             > Clone_EB37_EXPDP_sed_file
  echo "s-SED_DEF_EXPDP_CMD-/mount/PRODDBA/oracle_scripts/cloning/exa_clone_expdp_gen_X8.sh-g"        >> Clone_EB37_EXPDP_sed_file
  echo "s/SED_CLONE_SELECTOR/$clone_entered/g"                                                        >> Clone_EB37_EXPDP_sed_file
  echo "s/SED_CHANGE/$change_entered/g"                                                               >> Clone_EB37_EXPDP_sed_file
  echo "s/SED_SRC4_CAPS/$SELECTED_UC4_SRC/g"                                                          >> Clone_EB37_EXPDP_sed_file
  echo "s/SED_DB8_CAPS/$SELECTED_CLONE/g"                                                             >> Clone_EB37_EXPDP_sed_file
  echo "s/SED_SOURCE_LOW/$SELECTED_BACKUP_SOURCE/g"                                                   >> Clone_EB37_EXPDP_sed_file
  echo "s/SED_BACKUP_START_TIME/$SELECTED_BACKUP_DATE_TIME/g"                                         >> Clone_EB37_EXPDP_sed_file

# Generate the expdp parameter file to be used for this run
# sed -f  Clone_EB37_EXPDP_sed_file  Clone_EB37_EXPDP_parm_file.sed > Clone_EB37_EXPDP_parm_file.par
# Generate the Clone_EXPDP_Job parameter file for parameter supplied
  sed -f Clone_EB37_EXPDP_sed_file Clone_EXPDP_Job_EB37.sed > Clone_EXPDP_Job_${SELECTED_UC4_DB}.txt

  echo "+ + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + "
  echoVars
  echo "Be aware: Current Selected ESN Status : " $SELECTED_BACKUP_SCP " - ensure (E)xpdp is set!!"
  echo "+ + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + "
  ContYN
      ;;
  esac

}
#========================================================================================

generate_cloning_jobs_par_files
echo " Files created/used during the execution of this family of jobs"
ls -lrt Clone*|egrep -v 'sed|CHG'
[oracle@slglnbby ~]$ cat Clone_Jobs_Create_EB21_TZ02_TZ21_EB37.sh
#!/bin/sh
# Script : Clone_Jobs_Create_EB21_TZ02_TZ21_EB37.sh
# Author : David Green SL093R 23 April 2020
# Version: 1

if [ $# -ne 2 ] ; then
  echo "$0:CHG Number and single clone selection parameter EB21|TZ02|TZ21|EB37 required to select jobs to be scheduled"
  exit 8
fi
change_entered=$1
clone_entered=$2

OEM_USERNAME=TPS_ORACLE_SCHEDULER
OEM_PASSWORD=`cat ~/TOS.pwd`

export ORAENV_ASK=NO
export ORACLE_SID=OMS135; . oraenv

emcli login -username=${OEM_USERNAME} -password=${OEM_PASSWORD}

case $clone_entered in
  EB21)
        emcli create_job -input_file=property_file:Clone_Backup_Job_EB21.txt;
        emcli create_job -input_file=property_file:Clone_SCP_EB21_51.txt;
        emcli create_job -input_file=property_file:Clone_SCP_EB21_52.txt;
        emcli create_job -input_file=property_file:Clone_SCP_EB21_53.txt;
        emcli create_job -input_file=property_file:Clone_SCP_EB21_54.txt;
        emcli create_job -input_file=property_file:Clone_SCP_EB21_55.txt;
        emcli create_job -input_file=property_file:Clone_SCP_EB21_56.txt;
        echo "EB21 selected";;
  TZ02)
        emcli create_job -input_file=property_file:Clone_Backup_Job_TZ02.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ02_51.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ02_52.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ02_53.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ02_54.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ02_55.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ02_56.txt;
        echo "TZ02 selected";;
  TZ21)
        emcli create_job -input_file=property_file:Clone_Backup_Job_TZ21.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ21_51.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ21_52.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ21_53.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ21_54.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ21_55.txt;
        emcli create_job -input_file=property_file:Clone_SCP_TZ21_56.txt;
        echo "TZ21 selected";;
  EB37)
        echo "EB37 selected";;

     *) echo "Invalid option"
esac

emcli get_jobs -status_ids=1 | egrep $change_entered > Clone_EB21_TZ02_TZ21_EB37_${change_entered}_jobs.txt
echo "+ + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + +"
cat Clone_EB21_TZ02_TZ21_EB37_${change_entered}_jobs.txt
echo "+ + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + + +"

emcli logout
[oracle@slglnbby ~]$ cat Clone_EXPDP_Job_EB37.sed
# Job Parameter file to create the cloning job for SED_DB4_CAPS EXPDP Export DataPump
# The variable.default_shell_command is filled in as below
##command=SED_DEF_EXPDP_CMD SED_CLONE_SELECTOR { EB37 5 for exa7 | EB37 6 for exa8 }

name=EXA_EXPDP_GEN_SED_CHANGE_SED_DB4_CAPS_SED_SRC4_CAPS

# Job Type.
type=OSCommand

# Job Description.
description='EXPDP for SED_DB8_CAPS from SED_SOURCE_LOW: SED_CHANGE'

# Job Owner.
owner=sysman

# Kind of Job.
kind=active

# Target List.
target_list=SED_SOURCE_LOW.corp.standardlife.com:host

# Target Criteria.
targetType=host

# Credential List.
cred.defaultHostCred.<all_targets>:host=NAMED:SYSMAN:HOST_OEM13C_ORACLE_SSH

# Description: (Optional) The complete command line, including parameters.
# This text will appear as the command if this job is edited in the UI.
# This command will be run without a shell.
variable.default_shell_command=SED_DEF_BACK_CMD

# Schedule.
schedule.frequency=ONCE
schedule.startTime=SED_BACKUP_START_TIME
schedule.timezone.type=TIMEZONE_TARGET
schedule.timezone.targetIndex=1

# Notify Rules.
notification=ACTION_REQUIRED, PROBLEMS

[oracle@slglnbby ~]$ cat Clone_EXPDP_Job_EB37.txt
# Job Parameter file to create the cloning job for EB37 EXPDP Export DataPump
# The variable.default_shell_command is filled in as below
##command=/mount/PRODDBA/oracle_scripts/cloning/exa_clone_expdp_gen_X8.sh 6 { EB37 5 for exa7 | EB37 6 for exa8 }

name=EXA_EXPDP_GEN_CHG0406216_EB37_EXA7

# Job Type.
type=OSCommand

# Job Description.
description='EXPDP for EB37GPRO from exa7dbadm02: CHG0406216'

# Job Owner.
owner=sysman

# Kind of Job.
kind=active

# Target List.
target_list=exa7dbadm02.corp.standardlife.com:host

# Target Criteria.
targetType=host

# Credential List.
cred.defaultHostCred.<all_targets>:host=NAMED:SYSMAN:HOST_OEM13C_ORACLE_SSH

# Description: (Optional) The complete command line, including parameters.
# This text will appear as the command if this job is edited in the UI.
# This command will be run without a shell.
variable.default_shell_command=SED_DEF_BACK_CMD

# Schedule.
schedule.frequency=ONCE
schedule.startTime=2025-04-13 15:00
schedule.timezone.type=TIMEZONE_TARGET
schedule.timezone.targetIndex=1

# Notify Rules.
notification=ACTION_REQUIRED, PROBLEMS
[oracle@slglnbby ~]$ cat Clone_EB21_TZ02_TZ21_EB37_CHG0406216_jobs.txt



I am not able to create job for EB37 why ?
