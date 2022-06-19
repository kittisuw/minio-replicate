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
