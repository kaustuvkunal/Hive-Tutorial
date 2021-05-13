###  Steps to Setup mysql as hive metastore

1.  Install mysql  and login to mysql terminal 
2.  Create metastore database <li>
             `CREATE DATABASE metastore;`
3.  Download   mysql-connector-java-($mysql-version)-jar  and copy into HIVE_HOME's lib directory . use [link](https://mvnrepository.com/artifact/mysql/mysql-connector-java) for download
     -  `cp mysql-connector-java-8.0.19.jar HIVE_HOME/lib/ `
4.  Use Metastore  `use metastore;`
5.  Create the Initial database schema using  hive-schema-3.1.0.mysql (here 3.1.0 is hadoop version select wrt to your hadoop version)
      
      `SOURCE $HIVE_HOME/scripts/metastore/upgrade/mysql/hive-schema-3.1.0.mysql.sql;`
      
6.   Create mysql user account for hive user  and grant access to metastore db
      `CREATE USER 'hiveuser'@'localhost' IDENTIFIED BY 'hivepassword';`
      
      `GRANT ALL  PRIVILEGES  on metastore.* to 'hiveuser'@'localhost' ;`
      
7. Configure hive-site.xml  inside $HIVE_HOME/conf/ and paste below properties

    ```
   <configuration> 
   <property> 
      <name>javax.jdo.option.ConnectionURL</name> 
      <value>jdbc:mysql://localhost/metastore?createDatabaseIfNotExist=true</value> 
      <description>metadata is stored in a MySQL server</description> 
   </property> 
   <property> 
      <name>javax.jdo.option.ConnectionDriverName</name> 
      <value>com.mysql.cj.jdbc.Driver</value> 
      <description>MySQL JDBC driver class</description> 
   </property> 

   <property> 
      <name>javax.jdo.option.ConnectionUserName</name> 
      <value>hiveuser</value> 
      <description>user name for connecting to mysql server</description> 
   </property> 
   <property> 
      <name>javax.jdo.option.ConnectionPassword</name> 
      <value>hivepassword</value> 
      <description>hivepassword for connecting to mysql server</description> 
   </property>
   <property> 
        <name>metastore.warehouse.dir</name> 
        <value>/user/hive/warehouse</value> 
        <description>location of default database for the warehouse</description> 
    </property> 
     
     <property> 
        <name>hive.metastore.uris</name> 
        <value>thrift://localhost:9083</value> 
        <description>Thrift URI for the remote metastore.</description> 
    </property> 
    <property> 
        <name>hive.server2.enable.doAs</name> 
        <value>false</value> 
    </property>
</configuration>
    
8. Refresh haddop and start hive 
      - `hive`
