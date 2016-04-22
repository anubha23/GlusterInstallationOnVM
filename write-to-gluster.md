After installing and configuing Gluster, one can begin to use the created volume. Below are outlined some steps required in order to start writing to a Gluster volume.

From any one machine in the cluster (in our case, it's 192.168.122.109), you may run these commands to write to the volume and to ensure that replication is happening, which means to ensure that the data is getting duplicated and stored on the other machine (our cluster contains two servers or machines).

1. mkdir mount_point

This command creates a directory called as 'mount_point' under root, because you are logged in to root. This is where we will store our files that we want to save and replicate. The directory can be called anything, we are calling it 'mount_point'. Also, you can choose to create the directory anywhere and not just under "root".

2. mount -t glusterfs 192.168.122.109:/gv0 /root/mount_point/ 

This mounts our directory "mount_point" to the Gluster volume 'gv0' that we created.

3. cd mount_point (Getting into the directory)

4. touch f1 f2 f3 (Creating 3 new files in the directory)

Now, from the other machine (in our case, the other machine in the cluster is 192.168.122.227), open /export/vdb1/brick to view the files. If everything is right, this should display the directory 'mount_point' and within it, the files f1, f2 and f3 that we created in Step 4. 

These files should also be accessible under /export/vdb1/brick in the current machine, which is 192.168.122.109.

Now that the data is on two machines, we can call it a successful replication.

-------------------------------------------------------------------------------------------------------------------------

Writing from a remote client machine (on which gluster-client is installed, this machine does not need to have glusterfs server installed. In most cases, gluster-client will be pre-installed along with the OS.)

In many cases, the servers on which GlusterFS server is installed, are located remotely. In this case, we can follow the below steps to start writing to the volume which is now remote.

mkdir mount_host 
We are creating a directory called "mount_host" inside a directory: home/anubha. Again, the directory can be called anything and you can store it anywhere you like.

ssh-keygen
ssh-copy-id root@192.168.122.109

These ssh commands allow us to access any one of the servers or nodes in the cluster where our gluster volume resides.

If you're doing this using the Virtual Machine manager on your system (virtual instances) i.e. if you have followed the previous tutorial to install and configure GlusterFS, make sure the ssh service is running on both your virtual instances to avoid error. In case such an error is encountered, entering 'systemctl start sshd.service' on the terminals of both your instances should help. You may run the ssh commands again after starting this service.


mount -t glusterfs 192.168.122.109:/gv0 /home/anubha/mount_host/

This command just mounts our directory and the files inside it to our Gluster volume ('gv0') which is located remotely. The IP address here is the IP of the server that we requested access to, using the ssh commands.

The 'mount_host' directory along with any files under it should now be visible under 
192.168.122.109/export/vdb1/brick and under 192.168.122.227/export/vdb1/brick.

Thus, the data is now on both the servers in the remote cluster.

















