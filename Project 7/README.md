### **DEVOPS TOOLING WEBSITE SOLUTION**

- In previous Project 6 you implemented a WordPress-based solution that is ready to be filled with content and can be used as a full-fledged website or blog. Moving further we will add some more value to our solutions that your DevOps team could utilize. We want to introduce a set of DevOps tools that will help our team in day-to-day activities in managing, developing, testing, deploying and monitoring different projects.

The tools we want our team to be able to use are well-known and widely used by multiple DevOps teams, so we will introduce a single DevOps Tooling Solution that will consist of:

1. Jenkins : Free and open source automation server used to build CI/CD pipelines.
2. Kubernetes – An open-source container-orchestration system for automating computer application deployment, scaling, and management.
3. Jfrog Artifactory – Universal Repository Manager supporting all major packaging formats, build tools and CI servers. Artifactory.
4. Rancher – an open-source software platform that enables organizations to run and manage Docker and Kubernetes in production.  5. Grafana – a multi-platform open-source analytics and interactive visualization web application.
6. Prometheus – An open-source monitoring system with a dimensional data model, flexible query language, efficient time series database and modern alerting approach.
7. Kibana – Kibana is a free and open user interface that lets you visualize your Elasticsearch data and navigate the Elastic Stack.

**Note:** Do not feel overwhelmed by all the tools and technologies listed above, we will gradually get ourselves familiar with them in upcoming projects!


##### **SETUP AND TECHNOLOGIES USED IN PROJECT 7**

- As a member of a DevOps team, you will implement a tooling website solution which makes access to DevOps tools within the corporate infrastructure easily accessible. In this project you will implement a solution that consists of following components:

1. Infrastructure: AWS
2. Webserver Linux: Red Hat Enterprise Linux 8
3. Database Server: Ubuntu 20.04 + MySQL
4. Storage Server: Red Hat Enterprise Linux 8 + NFS Server
5. Programming Language: PHP
6. Code Repository: GitHub.

On the diagram below you can see a common pattern where several stateless Web Servers share a common database and also access the same files using Network File Sytem (NFS) as a shared file storage. Even though the NFS server might be located on a completely separate hardware – for Web Servers it look like a local file system from where they can serve the same files.

![alt text](<Images/3 tier web.png>)

It is important to know what storage solution is suitable for what use cases, for this – you need to answer the following questions: what data will be stored, in what format, how this data will be accessed, by whom, from where, how frequently, etc. Based on this you will be able to choose the right storage system for your solution.

#### **STEP 1 – PREPARE NFS SERVER**


- **Create NFS Instane-** Spin up a new EC2 instance with RHEL Linux 8 Operating System.


- **Configure LVM on the Server-** Create 3 volumes with:

1. Size: (e.g., 10 GiB each)
2. Volume type: gp3 (default)

3. Availability Zone: Same as your NFS server, attach the volumes, choose NFS instance, and set device names.
4. Install LVM `sudo dnf install lvm2 -y`

5. Create Physical Volumes, `sudo pvcreate /dev/xvdf /dev/xvdg /dev/xvdh`

6. Create Volume Group, `sudo vgcreate nfs-vg /dev/xvdf /dev/xvdg /dev/xvdh`

7. Create Logical Volumes, 
```
sudo lvcreate -n lv-apps -L 5G nfs-vg
sudo lvcreate -n lv-logs -L 5G nfs-vg
sudo lvcreate -n lv-opt -l 100%FREE nfs-vg
```

![alt text](<Images/Screenshot 2026-03-25 160744.png>)

8. Format (Use XFS) 
```
sudo mkfs.xfs /dev/nfs-vg/lv-apps
sudo mkfs.xfs /dev/nfs-vg/lv-logs
sudo mkfs.xfs /dev/nfs-vg/lv-opt
```

9. Create Mount Points,
```
sudo mkdir -p /mnt/apps
sudo mkdir -p /mnt/logs
sudo mkdir -p /mnt/opt
``` 

10. Mount Volumes, 
```
sudo mount /dev/nfs-vg/lv-apps /mnt/apps
sudo mount /dev/nfs-vg/lv-logs /mnt/logs
sudo mount /dev/nfs-vg/lv-opt /mnt/opt
```
11. Make mount permanent: 

`sudo blkid`; 

Copy UUIDs and edit /etc/fstab file ; `sudo vi /etc/fstab`

Then add:

 ```
UUID=xxx  /mnt/apps  xfs  defaults,nofail  0 0
UUID=xxx  /mnt/logs  xfs  defaults,nofail  0 0
UUID=xxx  /mnt/opt   xfs  defaults,nofail  0 0
```

- **Install NFS Server**

1. Run:
```
sudo dnf install nfs-utils -y
sudo systemctl enable nfs-server
sudo systemctl start nfs-server
```
2. Check:
`sudo systemctl status nfs-server`

![alt text](<Images/Screenshot 2026-03-25 161336.png>)

3. Set Permissions: 
```
sudo chown -R nobody:nobody /mnt/apps
sudo chown -R nobody:nobody /mnt/logs
sudo chown -R nobody:nobody /mnt/opt

sudo chmod -R 777 /mnt/apps
sudo chmod -R 777 /mnt/logs
sudo chmod -R 777 /mnt/opt
```

![alt text](<Images/Screenshot 2026-03-25 162139.png>)

4. Restart NFS: `sudo systemctl restart nfs-server`

5. Configure Exports:

Get your Subnet CIDR from AWS, 

![alt text](<Images/Screenshot 2026-03-25 153248.png>)

Edit:
`sudo vi /etc/exports`

