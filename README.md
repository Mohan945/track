#!/bin/ksh
# SCRIPT : prog_ora_TNS
# VERSION : 002
# Original AUTHOR : David Green
# DATE : April 2016
# PURPOSE :  This script "greps" the TNSNAMES.ORA file to produce a menu for the provided database prefix
#            which allow user helpdesk to change a users password on an Oracle database.
# FILES Referred: TNS_ADMIN="/home/helpdesk/TNSNAMES.ORA" | /home/sysadmin/all_inst_nonDataGuard_Y
# ALTERED: David Green 05th April 2016 and beyond
#          Major revamp of prog (original script) to eliminate menu and generate one instead for ANY Oracle Database in the TNSNAMES.ORA file
#          David Green 23rd Feb 2022
#          Introduction of new Oracle 19c Client for newer release databases and utilising corporate client tnsnames.ora /oem/oracle/admin/network/tnsnames.ora
#
###### Function: ContYN - continue after prompt ######
ContYN () {
  echo "Continue Y/N"
  read Var_CONTYN
  if [ -z ${Var_CONTYN} ] ; then
    Var_CONTYN="N"
  fi
  if [ ${Var_CONTYN} != "Y" -a ${Var_CONTYN} != "y" ] ; then
    echo "Stopping"
    exit
  fi
} # ContYN

###### Function: fn_extract_TNS_candidates ######
fn_extract_TNS_candidates () {
  Top_Banner=`egrep -n '#####################' ${TNS_File} | head -3 | tail -1 | awk 'BEGIN { FS=":" } {print $1} END {}'`
  TNS_Size=`wc -l ${TNS_File}|awk 'BEGIN { FS=" " } {print $1} END {}'`
  let Ignore_Head="$TNS_Size-$Top_Banner"

# echo "Ignore_Head" $Ignore_Head "Top_Banner" $Top_Banner "TNS_Size" $TNS_Size

  tail -$Ignore_Head ${TNS_File} | egrep ${DB_Prefix} | egrep -v '^#' > $DB_Selected_File

# cat $DB_Selected_File

  echo " "
  echo "Select TARGET database:"
  echo "-----------------------"
  awk 'BEGIN { FS = "="; seq=0 }
  {seq=seq+1}
  seq == 0 {print $1}
  seq != 0 {print seq ". " $1 }
  END {}' $DB_Selected_File
  echo ""
  TGT_NO=0
  export MAX_TGT_NO=`wc -l $DB_Selected_File|awk '{print $1}'`
  while [ ${TGT_NO} -le 0 -o ${TGT_NO} -gt ${MAX_TGT_NO} ]
    do
      echo "Input the Target number for selected Database 1 -" `expr ${MAX_TGT_NO}` " or q to quit"
      read TGT_NO
#  empty string input so try again
      if [ -z ${TGT_NO} ] ; then
        TGT_NO=0
      fi
# checking for q as input
      if [ "${TGT_NO}" = "q" ] ; then
        echo "Quitting , answer Y below"
        ContYN
        exit
      fi
# if it has other alphas then message numeric only set to 0 so we loop again.
      if [[ ! -z $(echo ${TGT_NO} | sed 's/[0-9]//g') ]] ; then
        echo "integer only"
        TGT_NO=0
      fi
    done
# fix the number as the config file has a header line.
  TGT_LINE=`head -${TGT_NO} $DB_Selected_File|tail -1`
  CONNECT_STRING=`echo ${TGT_LINE} | awk 'BEGIN { FS="=" } {print $1} END {}'`
  echo " "
  echo "Selected Database: " $CONNECT_STRING
  CONN_STR=`echo $CONNECT_STRING|cut -c 1-4`
  echo "Selected DB Group : $CONN_STR"
} # fn_extract_TNS_candidates

