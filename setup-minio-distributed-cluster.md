![image](https://github.com/dagonbo/minio-node-distributed-cluster/assets/51602389/6f0908b0-29cd-43cc-b2a6-f7e6d736d073)# minio-node-distributed-cluster-centos7
In this guide will install 3 node cluster

## Pre-Require
Open firwall port 9000 and 9001(or specific port)
`
firewall-cmd --permanent --zone=public --add-port=9000/tcp
firewall-cmd --permanent --zone=public --add-port=9001/tcp
firewall-cmd --reload
`
We need partition for mount minio volume
`
sudo lsblk
`
`
NAME                      MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
fd0                         2:0    1    4K  0 disk
sda                         8:0    0  127G  0 disk
├─sda1                      8:1    0    1G  0 part /boot
├─sda2                      8:2    0   80G  0 part
│ ├─centos-root           253:0    0   30G  0 lvm  /
│ ├─centos-swap           253:1    0  3.9G  0 lvm  [SWAP]
│ ├─centos-pool00_tmeta   253:3    0   28M  0 lvm
│ │ └─centos-pool00-tpool 253:5    0   30G  0 lvm
│ │   ├─centos-pool00     253:6    0   30G  1 lvm
│ │   └─centos-home       253:7    0   30G  0 lvm  /home
│ └─centos-pool00_tdata   253:4    0   30G  0 lvm
│   └─centos-pool00-tpool 253:5    0   30G  0 lvm
│     ├─centos-pool00     253:6    0   30G  1 lvm
│     └─centos-home       253:7    0   30G  0 lvm  /home
└─sda3                      8:3    0   40G  0 part
  └─minio-data            253:2    0   40G  0 lvm  <--- partition empty mount that we need
sr0                        11:0    1 1024M  0 rom
`
### Let Start mount session
We want to mount partition(minio-data) with /data
`
sudo su 
`
`
mkdir /data
`
`
echo "/dev/mapper/minio-data /data" >> /etc/fstab
`
`
mount -a
`
So, output should be.
`
df -h
`
`
Filesystem               Size  Used Avail Use% Mounted on
devtmpfs                 1.9G     0  1.9G   0% /dev
tmpfs                    1.9G     0  1.9G   0% /dev/shm
tmpfs                    1.9G   57M  1.8G   3% /run
tmpfs                    1.9G     0  1.9G   0% /sys/fs/cgroup
/dev/mapper/centos-root   30G  1.6G   29G   6% /
/dev/sda1               1014M  151M  864M  15% /boot
/dev/mapper/minio-data    40G   33M   40G   1% /data <-- Here!
/dev/mapper/centos-home   30G   33M   30G   1% /home
tmpfs                    379M     0  379M   0% /run/user/0
`
## Start Setup Guide
### Set hosts file
We finished mount partition, So we can start setup by set ipv4 on hosts file every instance(3 instance).
`
cat > /etc/hosts << EOF
<PrivateIpv4-VM1>    minio1
<PrivateIpv4-VM2>    minio2
<PrivateIpv4-VM3>    minio3
127.0.0.1     localhost
EOF
`
### Install Minio
In case didn't installed wget
`
yum install wget
`
`
wget -O /usr/local/bin/minio https://dl.minio.io/server/minio/release/linux-amd64/minio
`
`
chmod +x /usr/local/bin/minio
`
After install Minio on every instance(3 instance)
Create systemd file
`
cat > /lib/systemd/system/minio.service << EOF
[Unit]
Description=minio
Documentation=https://docs.min.io
Wants=network-online.target
After=network-online.target
AssertFileIsExecutable=/usr/local/bin/minio
[Service]
WorkingDirectory=/usr/local/
User=root
Group=root
EnvironmentFile=/etc/default/minio
ExecStart=/usr/local/bin/minio server $MINIO_OPTS $MINIO_VOLUMES
Restart=always
LimitNOFILE=65536
TimeoutStopSec=infinity
SendSIGKILL=no
[Install]
WantedBy=multi-user.target
EOF
`
Set env on every instance(3 instance), ACCESS and SECRET can set whatever you want.
`
cat > /etc/default/minio << EOF
MINIO_OPTS="--console-address :9001"
MINIO_VOLUMES="http://minio1:9000/data http://minio2:9000/data http://minio3:9000/data"
MINIO_ACCESS_KEY="<minioadmin>"
MINIO_SECRET_KEY="<minioadmin>"
EOF
`
Now we ready to start minio server on every instance.
Update config with deamon-reoad and start minio server
`
systemctl daemon-reload
systemctl enable minio
systemctl start minio.service
`
Check Service Status
`
systemctl status minio.service
`
`
● minio.service - minio
   Loaded: loaded (/usr/lib/systemd/system/minio.service; enabled; vendor preset: disabled)
   Active: active (running) since Fri 2024-05-24 11:11:59 +07; 50min ago
     Docs: https://docs.min.io
 Main PID: 19886 (minio)
   CGroup: /system.slice/minio.service
           └─19886 /usr/local/bin/minio server --console-address :9001 http://minio1:9000/data http://minio2:9000/data http://minio3:9000/data
May 24 11:18:39 minio1 minio[19886]: UserAgent: Go-http-client/1.1
May 24 11:18:39 minio1 minio[19886]: Error: remote disconnected (grid.RemoteErr)
May 24 11:18:39 minio1 minio[19886]: peerAddress="minio3:9000"
May 24 11:18:39 minio1 minio[19886]: 6: internal/logger/logger.go:268:logger.LogIf()
May 24 11:18:39 minio1 minio[19886]: 5: cmd/logging.go:18:cmd.iamLogIf()
May 24 11:18:39 minio1 minio[19886]: 4: cmd/iam.go:691:cmd.(*IAMSys).notifyForUser()
May 24 11:18:39 minio1 minio[19886]: 3: cmd/iam.go:751:cmd.(*IAMSys).SetTempUser()
May 24 11:18:39 minio1 minio[19886]: 2: cmd/sts-handlers.go:309:cmd.(*stsAPIHandlers).AssumeRole()
May 24 11:18:39 minio1 minio[19886]: 1: net/http/server.go:2166:http.HandlerFunc.ServeHTTP()
May 24 11:19:28 minio1 minio[19886]: Client 'http://minio3:9000/minio/storage/data/v57' re-connected in 3m24.3522277s
`
Try to access PrivateIpv4:9000 on web browser
![image](https://github.com/dagonbo/minio-node-distributed-cluster/assets/51602389/e3a61680-031e-45e2-983a-df3fef24ebbe)

Username, Password is ACESS_KEY, SECRET_KEY
Try to create bucket -> add file -> check file is avaliable on another minio server PrivateIpv4:9000
If them pass, the last is stop service on some instance
'
systemctl stop minio.service
' 
Try to add file to bucket on avaliable minio server -> Back to start minio.service on instance that we stop -> check the last file added on minio server
'
systemctl start minio.service
' 


