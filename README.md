# AWS Sysops Notes

## ELB

### ELB for SysOps

1. static IP only for NLB
2. support SSL for older browers: change LB security group to support a weaker cipher

## Auto Scaling Group

**You can not only define scaling policy based on aws metrics but also custom metric, or even based on a schedule.**

1. Send custom metric from application on EC2 to CloudWatch(PutMetric API)
2. Create CloudWatch alarm to react to high/low values
3. Use CloudWatch alarm as the scaling policy for ASG

**ASG is free.**

### ASG for SysOps

1. High availability - Multi-AZ
2. Healthcheck  
 2.1 EC2 status check - checking VM health
 2.2 ELB Health check - checking application
3. ASG will launch a new ins after terminating an unhealth one
4. ASG won;t reboot unhealth hosts
5. CLI  
 5.1 set-instance-health  
 5.2 terminate-instance-in-auto-scaling-group

### Troubleshooting ASG issues

1. Launching EC2 failure.  
 1.1 ASG has reached the limit set by the DesiredCapacity  
 1.2 Security Group is not existing  
 1.3 key pair not existing  
2. If ASG fails to launch an instance for over 24 hours, it will automatically suspend the process ( administration suspension)  

### CloudWatch Metrics for ASG

## Beanstalk for SysOps

1. Logs go to CloudWatch
2. AWS managed infra
3. deployment modes:  
 3.1 All at once: downtime  
 3.2 Rolling  
 3.3 Rolling with additional batches  
 3.4 Immutable: new ASG, swap all the intances  
 3.5 Blue/Green: swap URL  
4. custom domain: Route53 ALIAS or CNAME on Beanstalk URL
5. no need for patching the runtimes: Node.js, PHP..

## E2 Storage

### EBS & EFS

#### EBS

0. block storage
1. network drive,  might be a bit latency
2. locked to AZ, snapshot to move across AZ
3. Provisioned capacity GBs IOPS
4. types  
 4.1 GP2 SSD: general pupose SSD  
  4.1.1 ec2 boot volume  
  4.1.2 I/O burst  
 4.2 IO1 SSD: highest performance SSD, more than 16,000 IOPS(GP2 limit), best choice for DB storage  
 4.3 STI HDD: low cost HDD for frequently accessed, throughput intensive workloads. bigdata, and etc  
 4.4 SCI HDD: lowest cost HDD for infrequent access  
5. computing MB/s based on IOPS  
 5.1 gp2: MiB/s=(size in GiB)x(IOPS per GiB)x(I/O size in KiB)  

    ```code
    disk size: 100 GiB
    3 IOPS per GiB
    256 KiB Per IO
    throughput in MiB is 100x3x256KiB/s = 75MiB/s
    ```

    __Notes: Limit to 250MiB/s, which means volume >=344 GiB won't increase throughput__  
 5.2 io1: MiB/s=(Provisioned IOPS)x(I/O size in KiB)  
         256KiB/s for each provisioned IOPS  
    __Notes: Limit to a max of 500 MiB/s (at 32k IOPS) and 1000 MiB/s (at 64k IOPS)__

6. You can resize EBS volume, but only increase, and need to repartition to use the incremental storage
7. Snapshot  
 7.1 incremental copy only changed blocks  
 7.2 backup use IOs  
 7.3 recommended to detach volume to do snapshot  
 7.4 can be copied across AZ  
 7.5 can make AMI from snapshot  
 7.6 EBS restored from a snapshot need to be pre-warmed ( using fio or dd command to read entire volume)  
 7.7 snapshot can be automated using Amzon Data Lifecycle Manager  
8. Encryption  
 8.1 Data at rest/data in flight/snapshot/volumes restored from snapshot all encrypted  
 8.2 minimal impact on latency  
 8.3 encrypt key from KMS  
 8.4 copy an unencrypted snapshot allows encryption  
 8.5 To encrypt an unencrypted EBS volume  

    ```code
    create snapshot of the volume
    encrypt EBS volume using copy
    create new ebs volume from the snapshot
    attach the encrypt volume to original instance
    ```

#### EBS vs EC2 Instance Store

1. some instacne don't come with EBS root volume, instead they have "Instance Store" = ephemeral stoarege
2. physically attached to the machine compared to EBS as a network drive
3. good performance, very high IOPS, best for cache, buffer
4. cannot resize, backup by user, lost after termination

#### EBS RAID

1. RAID 0 to increase IOPS
2. RAID 1 to increase fault tolerance

#### EBS CloudWatch

1. VolumeIdelTime | VolumeQueueLength | BurstBalance
2. GP2 5min interval, IO1 1min interval
3. Status: OK Warning Impaired

#### EFS

1. Managed Network File System - NFS, works with EC2 in multi-AZ. 
2. Highly available, scalable, expense 3X gp2, pay per use
3. Content management, web servering, data sharing, wordpress..
4. NFSV4.1
5. security group to control access to EFS
6. EFS scale:  
 6.1 1000s of concurrent NFS clients, 10GB+ throughput  
 6.2 Grow to Petabyte-scale network file system  
7. Performance mode(set at EFS creation time):  
 7.1 general purpose: latency-sensitive use case, web server, CMS, etc..  
 7.2 max IO: higher latency, higher throughput, highly parallel, for big data, media processing..  

## S3 for SysOps

### S3 versioning for SysOps

1. encrypt a file will create a new version to protect data loss
2. delete a file only adds a delete marker on the versioning
3. to delete a bucket, you need to remove all the file versions with the bucket

### S3 MFA Delete

1. To use MFA Delete, enable versioning first
2. you will need MFA to  
 2.1 permanently delete an object version  
 2.2 suspend versioning  
3. You won't need MFA for
 3.1 enabling versioinging  
 3.2 listing deleted versions  
4. Only **bucket owner(root acount)** can enable/disable MFA delete
5. MFA delete can only be enabled using CLI