###### Function: fn_select_user ######
fn_select_user () {
  echo " "
  echo " Select TARGET User Account"
  echo " --------------------------"
  awk 'BEGIN { FS = "="; seq=-1 }
  {seq=seq+1}
  seq == 0 {print $1}
  seq != 0 {print seq ". " $1 " " $2 " " $3 " " $4}
  END {}' $User_Selected_File
  echo ""
  TGT_NO=0
  export MAX_TGT_NO=`wc -l $User_Selected_File|awk '{print $1}'`
  while [ ${TGT_NO} -le 0 -o ${TGT_NO} -gt ${MAX_TGT_NO} ]
    do
      echo "Input the Target number for selected User Account 1 -" `expr ${MAX_TGT_NO} - 1`  " or q to quit"
      read TGT_NO
#  empty string input so try again
      if [ -z ${TGT_NO} ] ; then
        TGT_NO=0
      fi
# checking for q as input
      if [ "${TGT_NO}" = "q" ] ; then
        echo "Quitting , answer Y below"
        ContYN
        exit
      fi
# if it has alphas then message numeric only set to 0 so we loop again.
      if [[ ! -z $(echo ${TGT_NO} | sed 's/[0-9]//g') ]] ; then
        echo "integer only"
        TGT_NO=0
      fi
    done
# fix the number as the config file has a header line.
  TGT_NO=`expr ${TGT_NO} + 1`

  TGT_LINE=`head -${TGT_NO} $User_Selected_File|tail -1`
  USER_STRING=`echo ${TGT_LINE} | awk 'BEGIN { FS=" " } {print $1} END {}'`
} # fn_select_user

###### Function: fn_assign_client ######
fn_assign_client () {
  export PATH=${PATH}:/usr/local/bin
  export ORAENV_ASK=NO
  Current_Client=`egrep $CONN_STR /home/sysadmin/all_inst_nonDataGuard_Y | awk 'BEGIN { FS=":" } {print $2} END {}'`
# echo "Current Client" $Current_Client
  case $Current_Client in
   "v12.2.0.1") export ORACLE_SID=CLIENT_19C;;
   "v19.0.0.0") export ORACLE_SID=CLIENT_19C;;
           "*") export ORACLE_SID=CLIENT_10G;;
  esac
 export Current_Client=$ORACLE_SID
#if [ "$[Current_Client]" = "v12.2.0.1" -o "$[Current_Client]" = "v19.0.0.0" ] ; then
#    export ORACLE_SID=CLIENT_19C
#  else
#    export ORACLE_SID=CLIENT_10G
#  fi
  . oraenv
  echo "Current Client : " $Current_Client
  echo $ORACLE_HOME $ORACLE_SID
} # fn_assign_client

#############################################################################
# Setup the Oracle and user bits
#
CURRENT_USER=`who am i | awk 'BEGIN { FS=" " } {print $1} END {}'`
TNS_File="/oem/oracle/admin/network/tnsnames.ora"
TNS_ADMIN="/oem/oracle/admin/network/tnsnames.ora"
Audit_File="/home/helpdesk/Oracle_Password_Audit.log"
DB_Selected_File="/home/helpdesk/${CURRENT_USER}_DB_selected.txt"
Log_F="/home/helpdesk/${CURRENT_USER}_password_ora_TNS_reset_unlock_process.log"
#
DBLoop="continue"
while [ "$DBLoop" = "continue" ]
  do
    tput clear
    echo ""
    echo ""
    echo "     The following menu can be used by IS Service Desk staff."
    echo "     From here, the password for an Oracle database user can be changed."
    echo ""
    echo "     Please read the screen and respond to the prompts."
    echo "     This script has been changed quite considerably, with database"
    echo "     connections selected from a corporate TNSNAMES.ORA, then users"
    echo "     selected from the chosen database."
    echo ""
    /home/helpdesk/Oracle_DB_List
    echo ""
    echo "     Select the number which corresponds to the database or user required."
    echo "     To quit this menu type 'q'"
    echo ""
    echo "     Please enter first four letters of the database in which the Oracle account resides"
    echo ""
    read DB_Prefix
    if [ "${DB_Prefix}" = "q" ]; then
      echo "Quitting , answer Y below"
      ContYN
      exit
    fi

    DB_Prefix=`echo $DB_Prefix | tr '[a-z]' '[A-Z]'`
    echo ""

    fn_extract_TNS_candidates

    fn_assign_client
#
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Login Name processing
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    echo " "
    echo "Please type the user name..."
    read User1
    User1=`echo $User1 | tr '[a-z]' '[A-Z]'`
    echo ""
  export ORAENV_ASK=NO
  . oraenv
 echo "Current Client, Oracle home : $Current_Client, $ORACLE_HOME "

    sed s/ACCT/$User1/g <query_expiry.sql_sedsource > ${CURRENT_USER}_query_${User1}_expiry.sql
    User_Exists_File="/home/helpdesk/${CURRENT_USER}_query_${User1}_exists.log"
    User_Selected_File="/home/helpdesk/${CURRENT_USER}_query_${User1}_expiry.log"
    sqlplus service_admin/`cat /oem/oracle/admin/passwords/service_admin.pwd`@${CONNECT_STRING} <${CURRENT_USER}_query_${User1}_expiry.sql > ${User_Exists_File}
