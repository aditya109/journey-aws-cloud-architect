Table of Contents

# EC2 Instance Storage

## EBS Overview

An EBS or Elastic Block Store Volume is a network drive you can attach to your instances while they run. 
It allows your instances to persist data, even after their termination.

- <span style="color:orange">They can only be mounted to one instance at a time, but one instance could have multiple EBS volumes attached to it.</span> 

- They bound to a specific AZ.

> Think of them as a *network USB stick*.

### EBS in detail

- It is network drive (*not a physical drive*)
  
  - It uses the network to communicate the instance, which means there might be a bit of latency.
  
  - It can be detached from an EC2 instance and attached to another one quickly.

- It's locked to an AZ. (same as target EC2 volume)
  
  - An EBS Volume is `us-east-1a` cannot be attached to `us-east-1b`.
    
    > For such a usecase, you first need to snapshot it.

- Have a provisioned capacity (size in GBs and IOPS)
  
  - You get billed for all the provisioned capacity.
  
  - You can increase the capacity of the drive over time.

**EBS - Delete on Termination attribute**

This attribute controls the EBS behaviour when an EC2 instance terminates.

While configuring storage options while creating an EC2 instance, you can select `Delete on Termination` to opt for so.

By default, the root EBS volume is deleted (attribute enabled) and any other attached EBS volume is not delete (attribute disabled).

> This can be controlled by AWS Console/CLI.

### **EBS Hands On**

After you attach an Amazon EBS to your instance, it is exposed as a block device. You can format the volume with any file system and then mount it.
After you make the EBS volume avaialble for use, you can access it in the same ways that you can access any other volume. Any data written to this file system is written to the EBS volume and is transparent to applications using the device.

You can take snapshots of your EBS volume for backup purposes or to use as a baseline when you create another volume.

#### To format and mount an EBS volume on Linux

> The device could be attached to the instance with a different device name than you specified in the block device mapping.

Use `lslbk` to view your available disk devices and their mount points to help you determine the correct device name to use. 
<span style="color:yellow">The output of `lslbk` removes the `/dev/` prefix from the full device paths.</span>

For example, for the given below output:

```bash
[ec2-user ~]$ lsblk
NAME          MAJ:MIN RM SIZE RO TYPE MOUNTPOINT
nvme1n1       259:0    0  10G  0 disk            # The attached volume is /dev/nvme1n1, which has no partitions and is not yet mounted.
nvme0n1       259:1    0   8G  0 disk            # The root device is /dev/nvme0n1, which has two partitions named nvme0n1p1 and nvme0n1p128.
-nvme0n1p1    259:2    0   8G  0 part /
-nvme0n1p128  259:3    0   1M  0 part
```

Another example is the output of a T2 instance.

```bash
[ec2-user ~]$ lsblk
NAME    MAJ:MIN RM  SIZE RO TYPE MOUNTPOINT
xvda    202:0    0    8G  0 disk         # The attached volume is /dev/xvdf, which has no partitions and is not yet mounted.
-xvda1  202:1    0    8G  0 part /       # The root device is /dev/xvda, which has one partition named xvda1. 
xvdf    202:80   0   10G  0 disk
```

1. Determine whether there is a file system on the volume.
   
   - New volumes are raw block devices, and you must create a file system before you can mount and use them. 
   
   - Volumes that were created from snapshots likely have a file system on them already;
      <span style="color:orange">**If you create a new file system on top of an existing file system, the operation overwrites your data.**</span>
   
   To determine whether a file system exists on the volume, use one or both of the following methods:
   
   1. Use the `file -s` to get information about a specific device such as its file system type.
      
      - If the output simply shows `data`, as in the following example output, there is no file system on the device.
        
        ```bash
        [ec2-user ~]$ sudo file -s /dev/xvdf
        /dev/xvdf: data
        ```
      
      - If the device has a file system, the command shows information about the file system type.
        
        ```bash
        [ec2-user ~]$ sudo file -s /dev/xvda1             # the selected device has XFS file system
        /dev/xvda1: SGI XFS filesystem data (blksz 4096, inosz 512, v2 dirs)
        ```
   
   2. Use the `lsblk -f` to get information about all of the devices attached to the instance.
      
      ```bash
      [ec2-user ~]$ sudo lsblk -f
      NAME               FSTYPE    LABEL    UUID                                       MOUNTPOINT
      nvme1n1               xfs                7f939f28-6dcc-4315-8c42-6806080b94dd                              
      nvme0n1                                                                                          
      ├─nvme0n1p1           xfs        /        90e29211-2de8-4967-b0fb-16f51a6e464c    /
      └─nvme0n1p128                                                                             # does not have file system
      nvme2n1                                                                                   # does not have file system 
      ```

