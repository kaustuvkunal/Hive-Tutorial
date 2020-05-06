### Hive Installation Steps :

1. Download a stable release from [hive mirror](https://hive.apache.org/downloads.html) (the one with .bin.tar.gz)


2. Set Environment variable HIVE_HOME to point to the installation directory and add $HIVE_HOME/bin to  PATH:

    `export HIVE_HOME=/Users/kaustuv/apache-hive-2.3.4-bin`

    `export PATH=$HIVE_HOME/bin:$PATH`

    Use `hive --version`  to check hive version



3.  create /tmp and /user/hive/warehouse in HDFS and give write permission to group

`hadoop fs -chmod g+w   /tmp`

`hadoop fs -chmod g+w   /user/hive/warehouse`


4) Set hive-env.sh with hadoop path, hive-conf  and  heap size 

    `export HADOOP_HOME=/Users/hadoop/hadoop-2.6.0/`
    
    `export HIVE_CONF_DIR=/Users/hadoop/hive/conf`
    `export HADOOP_HEAPSIZE=1024`


4)Now set hive metastore  [refer]()
 
