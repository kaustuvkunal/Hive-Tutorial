## Hive command


Start hive :

`hive`
  

Start hive in debug mode : 

  `hive -hiveconf hive.root.logger=DEBUG,console`
  

## Hive Tables 

#### Managed table  

```
CREATE TABLE customer(ID int, Name STRING, Age int, Address STRING, Salary FLOAT)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY  '\t' 
LINES TERMINATED BY '\n' 
tblproperties("skip.header.line.count"="1"); 
load data into managed table -->
LOAD DATA LOCAL INPATH 'tables/customers.txt'  INTO table customer;
```

```
CREATE TABLE orders (OID int , dt STRING, Customer_ID int, Amount float)
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY  '\t' 
LINES TERMINATED BY '\n' 
tblproperties("skip.header.line.count"="1"); 
LOAD DATA LOCAL INPATH 'tables/ORDERS.txt'  INTO table orders;
```

#### External table 

```
CREATE EXTERNAL TABLE IF NOT EXISTS names_text(
  student_ID INT, FirstName STRING, LastName STRING,    
  year STRING, Major STRING)
  COMMENT 'Student Names'
  ROW FORMAT DELIMITED
  FIELDS TERMINATED BY ','
  STORED AS TEXTFILE
  LOCATION '/user/kaustuv/student';
  
```
 

#### Partitioning

```
CREATE TABLE employee (id INT, name STRING, dept STRING,year STRING ) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY  ',' 
LINES TERMINATED BY '\n' 
tblproperties("skip.header.line.count"="1"); 
```

```
LOAD DATA LOCAL INPATH 'tables/employee.txt'  INTO table employee;
create table employee_partioned (id INT, name STRING, dept STRING) PARTITIONED BY(year STRING);
```

```
set hive.exec.dynamic.partition.mode=nonstrict
```
Note : When mode is set to strict, then you need to do at least one static partition. In non-strict mode, all partitions are allowed to be dynamic


```
INSERT OVERWRITE TABLE employee_partioned PARTITION(year) SELECT id, name, dept,year from employee;
```
 


#### Bucketing  

 To enable buckets -> 
 ` set.hive.enforce.bucketing=true; `

```
CREATE TABLE bucketed_employee (id INT, name STRING, dept STRING,year STRING)
CLUSTERED BY (id) INTO 4 BUCKETS;
```


`INSERT OVERWRITE TABLE bucketed_employee SELECT * FROM employee;`


#### Views  


```
CREATE VIEW emp_hr AS
SELECT * FROM employee e
WHERE e.dept=' HR'
```

Space in present in original file thats why.
 

#### Drop Table

```
DROP TABLE IF EXISTS employee;;
```

For externat table only metadata will be deleted 

#### Truncate table 

Remove all table valid for managed table only removes all partition and can not be restored 

#### Alter  

Rename table:
```
ALTER TABLE customer  RENAME TO  customers;
```

#### MCSK Repair table 

 Users can run a metastore check command with the repair table option:
 
 `MSCK [REPAIR] TABLE table_name [ADD/DROP/SYNC PARTITIONS];`
 
ADD PARTITIONS with this option,it will add any partitions that exist on HDFS but not in metastore to the metastore. The DROP PARTITIONS option will remove the partition information from metastore, that is already removed from HDFS. The SYNC PARTITIONS option is equivalent to calling both ADD and DROP PARTITIONS
 
 
#### SHOW 
 
 ```
 show databases
 show tables  like 'c*'
 ```
 
 
 #### Describe 
 
 ```
  describe partition
  describe tablename 
  describe formatted tabllename
 ```
 

 #### Lock
 Manual locking of tables & partitions is possible in hive

`Lock table table_name SHARED` ( other can only read)

`Lock table table_name EXCLUSIVE` ( other can not even read )

 `Unlock table table_name`

 
## HIVE DML  
 
 
#### Loading files into tables 

```
LOAD DATA LOCAL INPATH 'tables/customers.txt'  INTO table customer;
```

 
LOAD DATA [LOCAL] INPATH 'filepath' [OVERWRITE] INTO TABLE tablename [PARTITION (partcol1=val1, partcol2=val2 ...)] [INPUTFORMAT 'inputformat' SERDE 'serde'] (3.0 or later)

 
#### Inserting data into Hive Tables from queries 
 
```
INSERT OVERWRITE TABLE employee_partioned PARTITION(year) SELECT id, name, dept,year from employee;
```

 

#### Writing data into the filesystem from queries 

