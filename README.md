## SeaweedFS:

SeaweedFS is an open-source, highly scalable distributed file system designed to store and serve huge numbers of files efficiently, with particular strength in handling many small files and offering fast access.


### Core Architecture:

SeaweedFS uses a master–volume design rather than traditional chunk systems:

- Master Server:
    - Tracks volumes (large containers that hold many files).
    - Assigns file IDs and manages metadata at the volume level, not per file. 

- Volume Servers:
    - Store actual data (the file content).
    - Also hold metadata for the files they store, reducing load on the master. 

- Filer (optional):
    - Provides a directory structure and POSIX-like filesystem semantics.
    - Can store metadata in external databases such as MySQL, PostgreSQL, Redis, Cassandra, etc.



### Key Features:



### Prerequisites:


### Install Dependencies: 

```
apt install curl wget build-essential autoconf automake gdb git libffi-dev zlib1g-dev libssl-dev unzip golang -y
```



### Download SeaweedFS Binary:

SeaweedFS is distributed as a **single binary**.

```
cd /opt

wget https://github.com/seaweedfs/seaweedfs/releases/latest/download/linux_amd64.tar.gz

tar -xzf linux_amd64.tar.gz

chmod +x weed

cp weed /usr/local/bin/
```



```
weed version


version 30GB 4.03 bcce8d164c5387608207e4e935f24d6ca4a95f6a linux amd64
For enterprise users, please visit https://seaweedfs.com for SeaweedFS Enterprise Edition,
which has a self-healing storage format with better data protection.
```



### Create Directories for SeaweedFS Data:


_Create necessary directories for storage and configuration:_
```
sudo mkdir -p /opt/seaweedfs/etc /opt/seaweedfs/master /opt/seaweedfs/volume /opt/seaweedfs/filer
sudo chown -R $USER:$USER /opt/seaweedfs


//sudo mkdir -p /etc/seaweedfs /var/lib/seaweedfs/master /var/lib/seaweedfs/volume
//sudo chown -R $USER:$USER /etc/seaweedfs /var/lib/seaweedfs
```


### Configure SeaweedFS:

_Create a configuration file `/opt/seaweedfs/etc/seaweedfs.conf` for SeaweedFS:_

```conf
### SeaweedFS Configuration File:

master:
  #dir: /var/lib/seaweedfs/master
  dir: /opt/seaweedfs/master
  port: 9333
  #defaultReplication: 001

volume:
  #dir: /var/lib/seaweedfs/volume
  dir: /opt/seaweedfs/volume
  port: 8080
  max: 5

filer:
  port: 8888
  #dir: /var/lib/seaweedfs/filer
  dir: /opt/seaweedfs/filer
  collection: filer   # For embedded storage
```


### Run SeaweedFS Manually:

You can start the master, volume, and filer services manually to ensure everything works.
- The Master manages volumes and metadata (very lightweight).
- Volume servers store actual data.
- Filer provides directory structure & S3/FUSE support. Filer store metadata By default: `filerldb2`. 


#### 1. Start the master server:
```
weed master -mdir=/opt/seaweedfs/master -port=9333
```


#### 2. Start the volume server:
```
weed volume -dir=/opt/seaweedfs/volume -max=5 -mserver="localhost:9333" -port=8080
```


#### 3. Start the filer server (Optional but Recommended):
```
weed filer -master="localhost:9333" -port=8888

or,

weed filer -master="localhost:9333" -port=8888 -defaultStoreDir=/opt/seaweedfs/filer
```




### Access SeaweedFS:
```
Master UI: http://your-server-ip:9333

Volume UI: http://your-server-ip:8080

Filer UI: http://your-server-ip:8888
```




### Create a Systemd Service:

#### For Master Servers:

_Create a systemd service file `/etc/systemd/system/seaweed-master.service`:_

```bash
Description=SeaweedFS Master
After=network.target

[Service]
ExecStart=/usr/local/bin/weed master -ip=0.0.0.0 -port=9333 -mdir=/opt/seaweedfs/master
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```



#### For Volume Servers:


```
mkdir -p /opt/seaweedfs/volume1
mkdir -p /opt/seaweedfs/volume2
```


_Create a systemd service file `/etc/systemd/system/seaweed-volume1.service`:_

```bash
[Unit]
Description=SeaweedFS Volume
After=network.target seaweedfs-master.service

[Service]
ExecStart=/usr/local/bin/weed volume -mserver=localhost:9333 -port=8080 -max=10 -dir=/opt/seaweedfs/volume1
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```




_Create a systemd service file `/etc/systemd/system/seaweed-volume2.service`:_

```bash
[Unit]
Description=SeaweedFS Volume
After=network.target seaweedfs-master.service

[Service]
ExecStart=/usr/local/bin/weed volume -mserver=localhost:9333 -port=8081 -max=10 -dir=/opt/seaweedfs/volume2
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```






#### For Filer Servers:


_Create a systemd service file `/etc/systemd/system/seaweed-filer.service`:_

```bash
[Unit]
Description=SeaweedFS Filer
After=network.target seaweedfs-master.service

[Service]
#ExecStart=/usr/local/bin/weed filer -master=localhost:9333 -port=8888
ExecStart=/usr/local/bin/weed filer -master=localhost:9333 -port=8888 -defaultStoreDir=/opt/seaweedfs/filer
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```




#### Start and Verify: 

```
systemctl daemon-reload

systemctl start seaweed-master
systemctl start seaweed-volume1
systemctl start seaweed-volume2
systemctl start seaweed-filer
```


```
systemctl status seaweed-master
systemctl status seaweed-volume1
systemctl status seaweed-volume2
systemctl status seaweed-filer
```






### Use Cases:

SeaweedFS fits well in scenarios like:
- Object storage backends for web apps or microservices
- Media/media asset storage (thumbnails, videos)
- Data lakes for analytics
- Backup and archival systems
- S3-compatible storage layers for distributed systems





### Ref:
- [github.com/seaweedfs](https://github.com/seaweedfs/seaweedfs)





