[oracle@exa7dbadm02 leavers]$ ./testing.sh
The Oracle base has been set to /u01/app/oracle
Setting Oracle Instance Suffix for EB37GPRO
                           .... to EB37GPRO_1
EB37GPRO_1 /u01/app/oracle/product/12.2.0.1/dbhome_1
./testing.sh: line 7: /mount/PRODDBA/oracle_scripts/leaversd/b_credentials.txt: No such file or directory
PASSWORD=
Environment Variables:
ORACLE_SID=EB37GPRO_1
ORACLE_HOME=/u01/app/oracle/product/12.2.0.1/dbhome_1
PATH=/usr/local/bin:/usr/bin:/usr/local/sbin:/usr/sbin:/opt/puppetlabs/bin:/u01/app/oracle/product/12.2.0.1/dbhome_1/bin
SQL*Plus location: /u01/app/oracle/product/12.2.0.1/dbhome_1/bin/sqlplus
Testing DB Connection...

SP2-0306: Invalid option.
Usage: CONN[ECT] [{logon|/|proxy} [AS {SYSDBA|SYSOPER|SYSASM|SYSBACKUP|SYSDG|SYSKM|SYSRAC}] [edition=value]]
where <logon> ::= <username>[/<password>][@<connect_identifier>]
      <proxy> ::= <proxyuser>[<username>][/<password>][@<connect_identifier>]
[oracle@exa7dbadm02 leavers]$ cat testing.sh
export ORACLE_SID=EB37GPRO
export ORAENV_ASK=NO
. /usr/local/bin/oraenv

# === Read password from file ===
PWD_FILE=/mount/PRODDBA/oracle_scripts/leaversd/b_credentials.txt
PASSWORD=$(<"$PWD_FILE")
echo "PASSWORD= $PASSWORD"

echo "Environment Variables:"
echo "ORACLE_SID=$ORACLE_SID"
echo "ORACLE_HOME=$ORACLE_HOME"
echo "PATH=$PATH"
echo "SQL*Plus location: $(which sqlplus)"

# === DB Connection Test ===
echo "Testing DB Connection..."
echo "select name from v\$database;" | sqlplus -s SYSTEM/"${PASSWORD}"
if [ $? -ne 0 ]; then
    echo "ERROR: Oracle environment or DB connection failed."
    echo "Export FAILED: Cannot connect to DB $DB_NAME." | mail -s "Export FAILED - $DB_NAME" "$EMAIL_TO"
    exit 1
fi
why getti9ng error
