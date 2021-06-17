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

2. Configure ssl certificate for mc command

```
[root@m2-maprts-vm163-173 certs]# pwd
/opt/mapr/objectstore-client/objectstore-client-2.0.0/conf/certs
[root@m2-maprts-vm163-173 certs]#
[root@m2-maprts-vm163-173 certs]# cp public.crt /root/.mc/certs/CAs
```

3. How to add host and  create bucket manually

``` 
[root@m2-maprts-vm163-173 certs]# /opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc config host add minio https://m2-maprts-vm163-173.mip.storage.hpecorp.net:9000 minioadmin minioadmin
Added `minio` successfully.
```
4. How to create bucket manually

``` 
/opt/mapr/objectstore-client/objectstore-client-2.0.0/util/mc mb minio/test1
```
