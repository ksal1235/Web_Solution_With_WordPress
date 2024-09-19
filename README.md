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

Create a new partition
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
26. 