2. If you have a file system on the deivuce in the previous step, this step can be skipped. If you have empty volume, use the `mkfs -t` command to create a file system on the volume.
   
   ```bash
   [ec2-user ~]$ sudo mkfs -t xfs /dev/xvdf
   # If you get an error that mkfs.xfs is not found, install the XFS tools
   [ec2-user ~]$ sudo yum install xfsprogs
   ```

3. Use the `mkdir` command to create a mount point directory for the volume. The mount point is where the volume is located in the file system tree and where you read and write files to after you mount the volume.
   
   ```bash
   #  The following example creates a directory named /data.
   [ec2-user ~]$ sudo mkdir /data
   ```

4. Use the following command to mount the volume at the directory you created tin the previous step.
   
   ```bash
   [ec2-user ~]$ sudo mount /dev/xvdf /data
   ```

5. Review the file permisssions of your new volume mount to make sure that your users and applications can write to the volume.

6. The mount point is not automatically preserved after rebooting your instance.

#### Automatically mount an attached volume after reboot

https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/ebs-using-volumes.html

To mount an attached EBS volume on every system reboot, add an entry for the device to the `/etc/fstab` file.

You can use the device name, such as `/dev/xvdf`, in `/etc/fstab`, but it is recommeneded using the device's 128-bit UUID instead.

> Device names can change, but the UUID persists throughout the life of the partition.

By using the UUID, you reduce the chances that the system becomes unbootable after a hardware reconfiguration.

1. *(Optional)* Create a backup of your `/etc/fstab` file that you can use if you accidentally destroy or delete this file while editing it.
   
   ```bash
   sudo cp /etc/fstab /etc/fstab.orig
   ```

2. Use `blkid` command to find the UUID of the device and note it down, as it would be used on other places as well.
   
   ```bash
   sudo blkid
   # or for Ubuntu 18.04
   sudo blkid -o +UUID
   ```

3. Edit `/etc/fstab` and add the following entry to mount the device at the specified mount point. For example, we want to mount `<MY_MOUNT_UUID>` at mount `/data` and we use the `xfs` file system. We also use the `defaults` and `nofail` flags.
   We specify `0` to prevent the file system from being dumped, and we specify `2` to indicate that it is a non-root device.
   
   ```bash
   UUID=<MY_MOUNT_UUID> /data xfs defaults,nofail 0 2
   ```
   
   > `nofail` mount option enables the instance to boot even if there are errors  mounting the volume. For Debian derivatives, including Ubuntu 16.04, must also add the `nobootwait` mount option.

4. To verify whether your entry works, run the following commands to unmount the device and then mount all the file systems in `/etc/fstab`. If there are no errors, the `/etc/fstab` file is OK and your file system will automatically mounted after it it rebooted.
   
   ```bash
   sudo umount /data && sudo mount -a
   # if you receive an error message, address the errors in the file
   ```
   
   > Errors in the `/etc/fstab` file can render a system unbootable. Do not shutdown a system that has errors in the `/etc/fstab` file.
   > 
   > If all fails, you can restore the original `fstab.orig` file to the current one.
   > 
   > ```bash
   > sudo mv /etc/fstab.orig /etc/fstab
   > ```

## EBS Snapshots Overview

- We can make a backup (snapshot) of your EBS volume at a point in time.

- We do not need to necessarily detach volume to do snapshot, but recommended.

- We can copy snapshots across AZ or region. 

![](https://raw.githubusercontent.com/aditya109/journey-aws-cloud-architect/main/04-ec2-instance-storage/assets/snapshots.svg)

**Hands-On**

1. Select the EBS volume and click on `Actions` > `Create Snapshot`.

2. 







## AMI Overview

## EC2 Instance Store

## EBS Volume Types

## EBS Multi-Attach

Amazon EBS Multi-Attach enables you to attach a single Provisioned IOPS SSD (`io1` or `io2`) volume to multiple instances that are in the same AZ. You can attach multiple Multi-Attach enabled volumes to an instance or set of instances. Each instance to which the volume is attached has full read and write permission to the shared volume. 

<span style="color:limegreen">It makes it easier for you to achieve higher application availability in clustered Linux applications that manage concurrent write operations. </span>

## EBS Encryption

## EFS Overview

## EFS vs EBS

# 
