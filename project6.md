### STEP 1 -PREPARE A WEB SERVER 
Launch an EC2 instance that will serve as "Web Server". Create 3 volumes in the same AZ as your Web Server EC2, each of 10 GiB.
![](/volumes.PNG)

2.attach all three volumes one by one to your web server EC2 instance
![](/attach.PNG)
Open up the Linux terminal to begin configuration

3.Use lsblk command to inspect what block devices are attached to the server. Notice names of your newly created devices. All devices in Linux reside in /dev/ directory. Inspect it with ls /dev/ and make sure you see all 3 newly created block devices there – their names will likely be xvdf, xvdh, xvdg.
![](/lsblk.PNG)

4. Use df -h command to see all mounts and free space on your server

5. Use gdisk utility to create a single partition on each of the 3 disks

`
sudo gdisk /dev/xvdf
`
![](firtt%20partion.PNG)
**NOTE** While using gdisk tool for file partioning 
use 

`
n
`

n creates new partion 

`
p
`

p-checks the selected pation 

`
8e00
`

8e00 -indicates linux partion type 

Attached all the disk using similar procedure 
Use lsblk utility to view the newly configured partition on each of the 3 disks.
![](/LSBLK%20AFTER%20ATTACHING.PNG)

Install *lvm2* package using `sudo yum install lvm2`. Run `sudo lvmdiskscan` command to check for available partitions.
7.Use `pvcreate` utility to mark each of 3 disks as physical volumes (PVs) to be used by LVM

`
sudo pvcreate /dev/xvdf1
`

`
sudo pvcreate /dev/xvdg1
`

`
sudo pvcreate /dev/xvdh1
`
8.Verify that your Physical volume has been created successfully by running `sudo pvs`
![](/sudo%20pvs.PNG)

9.Use *vgcreate* utility to add all 3 PVs to a volume group (VG). Name the VG **vg-webdata**

`
sudo vgcreate webdata-vg /dev/xvdh1 /dev/xvdg1 /dev/xvdf1
`
10.Verify that your VG has been created successfully by running **sudo vg**
![](/vgs-img.png)
11.Use `lvcreate` utility to create 2 logical volumes.**apps-lv (Use half of the PV size)**, and **logs-lv Use the remaining space of the PV size. NOTE:** apps-lv will be used to store data for the Website while, logs-lv will be used to store data for logs.

`
sudo lvcreate -n apps-lv -L 14G webdata-vg
sudo lvcreate -n logs-lv -L 14G webdata-
`

12.Verify that your Logical Volume has been created successfully by running sudo lvs
![](/lvs-logical.png)
13 Verify the entire setup 

`sudo vgdisplay -v #view complete setup - VG, PV, and LV
sudo lsblk 
`
![](/lsblk3-entire%20setup.png)
14. Use mkfs.ext4 to format the logical volumes with ext4 filesystem

`
sudo mkfs -t ext4 /dev/webdata-vg/apps-lv
sudo mkfs -t ext4 /dev/webdata-vg/logs-lv
`
15.Create **/var/www/html** directory to store website files

`
sudo mkdir -p /var/www/html
`

16.Create **/home/recovery/logs** to store backup of log data

`
sudo mkdir -p /home/recovery/logs
    `
  Mount **/var/www/html** on **apps-lv** logical volume  
  `
  sudo mount /dev/webdata-vg/apps-lv /var/www/html/
`

18.Use rsync utility to backup all the files in the log directory **/var/log**into **/home/recovery/logs** (This is required before mounting the file system) 

`
sudo rsync -av /var/log/. /home/recovery/logs/
`
19.Mount **/var/log** on **logs-lv** logical volume. (Note that all the existing data on /var/log will be deleted. That is why step 15 above is very
important)

`
sudo mount /dev/webdata-vg/logs-lv /var/log
`
20.Restore log files back into **/var/log** directory
`
sudo rsync -av /home/recovery/logs/. /var/log
`

21.Update `/etc/fstab` file so that the mount configuration will persist after restart of the server.
### UPDATE THE `/ETC/FSTAB` FILE
The UUID of the device will be used to update the */etc/fstab file*;

`
sudo blkid
`
![](/bldidoriginal.PNG)
`
sudo vi /etc/fstab
`

Update /etc/fstab in this format using your own UUID and rememeber to remove the leading and ending quotes.
![](/fstab.PNG)

1.Test the configuration and reload the daemon

`
sudo mount -a
`

`
 sudo systemctl daemon-reload
 `

 Verify your setup by running df -h, output must look like this:
 ![](/verifying%20setup.PNG)

 ## step 2 - Prepare the Database Server 
  Launch a second RedHat EC2 instance that will have a role – ‘DB Server’
  ![](/database.PNG)
  2. 