#   Either one of two cases is now relevant - no rows selected / at least TWO "hits" with this egrep 'STATUS|OPEN|LOCK|EXPIRE' - but the no rows selected is a good enough test
    norowcount=`egrep -c 'no rows selected' ${User_Exists_File}`
    if [ "$norowcount" = "1" ]; then
      echo " User: ${User1} not found in database ${CONNECT_STRING} - please confirm with the Customer the correct Database and User"
      ContYN
      exit
    else
      badfailure=`egrep -c 'STATUS|OPEN|LOCK|EXPIRE' ${User_Exists_File}`
      if [ ${badfailure} = "0" ]; then
        echo " An error has occurred. Please pass this information to Prod (GOUK Oracle) or Devt (DB_DESIGN) :  Database $CONNECT_STRING User $User1 Program $0"
        echo ""
        cat ${User_Exists_File}
        echo ""
        echo " An error has occurred. Please pass this information to Prod (GOUK Oracle) or Devt (DB_DESIGN) : Database $CONNECT_STRING User $User1 Program $0"
        ContYN
        exit
      else
        egrep 'STATUS|OPEN|LOCK|EXPIRE' ${User_Exists_File} > ${User_Selected_File}
        fn_select_user
      fi
    fi
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Display menu options to 1) reset password and unlock account or 2) unlock account
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    echo " "
    echo "For user " $USER_STRING " do you want to:"
    echo "1) Reset the Account password and Unlock the Oracle account"
    echo "2) Simply Unlock the Oracle account"
    read ResetUnlock
    if [ "$ResetUnlock" = "2" ] ; then
      cp /dev/null ${CURRENT_USER}_prog_ora_TNS_unlock_sed
      echo "s/RESET_ACCT/$USER_STRING/g" >> ${CURRENT_USER}_prog_ora_TNS_unlock_sed
      sed -f ${CURRENT_USER}_prog_ora_TNS_unlock_sed prog_ora_TNS_unlock_sed.sql > ${CURRENT_USER}_prog_ora_TNS_passwd_reset_unlock.sql
    else
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Password processing
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
      echo "Please type the user password..."
      read Pass1
      echo ""
      cp /dev/null ${CURRENT_USER}_prog_ora_TNS_pass_sed
      echo "s/RESET_ACCT/$USER_STRING/g" >> ${CURRENT_USER}_prog_ora_TNS_pass_sed
      echo "s/RESET_PASS/$Pass1/g" >> ${CURRENT_USER}_prog_ora_TNS_pass_sed
      sed -f ${CURRENT_USER}_prog_ora_TNS_pass_sed prog_ora_TNS_passwd_reset_sed.sql > ${CURRENT_USER}_prog_ora_TNS_passwd_reset_unlock.sql
    fi
##########################################################################################################
# Check the user is not a schema user  Added after CRTS password change incident INC0946018. 02/09/2014 JC
##########################################################################################################
    i_ctr=0
    i_ctr=`grep -i $USER_STRING /home/helpdesk/USER_LIST/*USR_LST|wc -l`

    if [ $i_ctr -ne 0 ] ; then
      Now_Date=`date`
      echo " ==============================================================================\n"
      echo " Password for this user : $USER_STRING can't change!!!!!! contact GOUK Oracle.........\n"
      echo " ==============================================================================\n"
      ContYN
      exit;
    fi
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Execution Phase
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Now call sqlplus
    sqlplus service_admin/`cat /oem/oracle/admin/passwords/service_admin.pwd`@$CONNECT_STRING <${CURRENT_USER}_prog_ora_TNS_passwd_reset_unlock.sql > $Log_F
#                     Oracle Ok/failure
    echo " "
    n=`egrep 'altered' $Log_F|wc -l`
    if [ $n -ne 0 ] ; then
      Now_Date=`date`
      if [ "$ResetUnlock" = "1" ] ; then
        Account_Process=" password changed"
      else
        Account_Process=" account unlocked"
      fi
