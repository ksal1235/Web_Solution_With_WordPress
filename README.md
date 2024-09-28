# Web_Solution_With_WordPress
In this project you will be tasked to prepare storage infrastructure on two Linux servers and implement a basic web solution using WordPress. WordPress is a free and open-source content management system written in PHP and paired with MySQL or MariaDB as its backend Relational Database Management System (RDBMS).

This Project consists of two parts:
1. Configure storage subsystem for Web and Database servers based on Linux OS. The focus of this part is to give practical experience of working with disks, partitions and volumes in Linux.
2. Install WordPress and connect it to a remote MySQL database server. This part of the project will solidify our skills of deploying Web and DB tiers of Web solution.

# Three-tier Architecture:
Generally, web, or mobile solutions are implemented based on what is called the Three-tier Architecture.

### Three-tier Architecture is a client-server software architecture pattern that comprise of 3 separate layers.

  ![image](https://github.com/user-attachments/assets/c70ca91b-8f7b-4412-bed6-3135783ca42b)

1. Presentation Layer (PL): This is the user interface such as the client server or browser on laptop.
2. Business Layer (BL): This is the backend program that implements business logic. Application or Webserver.
3. Data Access or Management Layer (DAL): This is the layer for data storage and data access. Database Server or File System Server such as FTP server, or NFS Server.

In this project, we will have the hands-on experience that showcases Three-tier Architecture while also ensuring that the disks used to store files on the Linux servers are adequately partitioned and managed through programs such as gdisk and LVM respectively.

### Our 3-Tier Setup Need:
1. A Laptop or PC to serve as a client.
2. An EC2 Linux Server as a web server (This is where you will install WordPress).
3. An EC2 Linux server as a database (DB) server.

### We will Use RedHat OS for this project.
By now we should know how to spin up an EC2 instanse on AWS, In previous projects we used 'Ubuntu', but it is better to be well-versed with various Linux distributions, thus, for this projects we will use very popular distribution called 'RedHat' (it also has a fully compatible derivative - CentOS)

#### Note: for Ubuntu server, when connecting to it via SSH/Putty or any other tool, we used ubuntu user, but for RedHat you will need to use ec2-user user. Connection string will look like ec2-user@<Public-IP>.

# Step 1 — Prepare a Web Server.

1. Launch an EC2 instance that will serve as "Web Server" and Create 3 volumes in the same AZ as Web Server EC2, each of 10 GiB.

![image](https://github.com/user-attachments/assets/c77a1830-4ad5-4907-8243-c5155f38c8a0)

2. Connect to your instance via ssh
```
  ssh -i <key-pair-name> ec2-user@<ip-address>
```
![image](https://github.com/user-attachments/assets/24b31fb8-ebb7-47c9-a1d3-32f9d809f127)

3. Creating 3 volumes in the same AZ as Web Server EC2, each of 10 GiB.

On same page of EC2 we have to scroll down and Have to Click on Snapshot.
![image](https://github.com/user-attachments/assets/bdac19d3-6deb-4026-afb4-4473a6351f94)


![image](https://github.com/user-attachments/assets/bdcb7285-ec38-4583-812d-5dd6e266a016)

Have to Create the 3 volume with the same setting.
![image](https://github.com/user-attachments/assets/948781c0-6254-40ec-b75b-22a01a22dc44)

![image](https://github.com/user-attachments/assets/d93676f2-0b96-4315-a2b6-febeb4577b1e)

Now all this 3 Newly Created Volume Need to attach on Existing VMs.

4. Use lsblk command to inspect what block devices are attached to the server.

![image](https://github.com/user-attachments/assets/b5314bfa-6ee8-4062-ad3d-0241cb3c43cb)

The attache EBS Volume their names will likely be xvdbf xvdbg xvdbh.

5. Use gdisk utility to create a single partition on each of the 3 disks.

```
sudo gdisk /dev/xvdbf
```

Create a new partition:

a) Type n to create a new partition.
b) Press Enter to select the default partition number
c) Press Enter to accept the default last sector.
d) Choose the partition type by typing 8300 for a Linux filesystem (ext4).
e) To Write changes to the disk, Once the partition is created, type w

Have to Do above step for all 3 disk xvdbf xvdbg xvdbh.

6. Use lsblk utility to view the newly configured partition on each of the 3 disks.
 ![image](https://github.com/user-attachments/assets/946c92cd-71c6-4be4-a601-0b724f7a5cdf)

7. Install lvm2 package using sudo yum install lvm2. Run sudo lvmdiskscan command to check for available partitions.

```
  sudo yum install lvm2 -y
```
8. Use pvcreate utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM.

```
sudo pvcreate /dev/xvdbf1
sudo pvcreate /dev/xvdbg1
sudo pvcreate /dev/xvdbh1

```

![image](https://github.com/user-attachments/assets/456df784-d45c-4824-be0d-c644a1cb8759)

9. Verify that Physical volume has been created successfully by running sudo pvs.

```
sudo pvs
```
![image](https://github.com/user-attachments/assets/3436be6e-c1fa-41f6-9927-9508c4ffa411)


10. Use vgcreate utility to add all 3 PVs to a volume group (VG). Name the VG webdata-vg

```
sudo vgcreate webdata-vg /dev/xvdbh1 /dev/xvdbg1 /dev/xvdbf1
```

11. Verify that your VG has been created successfully by running sudo vgs
```
sudo vgs
```
![image](https://github.com/user-attachments/assets/1319fbc6-7dc1-4ab8-abea-a155363aecae)

12. Use lvcreate utility to create 2 logical volumes. apps-lv (Use half of the PV size), and logs-lv Use the remaining space of the PV size. NOTE: apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.
```
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-vg
```
![image](https://github.com/user-attachments/assets/729b9a87-11ac-48dd-ac45-310165aca8dc)

13. Verify that your Logical Volume has been created successfully.
```
sudo lvs
```
![image](https://github.com/user-attachments/assets/69f60fab-e250-4e44-94b8-461e8864d84b)

14. Verify the entire setup
```
sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk
```
![image](https://github.com/user-attachments/assets/4df32d5e-384b-4fa8-a4df-cd71775b24cd)


15. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

```
    sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
    sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
```

16. Create /var/www/html directory to store website files sudo mkdir -p /var/www/html.
17. Create /home/recovery/logs to store backup of log data sudo mkdir -p /home/recovery/logs.
18. Mount /var/www/html on apps-lv logical volume.
```
sudo mount /dev/webdata-vg/apps-lv /var/www/html/
```
20. Use rsync utility to backup all the files in the log directory /var/log into /home/recovery/logs (This is required before mounting the file system).
```
sudo rsync -av /var/log/ /home/recovery/logs/
```
21. Mount /var/log on logs-lv logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very important).
```
sudo mount /dev/webdata-vg/logs-lv /var/log
```

22. Restore log files back into /var/log directory
```
sudo rsync -av /home/recovery/logs/ /var/log
```
23. Update /etc/fstab file so that the mount configuration will persist after restart of the server.
```
sudo blkid
```

```
sudo vi /etc/fstab
```
![image](https://github.com/user-attachments/assets/3837a03e-e274-4251-a1b8-6c7350721ee8)


Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
24. Test the configuration and reload the daemon.
```
sudo mount -a
sudo systemctl daemon-reload
```
25. Verify your setup by running df -h, output must look like this:
```
df -h
```
![image](https://github.com/user-attachments/assets/fdcc297b-87b4-47d8-ae71-797cebb56826)

# Step 2 — Prepare the Database Server
NOTE: Launch a second RedHat EC2 instance that will have a role - 'DB Server' Repeat the same steps as for the Web Server, but instead of apps-lv create db-lv and mount it to /db directory instead of /var/www/html/.

![image](https://github.com/user-attachments/assets/915b81ad-747c-4413-b7c3-fdc0e02dba71)


Step 3 — Install WordPress on your Web Server EC2 Which we have already Setup Before.
1. Update the repository.
```
sudo yum -y update
```
![image](https://github.com/user-attachments/assets/6bf73e6e-4c60-4c5b-b276-2c8ce8bca3e5)

2. Install wget, Apache and it's dependencies.
```
sudo yum -y install wget httpd php php-mysqlnd php-fpm php-json
```

3. Start apache.
```
sudo systemctl enable httpd && sudo systemctl start httpd
```
4. To install PHP and it's dependencies.
```
sudo yum install https://dl.fedoraproject.org/pub/epel/epel-release-latest-8.noarch.rpm
sudo yum install yum-utils http://rpms.remirepo.net/enterprise/remi-release-8.rpm
sudo yum module list php && sudo yum module reset php
sudo yum install php php-opcache php-gd php-curl php-mysqlnd
sudo systemctl start php-fpm
sudo systemctl enable php-fpm
sudo setsebool -P httpd_execmem 1
```
5. Restart Apache
```
sudo systemctl restart httpd
```
6. Download wordpress and copy wordpress to /var/www/html mkdir wordpress cd wordpress
```
sudo wget http://wordpress.org/latest.tar.gz && sudo tar xzvf latest.tar.gz
sudo rm -rf latest.tar.gz
sudo cp wordpress/wp-config-sample.php wordpress/wp-config.php
sudo cp -R wordpress /var/www/html/
sudo chown -R apache:apache /var/www/html/wordpress
sudo chcon -t httpd_sys_rw_content_t /var/www/html/wordpress -R
sudo setsebool -P httpd_can_network_connect=1
```

# Step 4 — Install MySQL on your DB Server EC2.

Update the package
```
sudo yum update
```
```
sudo yum install mysql-server
```
```
sudo systemctl restart mysqld
```
```
sudo systemctl enable mysqld
```
# Step 5 — Configure DB to work with WordPress

![image](https://github.com/user-attachments/assets/f291c10f-ad85-4558-a53f-8178642804b9)

```
sudo mysql
CREATE DATABASE wordpress;
CREATE USER `myuser`@`<Web-Server-Private-IP-Address>` IDENTIFIED BY 'mypass';
GRANT ALL ON wordpress.* TO 'myuser'@'<Web-Server-Private-IP-Address>';
FLUSH PRIVILEGES;
SHOW DATABASES;
exit
```

# Step 6 — Configure WordPress to connect to remote database.

hint: Do not forget to open MySQL port 3306 on DB Server EC2. For extra security, you shall allow access to the DB server ONLY from your Web Server's IP address, so in the Inbound Rule configuration specify source as /32
![image](https://github.com/user-attachments/assets/094f38e5-a99a-4ffa-9856-e90f0cf46ce4)

1. Install MySQL client and test that you can connect from your Web Server to your DB server by using mysql-client.

```
sudo yum install mysql
sudo mysql -u admin -p -h <DB-Server-Private-IP-address>
```
2. Change permissions and configuration so Apache could use WordPress:

After Login to databases
```
show databases;
```
![image](https://github.com/user-attachments/assets/65bb3154-c461-4113-bb9e-bc95ab36fa97)

4. Change permissions and configuration so Apache could use WordPress:

To configure Apache to use WordPress, you'll need to adjust permissions and make some configuration changes. Here's a step-by-step guide:

a. Set correct ownership:

```
sudo chown -R apache:apache /var/www/html/wordpress
Set correct permissions:
```
```
sudo find /var/www/html/wordpress -type d -exec chmod 755 {} \;
sudo find /var/www/html/wordpress -type f -exec chmod 644 {} \;
Make sure Apache can write to wp-content:
````
sudo chmod -R 775 /var/www/html/wordpress/wp-content
b. Configure Apache:

c. Edit your Apache configuration file (often /etc/httpd/conf/httpd.conf or /etc/apache2/sites-available/000-default.conf):
```
sudo nano /etc/httpd/conf/httpd.conf
```
d. Add or modify the following:
```
<Directory /var/www/html/wordpress>
    AllowOverride All
</Directory>
```
e. Enable mod_rewrite:

```
sudo a2enmod rewrite
```
f. Restart Apache:
```
sudo systemctl restart httpd
or

sudo service apache2 restart
```
d. Create a wp-config.php file:
```
cp /var/www/html/wordpress/wp-config-sample.php /var/www/html/wordpress/wp-config.php
```

e. Edit wp-config.php with your database details:
```
sudo nano /var/www/html/wordpress/wp-config.php
```

f. Update these lines:
```
define('DB_NAME', 'your_database_name');
define('DB_USER', 'your_database_user');
define('DB_PASSWORD', 'your_database_password');
define('DB_HOST', 'localhost');
```
g. Set proper SELinux contexts (if SELinux is enabled):
```
sudo chcon -R -t httpd_sys_content_t /var/www/html/wordpress
sudo chcon -R -t httpd_sys_rw_content_t /var/www/html/wordpress/wp-content
```
h. If using a firewall, ensure port 80 (and 443 for HTTPS) is open:
```
sudo firewall-cmd --permanent --add-service=http
sudo firewall-cmd --permanent --add-service=https
sudo firewall-cmd --reload
```
i. Now configuring the mysql
```
sudo nano /etc/my.cnf
```

Adding Bind Ip Block;
```
[mysqld]
bind-address = 0.0.0.0
```

4. Try to access from your browser the link to your WordPress http://<Web-Server-Public-IP-Address>/wordpress/

![image](https://github.com/user-attachments/assets/da9e413c-bd43-4d4f-bfe2-549e8fc84dbd)


# CONCLUSION

This project demonstrates the successful implementation of a scalable and robust web solution using WordPress, the world's most popular content management system. By leveraging Amazon Web Services (AWS) EC2 instances, we've created a flexible and powerful hosting environment for WordPress.

#### Key achievements of this project include:

- Successful setup and configuration of an EC2 instance to host both the web server and database server.
- Implementation of a three-tier storage architecture using Logical Volume Management (LVM), enhancing storage flexibility and scalability.
- Installation and configuration of Apache web server to serve WordPress content efficiently.
- Setup of a MySQL database to store WordPress data securely.
- Installation and configuration of WordPress, making it ready for content creation and management.
- Implementation of necessary security measures, including proper file permissions and database access controls.
- Throughout this project, we encountered and overcame various challenges, from storage management to database connectivity issues. These experiences have provided valuable insights into system administration, web hosting, and database management.

#### This solution offers several benefits:

- Scalability: The use of LVM allows for easy storage expansion as the website grows.
- Flexibility: WordPress provides a user-friendly interface for content management, suitable for users with varying technical expertise.
- Cost-effectiveness: Utilizing AWS EC2 instances allows for pay-as-you-go pricing and easy resource scaling.

#### Moving forward, potential improvements could include:

- Implementing a content delivery network (CDN) for improved global performance.
- Setting up regular backups to ensure data safety.
- Implementing additional security measures such as SSL certificates and web application firewalls.
- This project serves as a solid foundation for hosting WordPress-based websites and can be adapted or expanded to meet various web hosting needs.
