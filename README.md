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
```
change timezone to asia/bangkok
sudo timedatectl set-timezone Asia/Bangkok
#Install chrony
sudo apt install chrony -y
#Start chrony
sudo systemctl start chronyd
#check date-time
sudo timedatectl
```