# Thu 1 Dec 12:23:50 2005 sl093r :  sl093r  on  IN08PROD  password changed / account unlocked
      echo $Now_Date ": " $USER_STRING " on " $CONNECT_STRING $Account_Process
      echo $Now_Date ": " $USER_STRING " on " $CONNECT_STRING $Account_Process >> $Audit_File
      ContYN
    else
      egrep 'ORA-|punctuation' $Log_F
      ContYN
    fi
# remove temporary files as no longer required
    #rm $Log_F ${User_Exists_File} ${User_Selected_File}
  done
# End of Loop - User Menu Response not "q"

oracle@p5-f3mp15> /home/helpdesk/prog_ora_TNS


     The following menu can be used by IS Service Desk staff.
     From here, the password for an Oracle database user can be changed.

     Please read the screen and respond to the prompts.
     This script has been changed quite considerably, with database
     connections selected from a corporate TNSNAMES.ORA, then users
     selected from the chosen database.

 - - - - - - - - - - - - - - - - - - Oracle Database Name and Content - - - - - - -  - - - - - - - - - - -
 AU41PROD  Accurate           AU42PROD  Accurate MI         AV01PROD  Audit Vault
 BM21PROD  BAW Stream         BM22PROD  BPM 22              BM23PROD  BPM 23              BM24PROD  BPM 24
 CD21PROD  DB Synch           CI01PROD  Customer Interact
 CN03PROD  Cognos Exadata     CX21PROD  Corp Conformd Dims  DL21PROD  Data Leakage

 EB22PROD  OBIEE Finance MD   EB25PROD  OBIEE Finance WH
 EB21PROD  EBS FaST           EB23PROD  SSO                 EB37PROD  DRM
 EM13PROD  OEM 13c2           EM35PROD  OEM 13c5

 FF01PROD  Fortify            GP07PROD  MRM/Bingo           GP10PROD  Cognos/General      GP22PROD  General
 GX21PROD  General Apex
 IN04PROD  SalesForce/OTC Derivatives
 IN05PROD  FOS                IN07PROD  SDS/EUC/ADOS        IN09PROD  Inv Integ Layer     IN21PROD  Cognos Planning/Safari

 KW21PROD  Cust Platform      MD11PROD  Market Director     MI21PROD  MI21                MI22PROD  MI22
 N601PROD  Sales MI           OA21PROD  Open AM             PR05PROD  Property Offers

 QP04PROD  QPASA Dev          QP05PROD  QPASA Prod          RC11PROD  RecoCat 11          RC12PROD  RecoCat 12
 RD02PROD  Retire/Del         RD23PROD  ORAC                RI22PROD  RoI PSols/PRD1      SQ01PROD  SonarQube
 TZ02PROD  FaST PDS/FBI       TZ21PROD  Solvency 2          TZ23PROD  Finance Planning
 UR02PROD  MutualFunds        WT21PROD  WatchList 21        WT22PROD  WatchList 22        WT23PROD  WatchList 23

     Select the number which corresponds to the database or user required.
     To quit this menu type 'q'

     Please enter first four letters of the database in which the Oracle account resides


racle@p5-f3mp15> cat /home/helpdesk/prog_ora_TNS
#!/bin/ksh
# SCRIPT : prog_ora_TNS
# VERSION : 002
# Original AUTHOR : David Green
# DATE : April 2016
# PURPOSE :  This script "greps" the TNSNAMES.ORA file to produce a menu for the provided database prefix
#            which allow user helpdesk to change a users password on an Oracle database.
# FILES Referred: TNS_ADMIN="/home/helpdesk/TNSNAMES.ORA" | /home/sysadmin/all_inst_nonDataGuard_Y
# ALTERED: David Green 05th April 2016 and beyond
#          Major revamp of prog (original script) to eliminate menu and generate one instead for ANY Oracle Database in the TNSNAMES.ORA file
#          David Green 23rd Feb 2022
#          Introduction of new Oracle 19c Client for newer release databases and utilising corporate client tnsnames.ora /oem/oracle/admin/network/tnsnames.ora
#
###### Function: ContYN - continue after prompt ######
ContYN () {
  echo "Continue Y/N"
  read Var_CONTYN
  if [ -z ${Var_CONTYN} ] ; then
    Var_CONTYN="N"
  fi
  if [ ${Var_CONTYN} != "Y" -a ${Var_CONTYN} != "y" ] ; then
    echo "Stopping"
    exit
  fi
} # ContYN

