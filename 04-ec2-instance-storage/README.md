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

2. Once done, go to `Volumes` > `Snapshots` > select your snapshot > `Actions`.
   
   - Within actions, we can do the following:
     
     - Delete
     
     - Create Volume
     
     - Manage Fast Snapshot Restore
     
     - Create Image
     
     - Copy
     
     - Modify Permissions
     
     - Add/Edit Tags

## AMI Overview

Amazon Machine Image or AMI, is a customization of an EC2 instance. 

- You add your own software, configuration, operating system, monitoring, etc.

- Faster boot/configuration time because all your software is pre-packaged.

- AMI are built for a **specific** region (but can be copied across regions).

You can launch EC2 instances from:

- **A Public AMI**: AWS provided

- **Your own AMI**: you make and maintain them yourself

- **AN AWS Marketplace AMI**: an AMI someone else made (and potentially sells)

**Hands On - creating a custom AMI**

https://www.udemy.com/course/aws-certified-solutions-architect-associate-saa-c02/learn/lecture/26098284#overview

1. Create a normal EC2 instance and install your dependencies and required softwares.

2. Once every thing is done, right-click on the instance and go to `Images and Templates` > `Create an Image`.

3. Now we can create an instance from our AMI. 

## EC2 Instance Store

- EBS volumes are **network drives** with good but "limited" performance. Hence, if you need a high-performance hardware disk, use EC2 Instance Store.

- Better I/O performance

- EC2 Instance Store lose their storage if they're stopped (euphemeral).

- Good for buffer/cache/scratch data/temporary content.

- Risk of data loss if hardware fails.

- Backups and replication are your responsibility.

<span style="color:orange">look into local EC2 Instance Store</span>.

## EBS Volume Types

EBS volumes come in 6 types:

| Volume type     | Description                                                                                                                                                                                                         | Remarks                                                         |
| --------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------- |
| `gp2/gp3 (SSD)` | General purpose SSD volume that balances price and performance for a wide variety of workloads. Can be used for system boot volumes, virtual desktops, development and test environments.                           | <span style="color:limegreen">can be used as boot volume</span> |
| `io1/io2 (SSD)` | Highest performance SSD volume for mission-critical low-latency or high-throughput workloads. Great for databases workloads (sensitive to storage performance and consistency) or apps requiring more than 16k IOPS | <span style="color:limegreen">can be used as boot volume</span> |
| `st1 (HDD)`     | Low cost HDD volume designed for frequently accessed, throughput -intensive workloads. For big data, data warehousing, log processing, etc.                                                                         | <span style="color:crimson">not supported</span>                |
| `sc1 (HDD)`     | Lowest cost HDD volume designed for less frequently accessed workloads. For data that is infrequently accessed                                                                                                      | <span style="color:crimson">not supported</span>                |

EBS volumes are characterized in <span style="color:orange">size | throughput | IOPS (I/O ops per sec)</span>. 

| Volume Type                    | Volume Example      | Remarks                                                                    | Size available   | Throughput       | IOPS                                                                                    |
| ------------------------------ | ------------------- | -------------------------------------------------------------------------- | ---------------- | ---------------- | --------------------------------------------------------------------------------------- |
| General Purpose Instance (SSD) | `gp2`               | cost-effective storage, low latency                                        | 1 Gib - 16 TiB   |                  | size of volume and IOPS are linked 3k to 16k                                            |
| General Purpose Instance (SSD) | `gp3`               | cost-effective storage, low latency                                        | 1 Gib - 16 TiB   | 125 - 1000 MiB/s | 3k - 16k                                                                                |
| Provisioned IOPS SSD           | `io1/io2`           | `io2` is more durable and gives more IOPS per GiB (at same price as `io1`) | 4 GiB - 16 TiB   |                  | PIOPS 64k for Nitro EC2 and 32k for other (can be increased independently from storage) |
| Provisioned IOPS SSD           | `io2 Block Express` | sub-milisecond latency; supports EBS Multi-Attach                          | 4 GiB - 64 TiB   |                  | Max PIOPS = ~256k with IOPS:GiB ratio of 1k:1                                           |
| HDD                            | `st1`               | optimized HDD                                                              | 125 MiB - 16 TiB | ~500 MiB/s       | ~500 IOPS                                                                               |
| HDD                            | `sc1`               | lowest cost                                                                |                  | ~250 MiB/s       | ~250 IOPS                                                                               |

## EBS Multi-Attach

Amazon EBS Multi-Attach enables you to attach a single <span style="color:yellow">Provisioned IOPS SSD (`io1` or `io2`) volume </span> to multiple instances that are in the same AZ. You can attach multiple Multi-Attach enabled volumes to an instance or set of instances. Each instance to which the volume is attached has full read and write permission to the shared volume. 

>  <span style="color:yellow">**Must use a file system that's cluster-aware (not xfs, ex4, etc...)**.</span>
>  <span style="color:limegreen">It makes it easier for you to achieve higher application availability in clustered Linux applications that manage concurrent write operations. </span>

## EBS Encryption

When we create an encrypted EBS volume, we get the following by-default:

1. Data at rest is encrypted inside the volume.

2. All the data in flight moving between the instance and volume is encrypted.

3. All snapshots are encrypted.

4. All volumes created from the snapshot.

> Encryption has a minimal impact on latency. EBS encryption leverages keys from KMS (AES-256).

**Hands-on - encrypt an unencrypted EBS volume**

1. Create an EBS snapshot of the volume.

2. Encrypt the EBS snapshot (using copy) or we can use the snapshot to create volume and encrypt the volume.

3. Create new EBS volume from the snapshot (the volume will also be encrypted.)

4. Now you can attach the encrypted volume to the EC2 instance.

## EFS Overview

Elastic File System, or managed NFS than can be mounted on many EC2.

- It works with EC2 instances in multi-AZ.

- Highly available, scalable, expensive (*3 times gp2*), pay per use.

- 

## EFS vs EBS

# 
