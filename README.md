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
[oracle@slglnbby ~]$ ls -lrt