###### Function: fn_extract_TNS_candidates ######
fn_extract_TNS_candidates () {
  Top_Banner=`egrep -n '#####################' ${TNS_File} | head -3 | tail -1 | awk 'BEGIN { FS=":" } {print $1} END {}'`
  TNS_Size=`wc -l ${TNS_File}|awk 'BEGIN { FS=" " } {print $1} END {}'`
  let Ignore_Head="$TNS_Size-$Top_Banner"

# echo "Ignore_Head" $Ignore_Head "Top_Banner" $Top_Banner "TNS_Size" $TNS_Size

  tail -$Ignore_Head ${TNS_File} | egrep ${DB_Prefix} | egrep -v '^#' > $DB_Selected_File

# cat $DB_Selected_File

  echo " "
  echo "Select TARGET database:"
  echo "-----------------------"
  awk 'BEGIN { FS = "="; seq=0 }
  {seq=seq+1}
  seq == 0 {print $1}
  seq != 0 {print seq ". " $1 }
  END {}' $DB_Selected_File
  echo ""
  TGT_NO=0
  export MAX_TGT_NO=`wc -l $DB_Selected_File|awk '{print $1}'`
  while [ ${TGT_NO} -le 0 -o ${TGT_NO} -gt ${MAX_TGT_NO} ]
    do
      echo "Input the Target number for selected Database 1 -" `expr ${MAX_TGT_NO}` " or q to quit"
      read TGT_NO
#  empty string input so try again
      if [ -z ${TGT_NO} ] ; then
        TGT_NO=0
      fi
# checking for q as input
      if [ "${TGT_NO}" = "q" ] ; then
        echo "Quitting , answer Y below"
        ContYN
        exit
      fi
# if it has other alphas then message numeric only set to 0 so we loop again.
      if [[ ! -z $(echo ${TGT_NO} | sed 's/[0-9]//g') ]] ; then
        echo "integer only"
        TGT_NO=0
      fi
    done
# fix the number as the config file has a header line.
  TGT_LINE=`head -${TGT_NO} $DB_Selected_File|tail -1`
  CONNECT_STRING=`echo ${TGT_LINE} | awk 'BEGIN { FS="=" } {print $1} END {}'`
  echo " "
  echo "Selected Database: " $CONNECT_STRING
  CONN_STR=`echo $CONNECT_STRING|cut -c 1-4`
  echo "Selected DB Group : $CONN_STR"
} # fn_extract_TNS_candidates

###### Function: fn_select_user ######
fn_select_user () {
  echo " "
  echo " Select TARGET User Account"
  echo " --------------------------"
  awk 'BEGIN { FS = "="; seq=-1 }
  {seq=seq+1}
  seq == 0 {print $1}
  seq != 0 {print seq ". " $1 " " $2 " " $3 " " $4}
  END {}' $User_Selected_File
  echo ""
  TGT_NO=0
  export MAX_TGT_NO=`wc -l $User_Selected_File|awk '{print $1}'`
  while [ ${TGT_NO} -le 0 -o ${TGT_NO} -gt ${MAX_TGT_NO} ]
    do
      echo "Input the Target number for selected User Account 1 -" `expr ${MAX_TGT_NO} - 1`  " or q to quit"
      read TGT_NO
#  empty string input so try again
      if [ -z ${TGT_NO} ] ; then
        TGT_NO=0
      fi
# checking for q as input
      if [ "${TGT_NO}" = "q" ] ; then
        echo "Quitting , answer Y below"
        ContYN
        exit
      fi
# if it has alphas then message numeric only set to 0 so we loop again.
      if [[ ! -z $(echo ${TGT_NO} | sed 's/[0-9]//g') ]] ; then
        echo "integer only"
        TGT_NO=0
      fi
    done
# fix the number as the config file has a header line.
  TGT_NO=`expr ${TGT_NO} + 1`

  TGT_LINE=`head -${TGT_NO} $User_Selected_File|tail -1`
  USER_STRING=`echo ${TGT_LINE} | awk 'BEGIN { FS=" " } {print $1} END {}'`
} # fn_select_user

###### Function: fn_assign_client ######
fn_assign_client () {
  export PATH=${PATH}:/usr/local/bin
  export ORAENV_ASK=NO
  Current_Client=`egrep $CONN_STR /home/sysadmin/all_inst_nonDataGuard_Y | awk 'BEGIN { FS=":" } {print $2} END {}'`
# echo "Current Client" $Current_Client
  case $Current_Client in
   "v12.2.0.1") export ORACLE_SID=CLIENT_19C;;
   "v19.0.0.0") export ORACLE_SID=CLIENT_19C;;
           "*") export ORACLE_SID=CLIENT_10G;;
  esac
 export Current_Client=$ORACLE_SID
