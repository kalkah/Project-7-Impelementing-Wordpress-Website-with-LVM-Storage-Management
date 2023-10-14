# Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management

# IMPLEMENTING WORDPRESS WEBSITE WITH LVM STORAGE MANAGEMENT

## Implemeting LVM on Linux Servers (Web and Database Servers)

### Preparing the webserver

An EC2 instance was launched with Redhat OS, 3 EBS volume were created in the same AZ with the webserver EC2, each of 10GiB. Each of the 3 volumes were attached to the webserver EC2 instance.

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/fccc9b54-ad66-43ef-a455-b7be0b523f61)

The **`lsblk`** command was used to inspect what block of devices are attached to the server.

The command **`dh -h`** was use to view all mount and free space on the server.

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/e4b48f33-425b-4f1b-a509-895869fee5da)

The **`gdisk`** utility was used to create a single partition on each of the 3 disks

**`sudo gdisk /dev/nvme1n1`**    **`sudo gdisk /dev/nvme2n1`**    **`sudo gdisk /dev/nvme3n1`**

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/32d706ce-f40a-4b2c-84db-23b6b00e8760)

The 3 partitions are shown below...

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/27a40473-abb1-4cb2-ac5d-6b8691764569)

logical volume management was installed using **`sudo yum install lvm2`** command

**`sudo lvmdiskscan`** command was used to check for available partition.

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/9942220a-380c-4e6f-86a1-48f16c9439c2)

The **`pvcreate`** utility was used to mark each of the 3 disks as physical volumes (PV) to be used by LVM.
**`sudo pvcreate /dev/nvme1n1p1`**    **`sudo pvcreate /dev/nvme2n1p1`**    **`sudo pvcreate /dev/nvme3n1p1`**

**`sudo pvs`** commad was used to verify that the physical volume has been created and running successfully.

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/c0d973b6-f111-4a53-b251-38d88d2f34ed)

**`vgcreate`** utility was used to add all 3 PVs to a volume group (VG) named webdata-vg: **`sudo vgcreate webdata-vg /dev/nvme1n1p1 /dev/nvme2n1p1 /dev/nvme3n1p1`**

**`sudo vgs`** commad was used to verify that the physical volume group has been created and running successfully.

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/ee8300d2-3441-47c9-ba5d-1745db5f2483)

**`lvcreate`** utility was used to create 2 logical volume - apps-lv and logs-lv, each using half of the physical volume. apps-lv will be used to store data for the website and logs-lv will be used store data for the logs

**`sudo lvcreate -n apps-lv -L 5G webdata-vg`**        **`sudo lvcreate -n logs-lv -L 5G webdata-vg`**

**`sudo lvs`** commad was used to verify that the logical volume has been created and running successfully.

The entire setup is verified using the command: **sudo vgdisplay -v #view complete setup - VG, PV, and LV`**    **`sudo lsblk`**

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/f750996a-d9a6-414f-833c-a387963d280f)

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/c65bd345-1e8e-44b2-ad72-3d70ebfa65bb)
![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/d74bc184-6712-4126-bf04-94dfec1b13f3)

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/78ee56ab-c851-412b-86a5-56608b5c5a32)

**`mkfs.ext4`** command was used to format the logical volume with ext4 file system
**`sudo mkfs -t ext4 /dev/webdata-vg/apps-lv`**      **`sudo mkfs -t ext4 /dev/webdata-vg/logs-lv`**

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/445ef2f8-72eb-4198-b89a-b6f7208f13df)

`/var/www/html`directory was created to store websites files: **`sudo mkdir -p /var/www/html`**

`/home/recovery/logs` directory was created to store backup of log data: **`sudo mkdir -p /home/recovery/logs`**
 
`/var/www/html`directory was mounted on apps-lv logical volume: **`sudo mount /dev/webdata-vg/apps-lv /var/www/html/`**

`rsync` utility was used to backup all the files in the directory /var/log into /home/recovery/logs (this is require before mounting the file system)

**`sudo rsync -av /var/log/. /home/recovery/logs/`**

Mount /var/log on logs-v (NB: all the existing data on /var/log will be deleted. Hence the above step is very important): 

**`sudo mount /dev/webdata-vg/logs-lv /var/log`**

Files were restored back into /var/log directory: **`sudo rsync -av /home/recovery/logs/log/. /var/log`**

The `/etc/fstab` file was updated so that the mount configuration will persist after restart of the server. The UUID of teh device will be used to update the `/etc/fstab` file.

**`sudo blkid`**

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/9b6cb243-dd12-4865-b473-cb669cb535bc)

**`sudo nano /etc/fstab`** to edit the fstabl file

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/9d0ecb5f-93f3-458e-aac7-81841066bfd8)

**`sudo mount -a`** was use to test the configuration
**`sudo systemctl daemon-reload`** command was use to reload. 
**`df -h`** command was use to verify the setup

![image](https://github.com/kalkah/Project-7-Impelementing-Wordpress-Website-with-LVM-Storage-Management/assets/95209274/6f01d20b-418a-42de-8b05-f2e00f32f58a)

### Preparing the database server

A second EC2 instance was launched that will serve as the database server. The same steps above for the web server was repeated. instead of apps-lv, db-lv was created and mount it to /db directory instead of /var/www/html/

