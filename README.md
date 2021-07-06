# minio-

How to  install minio on secure  mapr cluster.

# Pre-req:-
Secure mapr data node or edge node.


1.  Install Posix client if not already installed on edge node or data node.

``` sudo yum install mapr-posix-client-basic

mkdir -m 777 /mapr
sudo systemctl start
mapr-posix-client-basic.service
```

2. Verify /mapr is mounted.


``` 
ls -la /mapr/
total 1
drwxr-xr-x 7 mapr mapr 6 Feb 16 18:06 example.mapr.cluster
```


# For Edge node.

1. Install  mapr-objectstore-client rpm 

``` 
sudo yum install mapr-objectstore-client -y
```
2.  Configure setup with mapr path

``` 
sudo /opt/mapr/objectstore-client/objectstore-client-*/bin/configure.sh -c -u mapr -g mapr --path /mapr/example.mapr.cluster/minio
```
3. Start object store services
Run the objectstore.sh script to start the Object Store:

``` 
sudo /opt/mapr/objectstore-client/objectstore-client-<version>/bin/objectstore.sh start
``` 
4. Verify that the Object Store process is running:
``` 
sudo /opt/mapr/objectstore-client/objectstore-client-<version>/bin/objectstore.sh status
When the process is running, the system prints the following message:
Minio is running
```

# For Data node


1. Install rpms and service is started successfully

```
yum install mapr-objectstore-client mapr-objectstore-gateway -y

sudo /opt/mapr/server/configure.sh -R

/opt/mapr/objectstore-client/objectstore-client-*/bin/objectstore.sh status

```

# Accesssing Minio Dashboad.

http://10.163.173.163:9000/minio/

Default username and password :- minioadmin/minioadmin


# Working with Minio Client

1 Configure ssl certificate for mc command

```
[root@vm1 certs]# pwd
/opt/mapr/objectstore-client/objectstore-client-2.0.0/conf/certs
[root@vm1 certs]#
[root@vm1 certs]# cp public.crt /root/.mc/certs/CAs
```

2. How to add host withi minio

``` 
[root@vm1 certs]# /opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc config host add minio https://vm1.mip.storage.hpecorp.net:9000 minioadmin minioadmin
Added `minio` successfully.
```
3. How to create bucket manually

``` 
/opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc mb minio/test1
```
4. How to list bucket contents 

``` 
/opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc ls minio/test1
```

# Hadoop Access via s3 api.

1. Add below line in core-site.xml according to your environment.

/opt/mapr/hadoop/hadoop-2.7.4/etc/hadoop/core-site.xml

```
<property>
        <name>fs.s3a.endpoint</name>
        <value>https://vm1.mip.storage.hpecorp.net:9000</value>
</property>
<property>
        <name>fs.s3a.access.key</name>
        <value>minioadmin</value>
</property>
<property>
        <name>fs.s3a.secret.key</name>
        <value>minioadmin</value>
</property>
<property>
        <name>fs.s3a.path.style.access</name>
        <value>true</value>
</property>
<property>
        <name>fs.s3a.impl</name>
        <value>org.apache.hadoop.fs.s3a.S3AFileSystem</value>
</property>
<property>
        <name>fs.s3a.connection.ssl.enabled</name>
        <value>false</value>
</property>
```

2. Setup hosts.

```
/opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc config host add maprfs https://vm1.mip.storage.hpecorp.net:9000 minioadmin minioadmin
```

3. Create bucket.

```
[root@vm1 ~]# /opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc mb maprfs/customer1
Bucket created successfully `maprfs/customer1`.
```

4. List buckets.

```
[root@vm1 ~]# /opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc ls maprfs
[2021-07-06 07:30:45 PDT]      0B customer1/
[2021-07-06 05:54:16 PDT]      0B sai1/
[2021-07-06 07:12:15 PDT]      0B test1/
[2021-07-06 07:28:04 PDT]      0B test2/

```

5. Copy contents.

```
[root@vm1 ~]# /opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc cp /etc/passwd maprfs/customer1
/etc/passwd:                    1.46 KiB / 1.46 KiB ┃▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓▓┃ 123.44 KiB/s 0s

```

6. List contents.

```
[root@vm1 ~]# /opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc ls maprfs/customer1
[2021-07-06 07:35:36 PDT]  1.5KiB passwd
```

7.IMP :- Remember to give complete path for bucket other wise you will get error bucket doesn't exists.
Example:-

```
[root@vm1 ~]# hadoop fs -ls s3a://customer1
ls: `s3a://customer1': No such file or directory
[root@vm1 ~]#
```

Correct way (Please note there is / after customer1 or bucket name)

```
[root@vm1 ~]# hadoop fs -ls s3a://customer1/
Found 1 items
-rw-rw-rw-   1       1494 2021-07-06 07:35 s3a://customer1/passwd
[root@vm1 ~]#
![image](https://user-images.githubusercontent.com/47996546/124620296-f746b900-de96-11eb-966e-a9a72106fd4b.png)

```