#if [ "$[Current_Client]" = "v12.2.0.1" -o "$[Current_Client]" = "v19.0.0.0" ] ; then
#    export ORACLE_SID=CLIENT_19C
#  else
#    export ORACLE_SID=CLIENT_10G
#  fi
  . oraenv
  echo "Current Client : " $Current_Client
  echo $ORACLE_HOME $ORACLE_SID
} # fn_assign_client

#############################################################################
# Setup the Oracle and user bits
#
CURRENT_USER=`who am i | awk 'BEGIN { FS=" " } {print $1} END {}'`
TNS_File="/oem/oracle/admin/network/tnsnames.ora"
TNS_ADMIN="/oem/oracle/admin/network/tnsnames.ora"
Audit_File="/home/helpdesk/Oracle_Password_Audit.log"
DB_Selected_File="/home/helpdesk/${CURRENT_USER}_DB_selected.txt"
Log_F="/home/helpdesk/${CURRENT_USER}_password_ora_TNS_reset_unlock_process.log"
#
DBLoop="continue"
while [ "$DBLoop" = "continue" ]
  do
    tput clear
    echo ""
    echo ""
    echo "     The following menu can be used by IS Service Desk staff."
    echo "     From here, the password for an Oracle database user can be changed."
    echo ""
    echo "     Please read the screen and respond to the prompts."
    echo "     This script has been changed quite considerably, with database"
    echo "     connections selected from a corporate TNSNAMES.ORA, then users"
    echo "     selected from the chosen database."
    echo ""
    /home/helpdesk/Oracle_DB_List
    echo ""
    echo "     Select the number which corresponds to the database or user required."
    echo "     To quit this menu type 'q'"
    echo ""
    echo "     Please enter first four letters of the database in which the Oracle account resides"
    echo ""
    read DB_Prefix
    if [ "${DB_Prefix}" = "q" ]; then
      echo "Quitting , answer Y below"
      ContYN
      exit
    fi

    DB_Prefix=`echo $DB_Prefix | tr '[a-z]' '[A-Z]'`
    echo ""

    fn_extract_TNS_candidates

    fn_assign_client
#
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Login Name processing
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    echo " "
    echo "Please type the user name..."
    read User1
    User1=`echo $User1 | tr '[a-z]' '[A-Z]'`
    echo ""
  export ORAENV_ASK=NO
  . oraenv
 echo "Current Client, Oracle home : $Current_Client, $ORACLE_HOME "

    sed s/ACCT/$User1/g <query_expiry.sql_sedsource > ${CURRENT_USER}_query_${User1}_expiry.sql
    User_Exists_File="/home/helpdesk/${CURRENT_USER}_query_${User1}_exists.log"
    User_Selected_File="/home/helpdesk/${CURRENT_USER}_query_${User1}_expiry.log"
    sqlplus service_admin/`cat /oem/oracle/admin/passwords/service_admin.pwd`@${CONNECT_STRING} <${CURRENT_USER}_query_${User1}_expiry.sql > ${User_Exists_File}
