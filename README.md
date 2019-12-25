# aws-solutionsarchitect-associate-exam-notes
AWS Solution Architect Exam Notes

# 1- Multi-Account Management and Organizations


**Consolidated Billing**
http://www.imgim.com/screenshot2019-12-25at165856.png



**Organizational Units Logic**




* You can create two accounts into OUs by default but you can create ticket for AWS Support in order to increase limit of accounts.

**Role Switching Between Accounts**

* When you create account with AWS Organization, OrganizationAccountAccessRoles automatically installed in target account(Member Account). 

**Service Control Policy**
* Service control policies (SCPs) are one type of policy that you can use to manage your organization. SCPs offer central control over the maximum available permissions for all accounts in your organization, allowing you to ensure your accounts stay within your organization’s access control guidelines. SCPs are available only in an organization that has all features enabled. SCPs aren't available if your organization has enabled only the consolidated billing features. 
* The only way to restrict root user is Service Control Policy.
* Master account can not be restricted with Service Control Policy.


# 2- EC2

* You can not see datas in EC2 such as memory utilization in Cloudwatch monitoring for EC2. You can set agent in EC2 and connect it to Cloudwatch.
* Every account has limited EC2 size. You can increase by creating ticket.
* When you create EC2 you can see Health Check in console 2/2 checks. First is hardware second is EC2 itself.
* You will be charged if you shutdown OS instead of changing EC2 state to stop.
* Even if you stop EC2, you will be charged for EBS volume. You need to terminate instance in order to remove EBS.
* In order to get instance IP, AMI ID, instance type, etc… in your code, there is a http endpoint which you can send request and retrieve information.

```

[ec2-user ~]$ curl http://169.254.169.254/latest/meta-data/    
ami-id
ami-launch-index
ami-manifest-path
block-device-mapping/
events/
hostname
iam/
instance-action
instance-id
instance-type
local-hostname
local-ipv4
mac
metrics/
network/
placement/
profile
public-hostname
public-ipv4
public-keys/
reservation-id
security-groups
services/

``` 
```

[ec2-user ~]$ curl http://169.254.169.254/latest/meta-data/ami-id
ami-0abcdef1234567890

```
```
[ec2-user ~]$ curl http://169.254.169.254/latest/meta-data/public-hostname
ec2-203-0-113-25.compute-1.amazonaws.com

```
* Instance EC2 types: https://aws.amazon.com/ec2/instance-types/
* An instance store provides temporary block-level storage for your instance.
* Benefits of instance store is EC2 can directly access it because of hosting on same host computer.
* Trade off instances store is that if host computer fails datas will be lost.
* They are included price in EC2’s except Storage Optimized EC2s.
* Level of OS and instance, if you restart OS or instance instance store is still available. The only way to remove instance store is stopping instance.
* EBS has Solid-State Drives(SSD) and Hard Disk Drives(HDD).
* SSD mainly focused on IOPS, HDD mainly focuses Throughput.
* IOPS is related with input-output rates, Throughput is related with data read-write rates.
* If you need more than 80.000 IOPS you need to use instance store.
* You can change instance type and size whenever you want without data lost.
    
## EBS

* If you take snapshots of EBS, it will be copied to S3 bucket.
* Best way to take a snapshot is stopping instance and take snapshot of it. Because while taking snapshot there can be datas on memory which is not written into disk. So firstly, if you stop the instance memory datas will be written into disk after that takes the snapshot.
* EBS is the service which resides in Availability Zones. In the case of failing AZ, taking snapshot is going to save life because S3 service duplicates itself in different AZs.
* There are two options which are you can provide permission in between another AWS accounts in private or make it public in order to see by all AWS accounts.
* You can create volumes from snapshots. Even if snapshot is taken by volume which is in A AZ, there is an option that it can be created volume in B AZ by snapshot.
* Snapshots can be copied between different regions in order to prevent disasters.
* Snapshot creations and deletions must not be manually, it can be automated by “Data Lifecyle Manager” service.
* Snapshots is taken incrementally. If one of snapshot would be deleted others can relies previous ones.

## SECURITY GROUPS

* Security groups are attached to physically interface which surrounds the EC2 instance.
* Maximum five SG can be attached to interface.
* SG can be in one VPC.
* SG has by default implicit DENY rules.
* You are not allowed to add explicitly DENY rule. For example, you want to provide EC2 instance access for ten people but one of them has infected computer which you do not want to him access. That’s why you need to set smaller IP CIDR block.
* It does not required to refer IPs in every time. You can set other security groups as well as inbound rules.
* SGs are stateful which means there is a data transfer between client and server. Just imagine, when you send a request to website via browser, browser send a request to server and get datas back in order to show them in browser because of outbound traffic rules are opened for 0.0.0.0/0 which means all. As a second example, you connected server with SSH and want to update “yum” package. Please have a look below image for better understanding.

## AMI

* There are two types of instances which are instance stored backed AMI and EBS backed AMI.
* The best practice is to create an AMI is stopping instance firstly because of consistency.
* AMI creates snapshots of EBS volumes, that snapshot includes instance’s root volume, permissions which are public and private. While launching instance with AMI, instance already have block mapping device because of AMI.
* AMIs can be used for various scenarios. If there is a complex architecture, using immutable architecture and in deploying rapidly.
* Privately shared but with whitelisting
* The EC2 Instance type and size for the source instance is not stored in a custom AMI when creating a pre-configured source instance
* Privately sharable images
* Searchable public images

## Bootstrapping

* Biggest advantage of baking AMI is that decreasing time of provisioning of instance.
* It’s not able to do dynamic configurations, whenever you already created configurations in AMI you are not able to change it dynamically.
* The advantage of bootstrapping is dynamically usage, disadvantage of is increasing time of provisioning.

Elastic Network Interfaces (ENI)

* EC2 instances can be configured with or without public IPv4/6 IP addressing.
* An elastic network interface (referred to as a network interface in this documentation) is a logical networking component in a VPC that represents a virtual network card.
* A network interface can include the following attributes:
    1. A primary private IPv4 address from the IPv4 address range of your VPC
    2. One or more secondary private IPv4 addresses from the IPv4 address range of your VPC
    3. One Elastic IP address (IPv4) per private IPv4 address
    4. One public IPv4 address
    5. One or more IPv6 addresses
    6. One or more security groups
    7. A MAC address
    8. A source/destination check flag
    9. A description
* You can create and configure network interfaces in your account and attach them to instances in your VPC.
* You can create a network interface, attach it to an instance, detach it from an instance, and attach it to another instance.
* Every instance in a VPC has a default network interface, called the primary network interface (eth0). You cannot detach a primary network interface from an instance.
* NAT Gateway translate private IP to public IP in order to access internet, also otherwise.
* When you reboot instance, public IP and public DNS do not change, but if you terminate or stop instance public IP changes.
* The default ENI is eth0.
* An ENI is assigned an IP.
* An EC2s instance size delegates the amount of assignable IP’s.
* Public IPs should be assigned to an EC2 for internet communication.
* When the instance is terminated private IP is released.
* ip-x-x-x-x.ec2.internal is the naming convention for a private DNS of EC2 instances.

## Instance Roles

* EC2 Instance roles are IAM roles that can be “assumed” by EC2 using an intermediary called an instance profile.
* An instance role sets permissions to an EC2 application or instance. An instance profile is a container for passing IAM roles information to EC2.
* An instance profile is either created automatically when using the console UI or manually when ısing the CLI.
* The instance profile allows applications on the EC2 instance to access the credentials from the role using the instance metadata.
* Instance roles are temporary credentials which has limited time.
