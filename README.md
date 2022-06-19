# How To Set Up MinIO Object Storage Server replicate Mode on Ubuntu 20.04 LTS
# Step 0 — Pre-requisite
## 0.1 - Prepare server
```shell
#OS Ubuntu 20.04 LTS
#server-name private-ip public-ip 
poc-minio-01 10.50.128.8 40.65.137.16
poc-minio-02 10.50.128.9 52.148.71.42
```
## 0.2 Setup timezone and sync
```shell
change timezone to asia/bangkok
sudo timedatectl set-timezone Asia/Bangkok
#Install chrony
sudo apt install chrony -y
#Start chrony
sudo systemctl start chronyd
#check date-time
sudo timedatectl
```
# Step 1 — Downloading and Installing the MinIO Server
```shell
#Update the package database
sudo apt update
#Update the system
sudo apt upgrade
#Download the MinIO server’s latest
wget https://dl.min.io/server/minio/release/linux-amd64/minio_20220611195532.0.0_amd64.deb
#Install the downloaded file
sudo dpkg -i minio_20220611195532.0.0_amd64.deb
```
# Step 2 — Creating the MinIO User, Group, Data Directory, and Environment File
```shell
#Create a system group that the MinIO server will run
sudo groupadd -r minio-user

#Create the user that the MinIO server will run
sudo useradd -M -r -g minio-user minio-user

#Create the data directory where MinIO will store
sudo mkdir /mnt/data
#Give ownership of the data directory to the MinIO user and group
sudo chown minio-user:minio-user /mnt/data

#Create and open MinIO’s environment file:
sudo vi /etc/default/minio
```
File : /etc/default/minio
```
MINIO_VOLUMES="/mnt/data"
MINIO_OPTS="--console-address :9001"
MINIO_ROOT_USER=minioadmin
MINIO_ROOT_PASSWORD=minioadmin
```
# Step 3 — Setting the Firewall to Allow MinIO Traffic
In this step, you will configure the firewall to allow traffic into the ports that access the MinIO server and MinIO Console. The following are pertinent to MinIO:
- 9000 is the default port that the MinIO server listens on.
- 9001 is the recommended port for accessing the MinIO Console.

Allow traffic to both ports through the firewall with the following command:
```shell
sudo ufw allow 9000:9001/tcp
```
Output: 
```shell
Output
Rule added
Rule added (v6)
```
Step 4 — Starting the MinIO Server
```shell
sudo systemctl start minio
sudo systemctl status minio
```

Step 5 — Connecting to MinIO Server via the MinIO Console   
Point your browser to https://your-server-ip:9001.
```shell
http://poc-minio-01.blockfint.com:9001
http://poc-minio-02.blockfint.com:9001
```
Step 6 — Installing and Using the MinIO Client on poc-minio-01
```shell
#Download the latest MinIO client
wget https://dl.min.io/client/mc/release/linux-amd64/mcli_20220611211036.0.0_amd64.deb
#Install
sudo dpkg -i mcli_20220611211036.0.0_amd64.deb

#Enable autocompletion for your shell
mcli --autocompletion
#To enable autocompletion in your current shell without actually shutting it down and restarting it
source .profile

#Add minio server profile to mcli
mcli alias set poc-minio-01/ http://10.50.128.8:9000 minioadmin minioadmin
mcli alias set poc-minio-02/ http://10.50.128.9:9000 minioadmin minioadmin

#Verfity
mcli --insecure admin info poc-minio-01
●  10.50.128.8:9000
   Uptime: 34 minutes
   Version: 2022-06-11T19:55:32Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1st

1 drive online, 0 drives offline

mcli --insecure admin info poc-minio-02
●  10.50.128.9:9000
   Uptime: 12 minutes
   Version: 2022-06-11T19:55:32Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1st
```
Step 7 — Setup Replicate   
7.1 Set Replicate
```shell
 mcli admin replicate add poc-minio-01 poc-minio-02
```
7.2 Check replicate status
```shell
mcli admin replicate info poc-minio-01
```
Output:
```shell
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint
87f39e99-eef4-4bf5-acea-fcfdbc9e9ac8 | poc-minio-01    | http://10.50.128.8:9000
81879d23-f001-4c25-a634-4202a0434a79 | poc-minio-02    | http://10.50.128.9:9000
```
```shell
mcli admin replicate info poc-minio-02
```
Output:
```shell
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint
87f39e99-eef4-4bf5-acea-fcfdbc9e9ac8 | poc-minio-01    | http://10.50.128.8:9000
81879d23-f001-4c25-a634-4202a0434a79 | poc-minio-02    | http://10.50.128.9:9000
```
# Step 8 — Testing replicate
```shell
#Test Create bucket and object
touch test.txt
mcli mb poc-minio-01/bucket1
mcli cp test.txt poc-minio-01/bucket1
mcli ls poc-minio-01
[2022-06-16 09:44:19 UTC]     0B bucket1/
mcli ls poc-minio-02
[2022-06-16 09:44:19 UTC]     0B bucket1/

#Test Delete object in bucket
mcli rm --recursive --versions --force poc-minio-01/bucket1
mcli ls poc-minio-01
```