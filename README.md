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
