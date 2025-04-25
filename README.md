connection to RAC
-  power cables (PDUS)& netwroking cables ( one is network core switch & out of band switch)
- network core switch - client connectivity (aPplication)
- out of band switch - OS access - anagement network will connect to out of band switch in data center

Bckup -
We use ZFS appliance and this will be connected to database server via infinii band switch

-- patching in exadata we will call as Bundle patch


single 42u rac

----
Terminology
----
Exadata database machine - complete exadata rack with database servers and storage servers and other hardware & software componenets ( 6ft machine where we habe all these components inifiniband, admin switch, db servers, cell serves, cabling)
exadata db server - one db server
exadata storage server - cell - storage cell - cell node - cell server

---
exadata benefits
---
exadata cell software offers smart scan, storage index, sfc, hcc, iorm, nwrm - exadata cell server software features
we have pipe bigger between db server and storage/cell
there is no i/o bottle neck, no loga btw db server and strorage server, data retrival is very fast
exadata uses inifiniband - 40gbps more than SCSI or FC 
exadata uses RoCE

-----
smart flash cache
-----
it is to improve the performance of redo log writes by directing writes to both flah & disk, but this only had a possitive effect on small percentage of redo writes

----
write through
----
the data first it will be written to disk then only it will written to smart flash cache

----
write back cache
---
the data will be written parralelly to smart flash cache & disk

---
ASR 
---
If all hardware components having any issues immediately one SR will raised and automatically logs wiill be uploaded.

---
[root@exa5dbadm01 ~]# dmidecode |grep -i "Product Name"
        Product Name: ORACLE SERVER X8-2
        Product Name: ASM,MB,X8-2
[root@exa5dbadm01 ~]#

---
oracle exadata extended xt storage servers are intended provide lower cost storage infrequently accesses older or regulatory data.
smart scan can be enable by licensing

---
what is oracle exadata?
oracle exadata is an engineered system designed by oracle corporation to deliver high performance database services, it combines oracle database software with software optimized for database workloads, providing features like smart scans, storage indexing and hybrid columnar compression which enhance preformance.

what are the key components ?
DB servers (compute nodes)
sorage server (cells) provi smart storage features like smart scan and storage indexing
inifiniband switch/RoCE switch - high speed network connectivity database servers and storage servers ensuring low latency and high throughput
oracle exadata storage software: manages data placemnet and storage operations, optimizing performance
access switch/management switch

what is smart scan ?
smart scan is a feature of oracle exadata that offloads data -intensive sql operations to the storage layer. it allows storage servers tp process and filter data, redusicng the amount of data transfered to db server, therefore query performance will improved.
(ex - if user is requested 10gb output the smart scan will read entire storage 10tb and it will only send the 10gb as output)

what is HCC (hybrid columnar compression)?
hcc is a compression technique used in oracle exadata that store data in a hybrid columnar format, significanly reducing storage requirements and improving query preformance by enabling better data compression and faster scans.
(whatever user is writing to db that will be stored in hcc format by compressing data)
compressed data takes less storage for ex 10g to 500m and data can be read faster 
hcc is not suitable for data where it constantly changing.

how does exadata handle backup and recovery ?
exadata supports rman for backup & reovery. it can leverage feauture like incremental backups, block change tracking, and optimized tape backups using oracle securebackup. additionally exadata integrates with oracle recovery appliances for continuous data protectiom and faster recovery.


storage index - DBA cannot control stoarge index
 
=========================================================================================
ARCHITECTURE
==============================================================
oracle linux and storage server will bepresent in LUN in the storage
lun will be formatted as cell disk
cell disk will be formatted as grid disk & flash dick that is visible in asm
DISK - LUN - CELL DISK - GRID DISK or flash cache - ASM

CELLCLI > create griddisk "path"

FLASH STOARGE ENTITIES & relations - 
--------
FLASHLUN - DELL DISK - FLASH CACHE OR FLASH CACHE & GRID DISK - VISIBLE IN ASM

CELLCLI > create flashcache 
CELLCLI > create griddisk .... flashcache ( both can be created )

With flashlun we can create flash cache
grid disk comes from physical lun

Disk group configuration
--------------
once we create cell disk by using cell disk we can create a cell group/ disk group
for ex- 3 cell disk will be created as one disk group  in cell 1 as data_1 and another 3 cell disk can be create as one diskgroup FRA_1 and in the same way in cell 2 we can create same disk groups as secondary copy if cell 1 fails we cell 2 (failover group)

classic db i/o & sql proccessing model
-----
If we ran any sql query  for example select cust_id from orders where ordere_amount>2000000; from instance it will go to infiniband switch and it will goto stoarge.
in stoarge we have this order table where order table is 10gb and with where condition it is like 2mb and this entire 10gb data will be sent to buffer cache and it will be filtered there for 2 mb which is time taking...

exadata samrt scan 
-----
If we ran the same query with here condition with the help of idb protocol and with stoarge server the filteration will hapeend in storage server and only 2mb will be sent to buffer cache.
No lag

exadata HCC
----
there is 10x and 5x compression will be provided exadata machine. data warehouse and archival compression
4 types of compression
query high, quer low, archive high, archive low

compression works under compression unit


Bundle patching
-------
bundle patching is free service for platinum support

platinum scheduling tool  - the unique url will be given each customer based on gateway server and this should be scheduled atleast 1 month before as slots will be not available and on sr will be create automatically with patch details.
exachk report need to be uploaded to that sr
patch template to be uploaded to sr
based on this patch assesment will be created ( curent home,current patch level, current grid home, target home, target patch level etc..)
patch plan will be done by vendor ( yum patching, cell patching, db paching, ib patching details and it will be using during the patching)
patch pre checks will be done by vendor they will login to our exadata and install the pacth versions and will do the prechecks if any issue this will be fixed cy customer
customer prechecks need to take backup of os /u01 & root
ecerything need to be noted before patching like cluster state listenere sate etc.
- main patching, engineer will do yum patching, db node patching (dbnodeupdate.sh) cell node patching (patch manger script), ibdwitch patch (patch manger script), db home patch will be done using opatch (oracle binary patching) and in db level they run cat bundle or data patch.
patch status will be updated in SR frequently
after patching exachk report need to uploaded to sr and engineer will compare with previous one.
within 48hrs if customer faces any issues oracle will fix it.
check all the db' listeners etc services status
doc id -888828.1 refer it for exadata versions & patch 

patching method -
rolling  - patch one node at a time ( ib switch by default is rolling)
non rolling - by shutting all db's cluster all nodes patch can be done at same time
hybrid - we can select rolling and nion rolling for db server & cell servers as per requirement


