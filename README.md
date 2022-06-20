# How To Set Up MinIO Object Storage Server Site Active-Active Replication on Ubuntu 20.04 LTS
## Table of contents
  - [Pre-requisite](#pre-requisite)
  - [Step 1 — Downloading and Installing the MinIO Server](#step-1--downloading-and-installing-the-minio-server)
  - [Step 2 — Creating the MinIO User, Group, Data Directory, and Environment File](#step-2--creating-the-minio-user-group-data-directory-and-environment-file)
  - [Step 3 — Setting the Firewall to Allow MinIO Traffic](#step-3--setting-the-firewall-to-allow-minio-traffic)
  - [Step 4 — Starting the MinIO Server](#step-4--starting-the-minio-server)
  - [Step 5 — Connecting to MinIO Server via the MinIO Console](#step-5--connecting-to-minio-server-via-the-minio-console)
  - [Step 6 — Installing and Using the MinIO Client](#step-6--installing-and-using-the-minio-client)
  - [Step 7 — Setup Replicate](#step-7--setup-replicate)
  - [Step 8 — Testing replicate](#step-8--testing-replicate)
  - [Step 9 Setup secure connect to MinIO Console](#step-9-setup-secure-connect-to-minio-console)
## Pre-requisite
@minio-01,02      
0.1 Prepare server
```shell
#OS Ubuntu 20.04 LTS
#server-name private-ip public-ip 
minio-01 10.50.128.8 40.65.137.16
minio-02 10.50.128.9 52.148.71.42
```
0.2 Config Domain name @Cloudflare
```shell
minio-01.yourdomain.com 40.65.137.16
minio-02.yourdomain.com 52.148.71.42
```
0.3 Setup timezone and sync
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
## Step 1 — Downloading and Installing the MinIO Server   
@minio-01,02
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
## Step 2 — Creating the MinIO User, Group, Data Directory, and Environment File   
@minio-01,02
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
## Step 3 — Setting the Firewall to Allow MinIO Traffic   
@minio-01,02   
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
## Step 4 — Starting the MinIO Server   
@minio-01,02
```shell
sudo systemctl start minio
sudo systemctl status minio
```

## Step 5 — Connecting to MinIO Server via the MinIO Console  
Point your browser to http://your-server-ip:9001
```shell
http://minio-01.yourdomain.com:9001
http://minio-02.yourdomain.com:9001
```
## Step 6 — Installing and Using the MinIO Client   
@minio-01
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
mcli alias set minio-01/ http://10.50.128.8:9000 minioadmin minioadmin
mcli alias set minio-02/ http://10.50.128.9:9000 minioadmin minioadmin

#Verfity
mcli --insecure admin info minio-01
●  10.50.128.8:9000
   Uptime: 34 minutes
   Version: 2022-06-11T19:55:32Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1st

1 drive online, 0 drives offline

mcli --insecure admin info minio-02
●  10.50.128.9:9000
   Uptime: 12 minutes
   Version: 2022-06-11T19:55:32Z
   Network: 1/1 OK
   Drives: 1/1 OK
   Pool: 1st
```
## Step 7 — Setup Replicate   
@minio-01,02   
7.1 Set Replicate
```shell
 mcli admin replicate add minio-01 minio-02
```
7.2 Check replicate status
```shell
mcli admin replicate info minio-01
```
Output:
```shell
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint
87f39e99-eef4-4bf5-acea-fcfdbc9e9ac8 | minio-01    | http://10.50.128.8:9000
81879d23-f001-4c25-a634-4202a0434a79 | minio-02    | http://10.50.128.9:9000
```
```shell
mcli admin replicate info minio-02
```
Output:
```shell
SiteReplication enabled for:

Deployment ID                        | Site Name       | Endpoint
87f39e99-eef4-4bf5-acea-fcfdbc9e9ac8 | minio-01    | http://10.50.128.8:9000
81879d23-f001-4c25-a634-4202a0434a79 | minio-02    | http://10.50.128.9:9000
```
## Step 8 — Testing replicate   
@minio-01 server or MinIO CONSOLE
```shell
#Test Create bucket and object
touch test.txt
mcli mb minio-01/bucket1
mcli cp test.txt minio-01/bucket1
mcli ls minio-01/bucket1
[2022-06-20 19:27:34 +07]     0B test.txt
mcli ls minio-02/bucket1
[2022-06-20 19:27:34 +07]     0B test.txt
#Test Delete object in bucket
mcli rm --recursive --versions --force minio-01/bucket1
mcli ls minio-01
#Remove bucket
mcli mb minio-01/bucket1
```
> MinIO Client Complete Guide: https://nm-muzi.com/docs/minio-client-complete-guide.html
## Step 9 Setup secure connect to MinIO Console   
@minio-01,02   
9.1 Install nginx
```shell
sudo apt install nginx
```
9.2 Create nginx config
File: minio.conf
```shell
cd /etc/nginx/conf.d
vi minio.conf
---
server {
    server_name poc-minio-0x.yourdomain.com;
    charset utf-8;
    client_max_body_size 100M;
    location / {
        proxy_set_header HOST $host;
    	proxy_http_version 1.1;
    	proxy_set_header Upgrade $http_upgrade;
    	proxy_set_header Connection "Upgrade";
        proxy_set_header X-Forwarded-Proto $scheme;
        proxy_set_header X-Real-IP $remote_addr;
	proxy_pass http://localhost:9001;
    }
    error_page  500 502 503 504  /50x.html;
    location = /50x.html {
        root  /usr/share/nginx/html;
    }
}
----
```
9.3 Check and reload the new config
```shell
#Check config
sudo nginx -t
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
#REload config
sudo nginx -s reload
```
9.4 Install certbot
```shell
sudo apt install certbot python3-certbot-nginx
```
9.5 turning on HTTPS access for minio-01.yourdomain.com 
```shell
sudo certbot --nginx -d minio-01.yourdomain.com
Saving debug log to /var/log/letsencrypt/letsencrypt.log
Plugins selected: Authenticator nginx, Installer nginx
Enter email address (used for urgent renewal and security notices) (Enter 'c' to
cancel): kittisuw@gmail.com

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Please read the Terms of Service at
https://letsencrypt.org/documents/LE-SA-v1.2-November-15-2017.pdf. You must
agree in order to register with the ACME server at
https://acme-v02.api.letsencrypt.org/directory
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(A)gree/(C)ancel: A

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Would you be willing to share your email address with the Electronic Frontier
Foundation, a founding partner of the Let's Encrypt project and the non-profit
organization that develops Certbot? We'd like to send you email about our work
encrypting the web, EFF news, campaigns, and ways to support digital freedom.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
(Y)es/(N)o: Y
Obtaining a new certificate
Performing the following challenges:
http-01 challenge for minio-01.yourdomain.com
Waiting for verification...
Cleaning up challenges
Deploying Certificate to VirtualHost /etc/nginx/conf.d/minio.conf

Please choose whether or not to redirect HTTP traffic to HTTPS, removing HTTP access.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
1: No redirect - Make no further changes to the webserver configuration.
2: Redirect - Make all requests redirect to secure HTTPS access. Choose this for
new sites, or if you're confident your site works on HTTPS. You can undo this
change by editing your web server's configuration.
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Select the appropriate number [1-2] then [enter] (press 'c' to cancel): 2
Redirecting all traffic on port 80 to ssl in /etc/nginx/conf.d/minio.conf

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Congratulations! You have successfully enabled
https://minio-01.yourdomain.com

You should test your configuration at:
https://www.ssllabs.com/ssltest/analyze.html?d=minio-01.yourdomain.com
...
```
> This step require open port 80,443 to public

9.6 Check current certificae
```shell
sudo certbot certificates
```
Output:
```shell
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Found the following certs:
  Certificate Name: minio-01.yourdomain.com
    Domains: minio-01.yourdomain.com
    Expiry Date: 2022-09-18 05:32:52+00:00 (VALID: 89 days)
    Certificate Path: /etc/letsencrypt/live/minio-01.yourdomain.com/fullchain.pem
    Private Key Path: /etc/letsencrypt/live/minio-01.yourdomain.com/privkey.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
9.7 Test automatic renewal
> By default certificate will be valid for 90 days from the date of renewal and Certbot will be renewal every 12 Hrs.
```shell
sudo certbot renew --dry-run
```
```shell
Saving debug log to /var/log/letsencrypt/letsencrypt.log

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Processing /etc/letsencrypt/renewal/minio-01.yourdomain.com.conf
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
Cert not due for renewal, but simulating renewal for dry run
Plugins selected: Authenticator nginx, Installer nginx
Renewing an existing certificate
Performing the following challenges:
http-01 challenge for minio-01.yourdomain.com
Waiting for verification...
Cleaning up challenges

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
new certificate deployed with reload of nginx server; fullchain is
/etc/letsencrypt/live/minio-01.yourdomain.com/fullchain.pem
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -

- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates below have not been saved.)

Congratulations, all renewals succeeded. The following certs have been renewed:
  /etc/letsencrypt/live/minio-01.yourdomain.com/fullchain.pem (success)
** DRY RUN: simulating 'certbot renew' close to cert expiry
**          (The test certificates above have not been saved.)
- - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - - -
```
9.7 Test brow to URL if success http will be redirect to https and poit to MinIO CONSOLE   
http://minio-01.yourdomain.com   
http://minio-02.yourdomain.com

# Reference
Multi-Site Active-Active Replication: https://blog.min.io/minio-multi-site-active-active-replication/   
How to use Replication to protect data from storage failures: https://youtu.be/gFlif9RGeHg   
Side replicate overview : https://docs.min.io/minio/baremetal/replication/site-replication-overview.html#minio-site-replication-overview