Add:

```
/mnt/apps CIDR(rw,sync,no_root_squash)
/mnt/logs CIDR(rw,sync,no_root_squash)
/mnt/opt  CIDR(rw,sync,no_root_squash)
```

![alt text](<Images/Screenshot 2026-03-25 152954.png>)

Apply the config: `sudo exportfs -rav`

6. Open Required Ports on NFS:
 Since we are on **NFSv4**, we will need Port 2049 (Custom TCP), with subnet as Source.
 
 ![alt text](<Images/Screenshot 2026-03-25 154058.png>)

 7. Verify NFS: `sudo exportfs -v`


 #### **STEP 2 - CONFIGURE THE DATABASE SERVER**

1. Install MariaDB server : `sudo yum install mariadb-server -y`

![alt text](<Images/Screenshot 2026-03-25 171621.png>)

2. Start & Enable:

```
sudo systemctl start mariadb
sudo systemctl enable mariadb
sudo systemctl status mariadb
```
![alt text](<Images/Screenshot 2026-03-25 171925.png>)

3. Secure Installation: `sudo mysql_secure_installation`


Login :`sudo mysql -u root -p`

 Create Database : `CREATE DATABASE tooling;`

Create User : `CREATE USER 'webaccess'@'172.31.%' IDENTIFIED BY 'StrongPassword123';`

Grant Permissions : `GRANT ALL PRIVILEGES ON tooling.* TO 'webaccess'@'172.31.%';`

`FLUSH PRIVILEGES;`

`SHOW DATABASES;`

`exit`

![alt text](<Images/Screenshot 2026-03-25 173319.png>)

**Note:** Why `'172.31.%'` is Used Instead of Full IP because host does not support it so `'172.31.%'` 
*Means; Allow connections from any IP starting with 172.31*

#### **STEP 3 - PREPARE THE WEB SERVERS**

- We need to make sure that our Web Servers can serve the same content from shared storage solutions, in our case – NFS Server and MySQL database. You already know that one DB can be accessed for reads and writes by multiple clients. For storing shared files that our Web Servers will use – we will utilize NFS and mount previously created Logical Volume `lv-apps` to the folder where Apache stores files to be served to the users (/var/www). This approach will make our Web Servers stateless, which means we will be able to add new ones or remove them whenever we need, and the integrity of the data (in the database and on NFS) will be preserved.

**NOTE:** During the next steps we will do following:

- Configure NFS client (this step must be done on all three servers)
- Deploy a Tooling application to our Web Servers into a shared NFS folder
- Configure the Web Servers to work with a single MySQL database.

1. Launch a new EC2 instance with RHEL 8 Operating System.

2. Install NFS client :
`sudo yum install nfs-utils nfs4-acl-tools -y`

3. Mount `/var/www/` and target the NFS server’s export for apps :

`sudo mkdir /var/www`

`sudo mount -t nfs -o rw,nosuid <NFS-Server-Private-IP-Address>:/mnt/apps /var/www`

-  Verify that NFS was mounted successfully by running: `df -h`.

4. Make sure that the changes will persist on Web Server after reboot: `sudo nano /etc/fstab`

- Add following line :

`<NFS-Server-Private-IP-Address>:/mnt/apps /var/www nfs defaults 0 0`

5. Install `Remi’s repository`, `Apache` and `PHP`

```
sudo yum install httpd -y
 
sudo dnf install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
 
sudo dnf install dnf-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
 
sudo dnf module reset php
 
sudo dnf module enable php:remi-7.4
 
sudo dnf install php php-opcache php-gd php-curl php-mysqlnd
 
sudo systemctl start php-fpm
 
sudo systemctl enable php-fpm
 
setsebool -P httpd_execmem 1
```

![alt text](<Images/Screenshot 2026-04-08 120142.png>)

![alt text](<Images/Screenshot 2026-04-08 120313.png>)
![alt text](<Images/Screenshot 2026-04-08 121118.png>)

![alt text](<Screenshot 2026-04-08 121247.png>)

**Repeat steps 1-5 for another 2 Web Servers.**

6. Verify that Apache files and directories are available on the Web Server in /var/www and also on the NFS server in `/mnt/apps`. If you see the same files – it means NFS is mounted correctly. You can try to create a new file `touch test.txt` from one server and check if the same file is accessible from other Web Servers.
![alt text](<Images/Screenshot 2026-04-08 123637.png>)

7. Locate the log folder for Apache on the Web Server and mount it to NFS server’s export for logs. Repeat step №4 to make sure the mount point will persist after reboot.

8. Fork the tooling source code from [HERE](https://github.com/theboyfromsalem/tooling) to your Github account.  (Learn how to fork a repo [here](https://youtu.be/f5grYMXbAV0?si=rbyxmmQ8b4Wunyba)).

9. Deploy the tooling website’s code to the Webserver. Ensure that the html folder from the repository is deployed to `/var/www/html`

![alt text](<Images/Screenshot 2026-04-08 124111.png>)

10. Update the website’s configuration to connect to the database (in /var/www/html/functions.php file).

- Apply tooling-db.sql script to your database using this command :
 `mysql -h <databse-private-ip> -u <db-username> -p <db-pasword> < tooling-db.sql`

![alt text](<Images/Screenshot 2026-04-08 124409.png>)

 11. Open the website in your browser `http://<Web-Server-Public-IP-Address-or-Public-DNS-Name>/index.php` and make sure you can login into the website with myuser user.

 ![alt text](<Images/Screenshot 2026-04-08 070424.png>)

 ![alt text](<Images/Screenshot 2026-04-08 072649.png>)

