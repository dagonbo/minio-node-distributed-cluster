### Replica Basic Setup with GUI
#### Run minio server
```
/usr/local/bin/minio server  --console-address :9001 /data
```
#### We can specific port(9001) and drive(/data)

#### Open URL(ip:9001) and Login to homepage
#### Focus on Site Replication and Click
![image](https://github.com/dagonbo/minio-node-distributed-cluster/assets/51602389/2e7a1809-8902-4dac-86de-594b3d999389)

#### Assign This Site(Current IP:PORT) and Peer Site(Target IP:PORT) also all key of there site
![image](https://github.com/dagonbo/minio-node-distributed-cluster/assets/51602389/8fbed895-2eac-4936-9055-064f82cd08ed)

#### The result of connect replica success, show below
![image](https://github.com/dagonbo/minio-node-distributed-cluster/assets/51602389/f0b9990d-eb31-4fc9-bbd6-8be5305b7c60)

#### When we create bucket or upload file, this will replica into replicated site

![image](https://github.com/dagonbo/minio-node-distributed-cluster/assets/51602389/03ac888d-4971-4646-b6b8-087f92dbd714)

### Note: Replica don't need quorum 50% in case >50% of node not avaliable
### But it has more latency to replicated(Distributed is faster)
### In case some of node is not available, remain node still can available
### When dead node comeback!! It need time to replica new update from available node
### Replica need Storage more than Distributed*2
