#!/bin/ksh
# Note guarantees that ksh will be used.
# Full command path is /home/sysadmin/Admin_Leaver
#----------------------------------------------------------------------------
# Author        : David Green
# Date          : 29th December 2011
# Calls         : none
# Called By     : Command line
# Description   : This script will create two files, and echo the contents of the
#                 /home/sysadmin/UserAdmin_Steer file containing any nominated
#                 local Systems Admin Oracle Users records.
# Files created : A Lock_Account script, and a Drop_Account script
# Amended       : David Green SL093R 17/09/2020
#                 Verification added for Leavers if they have any %Account% accounts, including OPS$ DBA DBD accounts - and locks them
#                 Verification added for Movers if they have OPS$ DBA DBD accounts - and locks them - irrespective of reason for Move
#---------------------------------------------------------------------------i
if [ $# -ne 1 ] ; then
  echo "$0:Please supply 1 input parameters: Report Date in format ddmmyyyy"
  exit 8
fi
#----------------------------------------------------------------------------
lock_list_target_file='Lock_List_'$1
cp /dev/null $lock_list_target_file

# Create a File of LeaverModify Users
cp /dev/null UserAdmin_Steer_Leaver_egrep_file

# egrep using reasons shown for Leavers - no Cut -d' ' as default tab used as field separators in "cut and paste" from Staff Leavers Excel worksheet
#Employee ID     Person Number   Name    Department      Details
#SL872K  1001860 Ian Brown       112112 Admin App Centre  Redundancy
# egrep using reasons shown for Leavers - no Cut -d' ' as default tab used as field separators in "cut and paste" from Staff Leavers Mail
#ID         Name                  Dept                 Details
#CO66408    Ryan Leslie           Platform Contact Cen ACTION - EMPLOYEE HAS LEFT

egrep 'Redundancy|Resignation|Retirement|Termination|EMPLOYEE HAS LEFT' Mail_Excel_csv.txt|cut -f 1,3,4,5  >> UserAdmin_Steer_Leaver_egrep_file
egrep 'EMPLOYEE HAS LEFT'                             Mail.txt          |cut -f1         >> UserAdmin_Steer_Leaver_egrep_file
#cat UserAdmin_Steer_Leaver_egrep_file

Leaver_Code=`egrep -v 'Redundancy|Resignation|Retirement|Termination|EMPLOYEE HAS LEFT' Mail_Excel_csv.txt|grep -v "^ "|wc -l`
if [ ${Leaver_Code} <> 0 ]; then
  echo "Leaver_Code: " $Leaver_Code
  echo "New Leaver Code in Mail_Excel_csv.txt file - not in expected list"
fi

echo "set heading off"            > /home/sysadmin/Lock_List_Query.sql
echo "set verify off"            >> /home/sysadmin/Lock_List_Query.sql
echo "set feedback off"          >> /home/sysadmin/Lock_List_Query.sql
echo "column user_q format a200" >> /home/sysadmin/Lock_List_Query.sql
echo "set linesize 300"          >> /home/sysadmin/Lock_List_Query.sql

#  Process the Mail.txt file again - and this time create a Lock_List file of Leavers AND Movers
cat UserAdmin_Steer_Leaver_egrep_file | while read LINE
do
  WHICH_ONE=`echo $LINE| awk '{print $NF}'`
# echo "WHICH" $WHICH_ONE
  if [ "$WHICH_ONE" == "LEFT" ] ; then    # from Mail.txt
    STEERING_ACCOUNT=`echo $LINE | cut -d' ' -f1`
    STEERING_NAME=`echo $LINE | cut -d' ' -f2-4`
    echo "LOCK:ALL_INST:"$STEERING_ACCOUNT":l0_ck_3d:"$STEERING_NAME":LEAVERLOCK::::::::7:$1|" >>$lock_list_target_file
    echo "select rtrim('LOCK:ALL_INST:'||rtrim(o.username)||':l0_ck_3d:""$STEERING_NAME"":LEAVERLOCK::::::::7:$1|') user_q FROM dba_users o WHERE (o.username like '%$STEERING_ACCOUNT%') or (o.username like 'OPS$%$STEERING_ACCOUNT%') or (o.username like 'DBA%$STEERING_ACCOUNT%') or (o.username like 'DBD%$STEERING_ACCOUNT%');" >> /home/sysadmin/Lock_List_Query.sql
  else                                    # from Mail_Excel_csv.txt
    STEERING_ACCOUNT=`echo $LINE | cut -d' ' -f 1`
    STEERING_NAME=`echo $LINE | cut -d ' ' -f 2-3`
    echo "LOCK:ALL_INST:"$STEERING_ACCOUNT":l0_ck_3d:"$STEERING_NAME":LEAVERLOCK::::::::7:$1|" >>$lock_list_target_file
    echo "select rtrim('LOCK:ALL_INST:'||rtrim(o.username)||':l0_ck_3d:""$STEERING_NAME"":LEAVERLOCK::::::::7:$1|') user_q FROM dba_users o WHERE (o.username like '%$STEERING_ACCOUNT%') or (o.username like 'OPS$%$STEERING_ACCOUNT%') or (o.username like 'DBA%$STEERING_ACCOUNT%') or (o.username like 'DBD%$STEERING_ACCOUNT%');" >> /home/sysadmin/Lock_List_Query.sql
  fi
done
echo "--"$lock_list_target_file `wc -m $lock_list_target_file` "characters - candidate Lock List"
cat $lock_list_target_file

#
# test whether any Prospective Oracle LEAVER/MOVER accounts exists in the Mail.txt or Mail_Excel_csv.txt file |  or is 0 bytes - exit
# if exists executes a new variant of the Ora_Lock_List script - to accomodate the %Account% theory
if [ -s $lock_list_target_file ] ; then
  target_file_sql='Lock_List_'$1'.sql'
  /oem/oracle/software_installs/server_scripts/Ora_Lock_List_All_dg ALL_INST $lock_list_target_file $target_file_sql
else
  echo "Either file " $source_file " does not exist or is 0 bytes long ";ls -l $lock_list_target_file
  date
  exit
fi

# Moved this here after the execution of the lock script - this checks whether any of the Leaver/Movers exist in the UserAdmin_Steer file
#
egrep -i -f UserAdmin_Steer_LeaverModify_egrep_file UserAdmin_Steer > Admin_Leaver_log

if [ -s Admin_Leaver_log ] ; then
  echo "/home/sysadmin/UserAdmin_Steer records need amending"
  cat Admin_Leaver_log
else
  echo "No Users present in the User_Admin_Steer file and, therefore, no action required"
fi

if [ -s All_Lock_List_$1.log  ] ; then
  echo " User amendments from All_Lock_List_"$1".log"
  egrep 'altered' All_Lock_List_$1.log
fi