```
INSERT OVERWRITE TABLE bucketed_employee SELECT * FROM employee;
```

 

#### Inserting values into tables from SQL 

``` 
 INSERT INTO TABLE tablename [PARTITION (partcol1[=val1], partcol2[=val2] ...)] VALUES values_row [, values_row ...]
``` 
 
``` 
INSERT INTO TABLE employee (id,name,dept,year) VALUES (9,' kunal',' TP',' 2020') ;
```

This write to a new file and need to merge those beacuse if you do select * from table_name it will not give updated result



`ALTER TABLE employee SET TBLPROPERTIES ("transactional"="true");`


Acid transaction property works with orc table only.


#### Creating Hive Acid tables

- Enable acid properties in hive-site 

```
    hive.support.concurrency – true
    hive.enforce.bucketing – true (Not required as of Hive 2.0)
    hive.exec.dynamic.partition.mode – nonstrict
    hive.txn.manager – org.apache.hadoop.hive.ql.lockmgr.DbTxnManager
    hive.compactor.initiator.on – true (for exactly one instance of the Thrift metastore service)
    hive.compactor.worker.threads – a positive number on at least one instance of the Thrift metastore service
```


- Create acid orc table 

 ```
CREATE TABLE employee_orc (id INT, name STRING, dept STRING,year STRING ) 
ROW FORMAT DELIMITED 
FIELDS TERMINATED BY  ',' 
LINES TERMINATED BY '\n' 
stored as orc
tblproperties("skip.header.line.count"="1","transactional"="true" ); 
```

```
INSERT INTO TABLE employee_orc SELECT * FROM employee;
```

#### Update 

- Updates can only be performed on tables that support ACID. 

```
UPDATE tablename SET column = value [, column = value ...] [WHERE expression]
```

```
UPDATE employee_orc SET dept = ' SC'  WHERE dept= 'SC' ;
```
 
 
#### Delete 

- Deletes can only be performed on tables that support ACID.
 
```
DELETE FROM tablename [WHERE expression]
 ```
```
DELETE FROM employee_orc WHERE  name='Prasanth'
```
 
 
#### Merge

- Merge can only be performed on tables that support ACID

 
- Use the MERGE statement to efficiently perform record-level INSERT, UPDATE, and DELETE operations within Hive tables.

 
 ```MERGE INTO <target table> AS T USING <source expression/table> AS S
ON <boolean expression1>
WHEN MATCHED [AND <boolean expression2>] THEN UPDATE SET <set clause list>
WHEN MATCHED [AND <boolean expression3>] THEN DELETE
WHEN NOT MATCHED [AND <boolean expression4>] THEN INSERT VALUES<value list>
```


 

 
### JOINS  

#### Inner join :

```
SELECT c.ID, c.NAME, c.AGE, o.AMOUNT
FROM CUSTOMERS c JOIN ORDERS o
ON (c.ID = o.CUSTOMER_ID);
```

#### Left Outer Join


```
SELECT c.ID, c.NAME,  o.AMOUNT, 0.dt
FROM CUSTOMERS c Left Outer Join ORDERS o
ON (c.ID = o.CUSTOMER_ID);
```


#### Right Outer Join

```
SELECT c.ID, c.NAME, o.AMOUNT, o.DT 
FROM CUSTOMERS c RIGHT OUTER JOIN 
ORDERS o ON (c.ID = o.CUSTOMER_ID);
```


#### Full Outer Join


```
SELECT c.ID, c.NAME, o.AMOUNT, o.DT
FROM CUSTOMERS c
FULL OUTER JOIN ORDERS o
ON (c.ID = o.CUSTOMER_ID);
```

#### MapSide join
```
SELECT /*MAPJOIN(c)*/ c.ID, c.NAME, c.AGE, o.AMOUNT
FROM CUSTOMERS c JOIN ORDERS o
ON (c.ID = o.CUSTOMER_ID)
```

#### Semi join
```
SELECT  c.ID, c.NAME, c.AGE, o.AMOUNT
FROM CUSTOMERS c  LEFT SEMI JOIN ORDERS o
ON (c.ID = o.CUSTOMER_ID)
```


### Hive Fuctions
- CONCAT
- CONCAT_WS
- TRIM
- LTRIM
- RTRIM
- REVERSE
- SPLIT
- SPACE

- EXPLODE : For lateral view  i.e mutilple value is segmement as one value at each row ( record)

```
select col1
from table1
LATERAL VIEW explode(col) explodeVal as col1
```




