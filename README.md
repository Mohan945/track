[oracle@exa5dbadm01 test]$ cat "/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"
SLAESL  CO70989 2027897 Charley Laidlaw 111403 Conformance and Risk     Termination
SLAESL  CO70000 2020897 Charley Loidlaw 111003 Conformance and Risk     Termination
[oracle@exa5dbadm01 test]$  grep -E "Redundancy|Resignation|Termination|Retirement|EMPLOYEE HAS LEFT" "/mount/PRODDBA/oracle_scripts/recert/leavers/test/employee_ids.txt"| awk -F'\t' '{print $2}'


[oracle@exa5dbadm01 test]$
