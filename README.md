#!/bin/sh
# Script : Process_AccessAble_Returns.sh
# Author : David Green SL093R 01 December 2021
# Version: 1
#

# Transform the "Save As csv" extracted from Excel into record based structure - fields with commas have double-quotes - awk struggles to process.
# Original File format
#dbms,staff_name,db_login_id,staff_id,database_name,application_name,action_required,decisioner,comments
#ORACLE,Gillian McKenna,SLD201,SLD201,BM04PROD,BPM,Remediated,Derek Maciver,Was required for a specific project but has now moved off that project so no longer needed.
#ORACLE,Dhillee Chelichemala,CO55289,CO55289,CD01PROD,DB SYNC,Remediated,Dave Campbell,Doesn't appear to be on the system
#ORACLE,Margaret Hobbins,CO64582,CO64582,CI01PROD,CUSTOMER INTERACTION,Remediated,Liam Kane,No need for now as database is migrated
# awk reformatted file
#Record 2:
#   $1=<ORACLE>
#   $2=<Gillian McKenna>
#   $3=<SLD201>
#   $4=<SLD201>
#   $5=<BM04PROD>
#   $6=<BPM>
#   $7=<Remediated>
#   $8=<Derek Maciver>
#   $9=<Was required for a specific project but has now moved off that project so no longer needed.>
#----

#awk -f Transform_csv.awk /mount/PRODDBA/oracle_scripts/recert/oracle_Recertifications_Results_Issue_AccessAble_Extract.csv > /mount/PRODDBA/oracle_scripts/recert/oracle_Recertifications_Results_Issue_AccessAble_Extract.records
#awk -f Transform_csv.awk /mount/PRODDBA/oracle_scripts/recert/bbb.txt > /mount/PRODDBA/oracle_scripts/recert/oracle_Recertifications_Results_Issue_AccessAble_Extract.records
egrep -v 'dbms|ORACLE' "/mount/PRODDBA/oracle_scripts/recert/Line Manager Certification for DB Platforms - May 2025.csv" | egrep "Remediated" | awk -f Transform_csv.awk > /mount/PRODDBA/oracle_scripts/recert/oracle_Recertifications_Results_Issue_AccessAble_Extract.records

# remove all previous runs files
rm *Returns.sql *record.sed
echo "/mount/PRODDBA/oracle_scripts/recert/oracle_Recertifications_Results_Issue_AccessAble_Extract.records"

########## Function echoVars: display the current values retrieved/being used ##########
echoVars () {
  echo "ROW_ID                    :$ROW_ID"
  echo "SELECTED_RDBMS            :$SELECTED_RDBMS"
  echo "SELECTED_STAFF            :$SELECTED_STAFF"
  echo "SELECTED_DBLOGIN          :$SELECTED_DBLOGIN"
  echo "SELECTED_STAFFID          :$SELECTED_STAFFID"
  echo "SELECTED_DBNAME           :$SELECTED_DBNAME"
  echo "SRC_LINE                  :$SRC_LINE"
  echo "SELECTED_APP              :$SELECTED_APP"
  echo "SELECTED_ACTION           :$SELECTED_ACTION"
  echo "SELECTED_AUTH             :$SELECTED_AUTH"
  echo "SELECTED_REASON           :$SELECTED_REASON"
}
########## Function extractVars: extract the data from the record structure ##########
extract_Vars () {
  ROW_ID=`echo ${SRC_LINE} | awk 'BEGIN {FS="="} {print $1} END {}'`
  case $ROW_ID in
    '$1') SELECTED_RDBMS=`echo ${SRC_LINE}   | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$2') SELECTED_STAFF=`echo ${SRC_LINE}   | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$3') SELECTED_DBLOGIN=`echo ${SRC_LINE} | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$4') SELECTED_STAFFID=`echo ${SRC_LINE} | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$5') SELECTED_DBNAME=`echo ${SRC_LINE}  | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print substr($1,1,4)"PROD"} END {}'`;;
    '$6') SELECTED_APP=`echo ${SRC_LINE}     | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$7') SELECTED_ACTION=`echo ${SRC_LINE}  | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$8') SELECTED_AUTH=`echo ${SRC_LINE}    | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
    '$9') SELECTED_REASON=`echo ${SRC_LINE}  | awk 'BEGIN {FS="<"} {print $2} END {}' | awk 'BEGIN {FS=">"} {print $1} END {}'`;;
       *) echo "Unexpected Record Type";;
  esac
}
########## Function process_AccessAble_file: process the record structure file for consolidation  ##########
process_AccessAble_file () {
# export CONFIGG="/mount/PRODDBA/oracle_scripts/recert/2_rows_oracle_Recertifications_Results_Issue_AccessAble_Extract.records"
  export CONFIGG="/mount/PRODDBA/oracle_scripts/recert/oracle_Recertifications_Results_Issue_AccessAble_Extract.records"
  echo "Original Config file: " ${CONFIGG}

  egrep -v '^Record|^----|=<>' $CONFIGG | while read SRC_LINE
    do
      extract_Vars
      if [ ${ROW_ID} = '$9' ]; then
        if [ ${SELECTED_RDBMS} != 'dbms' ]; then # not first row
#         echo $SELECTED_RDBMS $SELECTED_DBNAME $SELECTED_STAFF $SELECTED_DBLOGIN $SELECTED_ACTION $SELECTED_AUTH $SELECTED_REASON
          # if the Returns.sql file exists
          if [ -s /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_Returns.sql ]; then
            touch /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_Returns.sql
#           echo $SELECTED_RDBMS $SELECTED_DBNAME $SELECTED_STAFF $SELECTED_DBLOGIN $SELECTED_ACTION $SELECTED_AUTH $SELECTED_REASON
          else
            sed "s!SED_APP!$SELECTED_APP!g" /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_Returns_Header.sed > /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_Returns.sql
          fi
          echo "s/SED_LOGIN/$SELECTED_DBLOGIN/g" > /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed
          echo "s/SED_ACTION/$SELECTED_ACTION/g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed
          echo "s/SED_STAFF/$SELECTED_STAFF/g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed
          echo "s!SED_APP!$SELECTED_APP!g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed
          echo "s/SED_AUTH/$SELECTED_AUTH/g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed
          echo "s+SED_REASON+$SELECTED_REASON+g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed
          case $SELECTED_ACTION in
            "Remediated") echo "s/SED_SQL_COMMAND/DROP USER $SELECTED_DBLOGIN;/g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed;;
            "Lock"      ) echo "s/SED_SQL_COMMAND/ALTER USER $SELECTED_DBLOGIN ACCOUNT LOCK;/g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed;;
            "Unlock"    ) echo "s/SED_SQL_COMMAND/ALTER USER $SELECTED_DBLOGIN ACCOUNT UNLOCK;/g" >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed;;
            "*"         ) echo "Unexpected Action";;
          esac
        fi # not first row
        # if record.sed file exists - add it to the sql file
        if [ -s /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed ]; then
          sed -f /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_record.sed Process_AccessAble_Returns_Record.sed >> /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_${SELECTED_DBNAME}_Returns.sql
        fi
      fi # last record in block read
    done # egrep ... while loop
}
#========================================================================================

process_AccessAble_file

echo " Files created/used during the execution of this family of jobs"
ls -lrt /mount/PRODDBA/oracle_scripts/recert/Process_AccessAble_*Returns.sql
