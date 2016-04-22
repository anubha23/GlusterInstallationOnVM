This document takes the user through steps for installation and configuration of GlusterFS. 
It is intended to give a feel of what GlusterFS is, to a user who may be installing it for the first time.

#####Prerequisites: 
A basic introduction to GlusterFS and virtual machines.

#####Some useful terminologies

######Physical volume:

Physical volume (PV) refers to the physical hard disk. 

You can enter the command "pvs" on the terminal to check how many physical volumes you have and how much space each one holds.


######Logical volume:

Logical volume (LV) refers to a partiion within a physical volume / hard disk. A hard disk may have more than one partition.

You can enter the command "lvs" to see how many logical partitions you have on your disk and how much space each of them occupy.


######Volume Group:

A physical volume and the logical volumes within it together make up a volume group. Entering "vgs" on the terminal gives an overview of the physical volumes you have and the number of logical volumes within each.

---------------------------------------------------------------------------------------------------------------------------------------------------

Now, to install GlusterFs,

You need a minimum of two servers with two physical volumes (or two hard disks) on each server. 

#####Why do we need two servers?

Data is replicated and stored on multiple servers so that if one server fails, the data can be restored from another one. 
A minimum of two servers is thus required to ensure data safety.

#####Why do we need two physical volumes / hard disks on each server?

This is done so that the OS can be installed on one physical volume while the other physical volume can be kept for storing Gluster data. At any given time, if we need to format the hard disk on which the OS resides, the Gluster data remains untouched, since it resides on a different volume / hard disk. Also, in the unfortunate event in which the disk on which the OS is installed gets corrupted, the data is safe.

----------------------------------------------------------------------------------------------------------------------------------------------------

#####Installing GlusterFS using a Virtual Machine Manager

The best way to understanding GlusterFs for a beginner is to set it up using virtual instances. This will need a basic understanding of what a virtual machine is.

Steps:

1. Look for the "Virtual Machine Manager" on your Linux machine.

2. Click on Create new connection.

3. Add "hardware" - storage of some size (in order to have a total of 2 hard disks or in this case, virtual disks).

4. Install your OS on one hard disk / virtual disk.

5. Repeat steps 2 to 4 so that you have 2 connections in all.

With this, you will have met the minimum requirements of two servers and two physical volumes on each server.

------------------------------------------------------------------------------------------------------------------------------------------------------

Run the following commands from root on the terminals of *both* the instances:
(use the command "su" to switch to root)

1. wget -P /etc/yum.repos.d http://download.gluster.org/pub/gluster/glusterfs/LATEST/RHEL/glusterfs-epel.repo

2. yum install glusterfs-server

Gluster should be installed

**Note that though we install GlusterFS on the same virtual disk as the OS, we will be using the other virtual disk to store Gluster data to keep this data separate as mentioned earlier.

------------------------------------------------------------------------------------------------------------------------------------------------------

Next step is to find out the IPs of both the servers so that Gluster can be configured.

Run the command ifconfig. 

The "inet" value of ens3 (the name of the network interface card) should be the unique IP address of your instance / machine / server.

Once this command is run on both the terminals, the IP addresses should be obtained.

-------------------------------------------------------------------------------------------------------------------------------------------------------

#####Now to start configuring GlusterFS, 

* We need to first find out whether Gluster is running.

Enter this command to find out: pgrep glusterd

If Gluster is not running, this command will get the service running:

service glusterd start

Make sure Gluster is running on both the servers before starting to configure.


* We need to link these two servers where Gluster is running. 

For this, we need to run this command from the terminal of any *one* server: 

gluster peer probe <IP address of the other server>

A cluster (network) has thus been formed. 

When this happens, gluster commands that we run on any one of the servers also get executed on the other server. 


* After a cluster is formed, we have to now run some commands in order to store Gluster data on the separate physical volume that we had created (when we added hardware / storage to each instance).

cfdisk /dev/vdb 
This command is used to create a partition on the virtual hard disk that we added in the beginning when we said "Add hardware". A hard disk has to be partitioned and formatted to bring it to a state where it can be used.

How do we know that "vdb" is the name of the virtual disk we added? Simply say "ls /dev"  on the terminal and you will see the list of virtual disks and other devices. In the example that this document followed, the results were vda and vdb.

mkfs.xfs -i size=512 /dev/vdb1 
Run this command to format the partition we created. "vdb1" is the name of the partition.

How do we know this? Entering "ls/dev" again on the terminal will show us a list of devices along with any partitions.
"512" is the filesystem size, a term we aren't delving into right now. It has nothing to do with the size of the partition.

mkdir -p /export/vdb1 && mount /dev/vdb1 /export/vdb1 && mkdir -p /export/vdb1/brick
With the first part of this command, we are creating a new directory "vdb1" under the directory "export" (using 'mkdir -p').It has the same name as the partition 'vdb1' on which it is being mounted using the next command ('mount /dev/vdb1 /export/vdb1'). With the last command (mkdir -p /export/vdb1/brick), we are just creating a directory called "brick" under the directory "vdb1" where we intend to store Gluster data.

echo "/dev/sdb1 /export/sdb1 xfs defaults 0 0"  >> /etc/fstab 
This command adds an entry to fstab. This is done so that in case of system reboots, the system has a record of the mount of directory "vdb1" on the partition "/dev/vdb1".

gluster volume create gv0 replica 2 192.168.122.227:/export/vdb1/brick 192.168.122.109:/export/vdb1/brick 
This creates a distributed gluster volume called as "gv0" (you can name it anything you like). IP addresses of both the servers in the cluster/network have to be specified as shown.

gluster volume info - to check details of our newly created volume.

gluster volume start gv0 - to start the volume so that it becomes open for business.






 






