#   Either one of two cases is now relevant - no rows selected / at least TWO "hits" with this egrep 'STATUS|OPEN|LOCK|EXPIRE' - but the no rows selected is a good enough test
    norowcount=`egrep -c 'no rows selected' ${User_Exists_File}`
    if [ "$norowcount" = "1" ]; then
      echo " User: ${User1} not found in database ${CONNECT_STRING} - please confirm with the Customer the correct Database and User"
      ContYN
      exit
    else
      badfailure=`egrep -c 'STATUS|OPEN|LOCK|EXPIRE' ${User_Exists_File}`
      if [ ${badfailure} = "0" ]; then
        echo " An error has occurred. Please pass this information to Prod (GOUK Oracle) or Devt (DB_DESIGN) :  Database $CONNECT_STRING User $User1 Program $0"
        echo ""
        cat ${User_Exists_File}
        echo ""
        echo " An error has occurred. Please pass this information to Prod (GOUK Oracle) or Devt (DB_DESIGN) : Database $CONNECT_STRING User $User1 Program $0"
        ContYN
        exit
      else
        egrep 'STATUS|OPEN|LOCK|EXPIRE' ${User_Exists_File} > ${User_Selected_File}
        fn_select_user
      fi
    fi
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Display menu options to 1) reset password and unlock account or 2) unlock account
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
    echo " "
    echo "For user " $USER_STRING " do you want to:"
    echo "1) Reset the Account password and Unlock the Oracle account"
    echo "2) Simply Unlock the Oracle account"
    read ResetUnlock
    if [ "$ResetUnlock" = "2" ] ; then
      cp /dev/null ${CURRENT_USER}_prog_ora_TNS_unlock_sed
      echo "s/RESET_ACCT/$USER_STRING/g" >> ${CURRENT_USER}_prog_ora_TNS_unlock_sed
      sed -f ${CURRENT_USER}_prog_ora_TNS_unlock_sed prog_ora_TNS_unlock_sed.sql > ${CURRENT_USER}_prog_ora_TNS_passwd_reset_unlock.sql
    else
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Password processing
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
      echo "Please type the user password..."
      read Pass1
      echo ""
      cp /dev/null ${CURRENT_USER}_prog_ora_TNS_pass_sed
      echo "s/RESET_ACCT/$USER_STRING/g" >> ${CURRENT_USER}_prog_ora_TNS_pass_sed
      echo "s/RESET_PASS/$Pass1/g" >> ${CURRENT_USER}_prog_ora_TNS_pass_sed
      sed -f ${CURRENT_USER}_prog_ora_TNS_pass_sed prog_ora_TNS_passwd_reset_sed.sql > ${CURRENT_USER}_prog_ora_TNS_passwd_reset_unlock.sql
    fi
##########################################################################################################
# Check the user is not a schema user  Added after CRTS password change incident INC0946018. 02/09/2014 JC
##########################################################################################################
    i_ctr=0
    i_ctr=`grep -i $USER_STRING /home/helpdesk/USER_LIST/*USR_LST|wc -l`

    if [ $i_ctr -ne 0 ] ; then
      Now_Date=`date`
      echo " ==============================================================================\n"
      echo " Password for this user : $USER_STRING can't change!!!!!! contact GOUK Oracle.........\n"
      echo " ==============================================================================\n"
      ContYN
      exit;
    fi
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Execution Phase
# - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
# Now call sqlplus
    sqlplus service_admin/`cat /oem/oracle/admin/passwords/service_admin.pwd`@$CONNECT_STRING <${CURRENT_USER}_prog_ora_TNS_passwd_reset_unlock.sql > $Log_F
#                     Oracle Ok/failure
    echo " "
    n=`egrep 'altered' $Log_F|wc -l`
    if [ $n -ne 0 ] ; then
      Now_Date=`date`
      if [ "$ResetUnlock" = "1" ] ; then
        Account_Process=" password changed"
      else
        Account_Process=" account unlocked"
      fi
# Thu 1 Dec 12:23:50 2005 sl093r :  sl093r  on  IN08PROD  password changed / account unlocked
      echo $Now_Date ": " $USER_STRING " on " $CONNECT_STRING $Account_Process
      echo $Now_Date ": " $USER_STRING " on " $CONNECT_STRING $Account_Process >> $Audit_File
      ContYN
    else
      egrep 'ORA-|punctuation' $Log_F
      ContYN
    fi
# remove temporary files as no longer required
    #rm $Log_F ${User_Exists_File} ${User_Selected_File}
  done
# End of Loop - User Menu Response not "q"

I am getting this below error why


Current Client, Oracle home : CLIENT_19C, /oem/oracle/product/19.3.0.0_Client
A file or directory in the path name does not exist.
/home/helpdesk/prog_ora_TNS[198]: query_expiry.sql_sedsource: 0403-016 Cannot find or open the file.
A file or directory in the path name does not exist.
/home/helpdesk/prog_ora_TNS[201]: co70989_query_CO65131_expiry.sql: 0403-016 Cannot find or open the file.
egrep: 0652-033 Cannot open /home/helpdesk/co70989_query_CO65131_exists.log.
egrep: 0652-033 Cannot open /home/helpdesk/co70989_query_CO65131_exists.log.
/home/helpdesk/prog_ora_TNS[210]: test: 0403-004 Specify a parameter with this command.
egrep: 0652-033 Cannot open /home/helpdesk/co70989_query_CO65131_exists.log.
