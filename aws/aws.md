# AWS Associate Architect traning notes

Notes and cheat sheets made while following [this udemy training](https://www.udemy.com/course/aws-certified-solutions-architect-associate) preparing for the AWS Associate Architect Certification exam. I recommend this training for beginners: it a serie of videos covering both theory and practice. These notes below don't cover the practical part, only the theory. They might give you an overview of what's covered but it's more a reminder of what's in the videos.

**RECOMMANDATION**: you should read AWS FAQ for RDS, VPC, EC2, NACLS vs. Security Groups, etc. 

## Regions/Availability zones/edge locations

**MISSING PART ABOUT REGIONS/AVAILABILITY ZONE**

**The availabilty zone name are different from one account to the other**. This allow AWS to redistribute the resources more evenly (naturally people tend to pick more ften az-1a than az-1b).

## IAM (Identity and Access Management)

When you create an account on AWS, it's a **root account**. The **root account** is a super admin account, it have all right on everything.

So you need to enable **MFA** (multi-factor auth), for instance by installing Google Authenticator on your phone. Make sure you have a **backup** for your **MFA**, because if you don't, when you lose your phone, you lose your account, forever.

This root account can create other account with restricted access. These accounts can login through a special url, something like https://myalias.signin.aws.amazon.com/console

AWS has a very powerfull access control system.
You need to understand four concept : **users**, **groups**, **roles** et **policies**

- **Policies**: are just permissions basically. AWS has a long list of are-made policies like **AdministratorAccess** which give you all right on everything, or **AmazonS3ReadOnlyAccess**. But you can also make your own **policies**. For instance, **AmazonS3ReadOnlyAccess** allow you to access all S3 buckets. But might want create a policy to access only one specific S3 bucket.
  Here is the structure of a policy (**AdministratorAccess**):

  ```json
  {
    "Version": "2012-10-17",
    "Statement": [
      {
        "Effect": "Allow",
        "Action": "*",
        "Resource": "*"
      }
    ]
  }
  ```

  So policies are a list of "Allow" or "Forbid" to do "action" on "ressource".

- **Roles**: are sets of policies. You can assign a role to an user, **but also, to an AWS ressource, like a EC2 instance**. You can for instance create a role so that myEc2Instance, and only myEc2Instance can access myS3Bucket.
  From AWS IAM FAQ:

  > **Q: How do I assume an IAM role?**
  > You assume an IAM role by calling the AWS Security Token Service (STS) AssumeRole APIs (in other words, AssumeRole, AssumeRoleWithWebIdentity, and AssumeRoleWithSAML). These APIs return a set of temporary security credentials that applications can then use to sign requests to AWS service APIs.

- **Groups**: they contain users. You can assign them policies directly, and every user in the group will have these permissions.

- **Users**: an account created by the root account. You can put the user in a group, or directly assign policies to the user. **They have no permissions when first created**

When you create an user, if you check "programmatic access", it will generate an **access Id** and a **secret key**. YOU NEED TO SAVE THE SECRET KEY RIGHT AWAY, OR IT WILL BE LOST FOREVER.

Five things you can after creating your root account, in IAM:

- Create an alias for the signin url. By default it's something like (https://123456789012.signin.aws.amazon.com/console), you can make it easier to remember (like https://myalias.signin.aws.amazon.com/console)
- Enable MFA for the root account
- Create at least one user and avoid using the root account
- Create at least a group
- Create a IAM password poilcy (how often should user change their password, minimum length, etc...)

**RECOMMANDATION**: use your root account as little as possible. Create a admin IAM user and use it instead.

Various facts

>  Using SAML (Security Assertion Markup Language 2.0), you can give your  federated users single sign-on (SSO) access to the AWS Management  Console.
>
> Power User Access allows access to everything but IAM

## Using AWS CLI

On macos, you can install aws cli with homebrew

```bash
brew install awscli
```

If you are using **zsh** and **oh-my-zsh** you can get autocompletion for the aws cli by enabling the aws plugin in your .zshrc file:

```bash
plugins=(git ... aws)
```

Then you can login to your aws account using:

```bash
aws configure
AWS Access Key ID [None]: XXXXXXXXXXXXXXXXXXXX
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX
Default region name [None]: eu-west-3
Default output format [None]: json
```

Then you can use aws cli to query IAM groups for instance:

```bash
aws iam list-groups
{
    "Groups": [
        {
            "Path": "/",
            "GroupName": "Developpers",
            "GroupId": "XXXXXXXXXXXXXXXXXXXX",
            "Arn": "arn:aws:iam::333333333333:group/Developpers",
            "CreateDate": "2021-03-19T18:52:43+00:00"
        }
    ]
}
```



## Set an billing alarm with cloudwatch

You can create a billing almart so that you receive an email as soon as your monthly bill go beyond a certain amount. It's really helpfull if you forgot you had that expensive EC2 instance running, or if you underestimated the pricing of an AWS service.

For that search for **cloudwatch**, then in the left menu **alarms > billing** and then **create alarm**. Enter an amount at the bottom of  the page, check "create new topic (SNS)" in the next page, enter your email, on the next page enter name and description, then on the last page click create alarm and you're good. The alarm my be in "**insufficient data**" category. If you just created your account and are in free tier, **that's normal.**

You can create folders in your S3 bucket.

## S3 - Simple Storage Service

**S3 IS A BIG PART OF THE EXAM**
S3 allow you to store an unlimited number of files. You can create:

- A **bucket** : it's basically a folder.
  The **name** of your **bucket** must be **unique globally**, because it will be in the url of your bucket: https://mybucket.eu-west-3.amazonaws.com/.
- An **object**: it's just a file.
  **Key** is the name of the file, **value** is the content. You also have a **versionId**, (you can have multiple versions of one file). You can also store **metadata** with the file. You also have **subresources** attached to the file, like access control lists and torrent. The file can be 0 bytes up to 5TB. It's always inside a bucket.

If you have the right permission, you can upload a file with HTTP PUT and it will return **status 200**.

**Data consistency**: 

- **Read after write consistency for PUT**: when you put a new object in a bucket, you can read it immediatly
- **Eventual consistency for DELETE and overwrite**: If you overwrite or delete an object, it might take a while before it takes effect (you might read the previous version for a little while).

**Availability**: S3 aim for **99.99%** availability and amazon **guarantee 99.9% availability** and guarantee 99.99999999999% (99. 11 x 9) **durability** (basicaly the file will never be lost or corrupted).



- **Lifecycle management**: you can automaticaly move files when they are X days old.
- **Versionning**: You can have multiple version of the same file
- **Encryption**: you can encrypte the files

- **Tiered storage**: S3 has various Storage tier, with various pricing and use cases.
- **You can enable mandatory MFA for file delete** (important for the exam)
- Control S3 access using **Access Control Lists** and **Bucket Policies**

### S3 tiers

- **S3 standard**: 99.99% availability + 9.11x9% durability. Stored redundantly in multiple facilities (can sustain the loss of 2 at the same time). The most expensive option
- **S3 IA - Infrequent Accessed**: cheaper than S3 standard, instant access, but charge retriaval fee
- **S3 One zone IA**: same thing but even cheaper. It's only in one Availability zone so the availability is lower.
- **S3 - intelligent tiering**: Move files to the most appropriate tier automatically, without performance impact or operational overhead.
- **S3 glacier**: durable, for data archiving, every reliable, and very cheap. Take minutes to hours to retrieve data
- **S3 glacier deep archive**: Even cheaper, around 1$ per TB. Can take 12 hours to retrieve data.

![image-20210320213903832](./images/S3_tiers.png)



You are charged depending how:

- How much data you store
- How many request (reads/writes) you make
- The tier you are using for storage
- How much data is transfered
- Transfer acceleration
- Cross region replication pricing

**Transfer acceleration**: Ideal for uploads on long distance. When an user from Japan upload a file, instead of sending it directly to your S3 bucket in France, the file will be uploaded to a edge server close to the user. Then the file will be transfered from the edge server to your bucket. This is faster because the connection between aws edge server and the S3 bucket is super fast.

**BEFORE EXAM READ S3 FAQs. IT COMES UP A LOT.**

Three ways to restrict access:

- Bucket policies
- Object policies
- IAM policies to User & groups

**By default, 100 buckets max per account**

### Encryption

- Encryption in transit = HTTPS (SSL/TLS)
- Encryption at rest (server side)
  - S3 Manager Keys - **SSE-S3** (amazon manage the keys for you)
  - AWS Key Management Service, Managed keys : **SSE-KMS**
  - Server side encryption with customer provided keys - SSE-C
- Client side encryption

### Versionning

If you enable versionning, make the first version of a file public and then upload a new version of that file, the new version WILL NOT be public. You have to make every version public.

When you delete a document, when versionning is enabled, you don't delete the versionned objects. It just has a **"delete" marker**, you cannot access it through the url, and you don't see the file listed in the bucket, unless you enable "list version". But the data is still here. You can restore deleted versionned objects.

When you disable versionning in a bucket, it will not create new versions for objects, but it won't delete existing versions of objects.

You can permanently delete versions. You can enable **MFA for delete**

### Lifecycle actions

You can :

- Transfer current version to other Storage Tier after X days
- Transfer non-current version to other Storage Tier after Y days
- Delete permanently previous object version after X days

### S3 Object Lock and Glacier Vault Lock

S3 lock can be applied to individual Object or to the whole bucket

Write once, read many (WORM) : you can prevent objects to be deleted or modified for a period of time or indefinitely

**Governance mode**: WORM for most users but exception if you have special persmission

**Compliance mode**: no one can delete or overwrite, including root user

**Rentention period:** The period of time during which the object is protected

The retention period is stored in object metadata.

**Legal hold**: WORM with no rentention period. In effect until removed.

**Glacier Vault Lock**: similar to S3 Object lock. You create a Vault lock policy and then you can lock that policy: you can't change to policy after that.



### S3 performance

First bytes after 100 to 200ms

**prefix**: mybucketname/folder1/subfolder1/myfile.jpg > /folder1/subfolder1

Important because max **5500 GET** request/s and **3500 PUT/COPY/POST/DELETE** **per prefix**

All files in 1 prefix => you can serve at most 5500 files per second.
Files split into 2 prefixes => you can serve 11 000 files per second.
                         4 prefixes => you can serve 22 000 files per second.

**You should split your files in different prefixes**

**Performance limitations with S3 SSE-KMS**: limit number of read/write **per Region** (**5500, 10 000 or 30 000** depending on region). You cannot request quota increase.

**Multipart upload**: recommanded for files over 100MB, required for over 5G. Parallelize uploads (**increase efficiency**). The file is split 

**S3 Byte-Range Fetches**: same thing but for download. (Split big file in small part, and download parts in parallel). Can be used to read file header only too.



### S3 Select and Glacier select

**S3 Select**: you can extract data from an S3 Object using an SQL expressions.
You get data by rows or coulons. Save money, increase speed.

For example, If you have a gzipped CSV file on S3 and want to retrieve a single line, instead of downloading, unzipping and parsing the whole file, you can use **S3 Select** to get only the data you need with an **SQL request**.

**Glacier Select:** similar to S3 select

### Cross account S3 buckets access

You can access s3 buckets from another AWS account (*I mean **another root account NOT another IAM user***) using **AWS organizations**.

**POPULAR AND IMPORTANT EXAM TOPIC **(need to know the three way you can do this)

![Aws organizations diagram](./images/aws_organizations.png)

**AWS Organization**: A way to connect and organize several **root accounts** (not IAM users). You have a **master account**, which can be used to restrict other child root accounts. You can recreate your company structure with AWS accounts. You can create a hierachy using OUs (Organization Units). You can apply a policy to it. All the children accounts and children OU will follow this policy (the effect is recursive).

**Consolidated billing**: Aws the more you use, the less you Pay, so if you use the Organization's **master account** for billing, you pay less. **You get volume pricing discount** + it's easier to keep track of the costs. Consolidated billing = one bill per AWS account.

To create an organization you go to **AWS Organizations** in  **Management & Governance**

![Management & governance](./images/management_and_organizations.png)

Then you create an organization and you invite other accounts to join.

In the master account, you can have a tree view of your organization.

You can then apply **Service Control Policies** to OU and accounts: You can for instance disable everything but S3 for an account or OU.

**WHY USE AWS ORGANIZATIONS INSTEAD OF IAM USERS AND GROUPS ?**

- You can be a company with several unrelated projects and you want to isolate their AWS resources (for security and organization reason) but still want to benefit from volume pricing discount.
- You might want to seperate you dev/staging and production environnement so that your new intern doesn't screw your production env. That's the most typical use case.
- You might have several teams in your organization and you don't want them to use the same root account.

**RECOMMANDATIONS**:

- Enable MFA for all root accounts and set a strong password
- Do not deploy resources in the billing account
- Use Service Control Policies to disable unused services (maybe you don't want the finance team to be able to run EC2 instances)

### 3 ways to share buckets

- **Bucket policies** and IAM (for programmatic access only)
- **Bucket ACLs** (access control list) & IAM for individual objects (for programmatic access only)
- **Cross-accounts IAM roles**. Programmatic AND Console access

#### Cross-accounts IAM roles

You **create a role** (like CrossAccountS3AccessRole) in your **master account** and you set another account of your organization as the **trusted entity**. You give that role the S3 full access permissions. **This process will generate a link to switch role, you will need it.**

Then **in your other account** you create a IAM user, and login as that user. You **paste the link** mentioned above, and click "switch role". Then you have only have the permissions to use the S3 service.

### Cross region replication

If your app is international, and you have a bucket in France, your users in Japan will have harder time to access the file (slow transfer (unless transfer acceleration enabled) and latency). So you might want to copy all the files in your bucket in France to a bucket in Japan automatically and keep them in sync.

For that you need to create a bucket in Tokyo, then in your original bucket in France, you need to configure **Replication rules** in **Management**. You give it a name and chose your Japan bucket as a target. You must select a role. **You can choose to apply this to only a spefic prefix** (folder). Your target bucket can be in another account. You can also change the Storage class (tier) while transfering files. You can enable Replication Time Control (RTC) which guarantee that 99.99% of your objects will be replicated under 15minutes. You can explore the different options in the console, it's pretty clear.

**You need to enable versionning in your source and destination buckets for replication to work**

⚠️ THE REPLICATION RULE ONLY COPY OBJECTS AND VERSIONS ADDED **AFTER** YOU CREATED THE RULE.
⚠️ **IT ALSO DOES NOT REPLICATE PERMISSIONS (ACL)**
⚠️ **DELETING VERSIONS AND DELETE MARKERS WILL NOT BE REPLICATED**

### Transfer Acceleration

Your users upload files to an edge location instead of the S3 bucket directly. The edge location is used as a kind of proxy. Connection between user and edge location is good because close. The connection between the edge location and the S3 bucket is VERY GOOD because it uses AWS internal and highly optimised network (**backbone network**). This is much more efficient than upload the file directly to the bucket if your user is far from it.

If you use it keep in mind that you pay an extra:

> S3 Transfer Acceleration pricing is in addition to Data Transfer pricing.
>
> Accelerated by AWS Edge Locations in the United States, Europe, and Japan 	$0.04 per GB

You can use this tool to get an idea of the speed improvement:
[accelerate speed comparison](https://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html)

Can be slower sometimes (for instance France to Europe or US East to France), but France to Tokyo is 30% faster for me.

### AWS DataSync

Allow you to transfer large amounts of data to/from your AWS S3 bucket from outside AWS (like on-premise).

- Used to move large amounts of data (from on-premise for instance)
- Used with NFS and SMB 
- Replication can be done every hour, day or week
- You need to install a DataSync agent
- Can be used to replicate **EFS to EFS** (Elastic File System)

> Amazon EFS is designed to provide massively parallel shared access to thousands of [Amazon EC2](https://aws.amazon.com/ec2/) instances, and AWS containers and serverless compute services including [Amazon Elastic Container Service](https://aws.amazon.com/ecs/) (ECS), [Amazon Elastic Kubernetes Service](https://aws.amazon.com/eks/) (EKS), [AWS Fargate](https://aws.amazon.com/fargate/), and [AWS Lambda](https://aws.amazon.com/lambda/), enabling your applications to achieve high levels of aggregate throughput and IOPS with consistent low latencies.

### CloudFront

Cloudfront is a **CDN** (Cloud Delivery Network), hundreds of servers across the world that store cached versions of pages/files from the **Origin** server (S3 bucket, EC2 instance, etc...), in order to speed up page load, file transfer. These servers are the **Edge Locations**. This set of Edge Locations, the CDN, is called **Distribution**.

Edge location can be used as proxy (like we saw in transfer acceleration), so it can be used to **serve** static files, but also for **dynamic content and streaming !**

**Two types of distribution**:

- Web distribution, for websites
- RTMP - used for Media Streaming

**Edge locations are NOT read only. You can write to them too.**

Object cached for the TTL (Time To Live). You can clear cache manually or programmatically, but you will be charged for that. This is not free ! You are charged a fee.

It take time to create or delete a cloudfront distribution (like 30min - 1hour)

You can invalidate object with invalidation path. like `/images/*`

you can enable: force use of signed urls/cookies

#### Signed Urls vs signed cookies

**Signed url**: url containing authorisation/access control metadata, signed digitale. You can generate urls with an expiration date for instance. Perfect to restrict access to one specific file.

**Signed cookies**: Same thing but in a cookies. If you want to restrict access to several files.

When we create a signe dUrl or signed cookie, we attach a policy, which can include:

- URL expiration
- IP ranges
- Trusted signers (which Aws accounts can create signed URLS).

**OAI**: **Origin Access Identity**

You generate signed url/cookies using AWS SDK

**S3 signed URLs** different from cloudfront signed URL. URL signed as **IAM user**. Limited lifetime.

EC2 origin => cloudfront sign url

S3 bucket file => maybe S3 signed url

### Snowball

![snowball](https://docs.aws.amazon.com/fr_fr/snowball/latest/developer-guide/images/Snowball-Edge-Image.png)

**Snowball** is ... **a suitcase**. Yeah really. A suitcase containing disks full of data. You have a 50TB version and 80TB. It can be used to transfer petabytes of data. The data is encrypted with 256-bit encryption. Can be a fith of high-speed internet data transfer costs.

**AWS Snowball Edge**: 100TB data per snowball edge. Storage and **compute capabilities**. Perfect for remote or offline locations. (Example: in aircraft tests). It can run lambda functions. You can create clusters using this.

**AWS Snowmobile:** It's a freaking shipping container pulled by a truck. 100PB per snowmobile. For Exabyte-scale data transfer. I don't think I'm ever going to need this one...

**You can use all this to Import/export to S3.**

### Storage Gateway

Connect on-premise network with AWS cloud. It a safe way to extend your on-premise data center with S3 storage capabilities.

The AWS Storage Gateway's software appliance is a WM you setup in your data center and use as a Gateway for data transfer to AWS. You then need to use AWS console to configure it.

**File Gateway:** to store files in S3.

**Volume Gateway**: a copy of a volume (disk) in one big shunt

NOT really interested by this so I will come back to this if I really want to take the exam.

### Anthena and Macie

**Anthena**: Analyse and query data stored on S3 with SQL. No need to Extract/Transform/Load. It's **serverless**. Interactive query service. Mostly used to analyse logs.

Can be used to query logs. Can generate buisness report, by querying billing data. Can run queries on click-stream data.

**PII**: Personnally Identifiable Information. Critical information: Identity card, credit card number, etc...

**Macie**: A security service which automatically detect PII and other sensitive data stored on your S3 bucket, can give you alerts and stuff. It uses AI. Includes dashboards, reports, alerting. Great for PCI-CSS compliance and preventing ID theft. Can be used to detect suspicious activity together with CloudTrail logs.

## EC2

**EC2: Elastic Compute Cloud**

You can get instances (VMs) up and running in a few minutes.

You have three type of pricing:

- **On demand**: you pay for what you use
- **Reserved**: you reserve instances for 1 or 3 years terms. Cheaper (up to 75% off). You get discounts if you pay upfront. You **cannot move a reserved instance to a different Region** (but you can change the AZ).
- **Spot**: The instances price change depending on the demand. It can get really cheap if demand is low. You bid for your instance capacity. Usefull if you need very cheap compute resource with low constraint or if you have an urgent need for resources.
- **Dedicated Hosts**: Dedicated physical instances. If you need to run special software, or have legal constraints.

**For spot**: If Spot instance terminated by Amazon EC2, not charged for partial hour. If you terminate it yourself, you pay for it.

**Termination protection**: prevent you from terminating a EC2 instance by mistake. Turned off by default.

When you create an EC2 instance you get a **root volume**, and you can add **additional volumes**.
You can now encrypt the root volume of your EC2 instance.  You can choose several types of disk (SSD general purpose / Provisioned IOPS / HDD / etc...). By default your root volume is deleted when you terminate the instance.

You can encrypt a root volume anytime after instance creation.

**MAX 20 instances per region by default.**

You can stop your EC2 instance without terminating it, you will not be charged for it will it's stoped. (You pay for the root volume however). You can speed up the start time by using hibernation, but you need to encrypt the disk in order to do that.

### Security group

A security group is basically a firewall, and your EC2 instances are behind the firewall.
You have:

- **Inbound rules**: Allow you to filter the traffic from internet to your inctances.
  For instance: TCP 80 0.0.0.0/0  mean allow all TCP traffic from any IP going to port 80
  (0.0.0.0/0: allow all IPv4 addresses, ::/0 allow all IPv6 addresses)
- **Outbound rules**: Allow you to filter the traffic going out from your instances, to internet.

**By default all inbound traffic is blocked** (except maybo port 22).
**By default all outbound traffic is allowed**

Inbound/outbound rules work as a whitelist. You cannot blacklist a port.

**You cannot block a specific IP using security group**. For that you use Network Access Control List (**NACL**).

If you create an inbound rules, outbound traffic will automatically be allowed for the same kind of traffic (same port, etc...). This behavior is called stateful as opposed to stateless. **Security groups are stateful**.

**Security group changes take effect immediatly !**

You can add **multiple security groups to an instance**, it will combine the whitelists and allow everything take is in at least one security group.

You can have any number of instances in a security group.

### EBS

**Elastic Block Store**: virtual hard disk in the cloud. Mostly used as disks for EC2 instances. Data on it automatically replicated in different availability zones.

- **General Purpose (SSD)**: regular SSD. Default for root. Perfect to host programs / OS. You use this in most cases. (16 000 IOPS max / volume)

- **Provisioned IOPS (SSD)**: If you need really fast, low latency IO. For datables. (64 000 IOPS max / volume)

- **Throughput Optimised Hard Disk Drive**: For big data & data warehouses. Regular magnetic hard disk (not ssd) (500 OIPS max / volume)

- **Cold HDD**: For file servers (250 IOPS max / volume)

- **EBS Magnetic**: For infrequently accessed data (40/200 IOPS max / volume)

Detailed comparison [in aws doc](https://aws.amazon.com/ebs/features/).

The **EBS volume** is in **the same availability zone** as the **instance** it's attached to (make sense).

Root EBS volume is deleted when the instance is terminated. By default, Additional volumes are not deleted when the instance is terminated. 

Once created **you can extend** the size of an EBS volume and you can **change its type** (cold HDD can become Provisioned IOPS SSD). But it **takes time and affect performance**.

>  As of Feb 2020 you can attach certain types of EBS volumes to multiple  EC2 instances.  https://aws.amazon.com/blogs/aws/new-multi-attach-for-provisioned-iops-io1-amazon-ebs-volumes/

#### Snapshot

You can create a **snapshot** of the disk (**a copy**). It may takes some time to create (especially if it's the first snapshot). **It's stored on S3.**
**Snapshots are incremental**: if you create 2 snapshots, the second while only contain the changes since the previous one (the changed blocks).

Once you have a snapshot ready, you can create an **image** from it, also called **AMI** (Amazon Machine Image). Select Hardware-assisted virtualisation (not sure why).

You can use this image to **create a EC2 instance in another availability** zone.

**Migrate EBS/EC2 instance** : snapshot > image > recreate instance from image in another availability zone

You can copy your image to another **Region** too.

**It's best to STOP the instance before taking a snapshot of the EBS. (it's possible however)**

### AMI types (EBS vs Instance Store)

Two storages types for the root device volume:

- **EBS Backend volumes**

- **Instance Store (EPHEMERAL STORAGE)** : created from a template in S3. **It restricts what type of instance you can use !** You cannot use t3 instances for example. (It looks like m3 is the minimum for now but it can change). **Once you terminate an EC2 instance using this, all data is lost** ! And you cannot stop the instance, only terminate it. If the host fails, you lose your data. You can reboot without losing data however.

  > Instance store is ideal for temporary storage of information that changes frequently,                                    such as buffers, caches, scratch data, and other temporary content, or for data that is repli

**After** you **created** an EC2 instance with instance Store, you can attach additional EBS volumes, **but not additional instance store**.

### ENI vs ENA vs EFA

**ENI**: Elastic Network Interface (virtual network card basically)

**EN**: Enhanced Networking

**EFA**: Elasticc Fabric Adapter

You get one ENI by default with your EC2 instance. It comes with a public address and potentionally mutliple private IPs on your VPC.

**Enhanced Networking** well enhance your network. You get better I/O network performance. It uses **single root I/O virtualizaition (SR-IOV)** (no idea what it means). Higher PPS (packet per second). No additional charge for **EN** but your instance need to support it.

Two way to enable **EN**:

- **ENA**: Elastic Network Adapter (up to 100Gbps for supported instances)
- Intel 82599 **Virtual Function (VF)** up to **10 Gbps** used on older instances

In most scenario you probably want to choose ENA over VF or multiple ENI.

**Elasticc Fabric Adapter** is a network device you can attach to your EC2 instance. Used for High Performance Computing (HPC) and machine learning applications. Lower and more consistent latency, higher throughput than TCP transport alternatives. **Can use OS-bypass (bypass the linux kernel so linux only)** -> lot faster, lot lower latency

The **take-away**:

- Basic usage => **ENI** (default)
- reliable high throughput between 10 and 100gbps => **ENA**
- HPC/ML/OS-bypass => **EFA**

### Encrypted root Device volumes & snapshots

**If you want to encrypt an unencrypted root device** you need to create a snapshot. Then **actions > copy** snapshot and check "**Encrypt this snapshot**". Then you create an AMI from this snapshot. **Finally you create an encrypted instance from this image.**

- **Snapshots of encrypted volume are encrypted automatically.**
- Volumes restored from encrypted snapshots => encrypted
- Cannot share encrypted snapshots
- Snapshots can be shared with other AWS accounts or made public
- You can now encrypt the root volume directly.

### Spot Instances / spot fleets

When you need some computing done for as cheap as possible, without having constraints on when it should be done. Must keep in mind that it might get interupted. For instance if you want to compute decimals of PI, you might want to use spot instances during night-time in your region because activity is low, instance availability is high so price low.

**Discount up to 90%.** Your app need to be fault-tolerant, stateless.

**Your app must be flexible** and ready to stop automatically in **two minute** after you receive notice.

**Not for critical applications like back-ends or databases**

You decide on your maximum Spot price. Your instance will run as long as the spot price is below that price.

Spot block: you can keep your instance running even if price goes above max price for 1h to 6h

**Use cases**: BIg data and analystic, containerized workloads, CI/CD and testing, web services, Image and media rendering, HPC

![spot request](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/images/spot_lifecycle.png)

![spot lifecycle](https://docs.aws.amazon.com/AWSEC2/latest/UserGuide/images/spot_request_states.png)

**Spot fleet**: a mix of spot instances and on-demand instances. You set a number of instance you need, aws uses as many spot instances as possible, and complete with on-demande instances. You can also set a maximum total price.

1. You can setup pools in which you define EC2 type, OS, availability zone
2. You can have multiple pools. AWS will make the best choice depending on your strategy.
3.  Stop instances until, 

**Several strategies**: capacityOptimized, lowestPrice, diversified (distribute instances between all pools), InstancePoolsToUseCount

### EC2 hibernate

**Hibernation** save the content of the RAM to your EBS root volume. This way you can restore your EC2 instance to the state it was before to stopped it. **It's much faster to boot an EC2 that is in hibernation.**

The **root volume must be encrypted** to enable this.

The resumed **instances keep their instance ID**.

Instance **RAM** must be **less than 150GB**

**You cannot hibernate an instance for more than 60 days**.

For on-demande and reserved instance

Useful for long running processes or services that take time to initialize.

**When you create the instance** you need to check **"Enable hibernation as an additional stop behavior"** in "**Step 3**: configure instance details". We need to encrypt the root volume and make sure their is enough free space in it to store the RAM.

### CloudWatch

**Allow you to monitor AWS resources as well as your apps.**

**Cloudwatch** monitor **performance**. It can monitor EC2 instances, Autoscaling groups, Elastic Load Balancers, Route53 health checks. It can monitor EBS volumes, CloudFront, Storage Gateways

For instances: CPU, Network, Disk, Status Check

**CloudTrail** increase visibility by recording AWS Management Console actions and API calls. You can find out who id what and from where.

**CloudWatch** monitor **performance**. **CloudTrail** monitor **API calls** and actions through the AWS console. It's about auditing.

CloudWatch monitor events **every 5 minutes by default**, but you can get more **detailed monitoring** with **1 minute intervals** (you need to check "detailed monitoring" when you create the instance, but it's more expensive). You can create **alerts** to send you notification when CPU usage is too high for instance.

**CloudWatch Events**: helps you to response to state changes in your AWS resources

**CloudWatch Logs**: helps you to aggregate, monitor and store logs.

### AWS Cli

You can interact with all of AWS services from the cli using the aws command.

You need to input 

```bash
aws configure # Then enter your credentials
aws s3 ls # Lists your buckets
aws s3 mb s3://myawesomeawsclibucket # creates a bucket
```

By default your credentials are saved in .aws/credentials. That's a security issue so you should use roles instead.

#### Identity Access Management Roles

You can **create a role**, like **AdminRole** and then **attach it to an EC2 instance**.

Then **all the commands** you launch **from the EC2 instance** will have the **permissions from the role**.

When you create an EC2 instance in the AWS console, at step 3, **you can write a bash script in the "User Data"** field, it's **launched at startup**.

> I can **change the permissions of a role**, even if that **role is already assigned to an existing EC2 instance,** and these changes will take effect **immediately**. TRUE

### EC2 metadata

**From the CLI of your instances** you can get **metadata** about the current EC2 instance:

```bash
curl http://169.254.169.254/latest/user-data # Returns user-data (startup script)
curl http://169.254.169.254/latest/meta-data # List available information you can query
curl http://169.254.169.254/latest/meta-data/local-ipv4 # Returns your local IPV4
curl http://169.254.169.254/latest/meta-data/public-ipv4 # Returns your public IPV4
```

**169.254.169.254** is a special ip accessible only from **inside EC2 instances**

### EFS - Elastic File System

A managed NAS filer for EC2 instances.

A block storage you can **mount in multple instances** to share data. It **grows automatically**.

A great way to **share files between instances**. You can **mount** the **EFS** in **multiple instances** and any changes you make to a file is replicated instantly to all instances.

Like S3, you have **lifecycle** rules and a Infrequent Access (**EFS IA**) storage class. You can encrypt.

If not working, might be security group issue

- EFS supports NFSv4 (Network File Sytem).
- You only pay for the storage you use
- Can scale up to the petabytes
- Can support thousands of concurrent NFS connections
- Data is stored across multiple AZ's within a region
- Read After Write consistency
- Linux only, use FSX for Windows Server

### FSX

**FSX for Windows** is a File Server built on Windows Server, it's the EFS alternative for the Microsoft eco-system.

It supports AD (Active Directory) users, access control lists groups and security policies along with distributed file system. FSX uses on SMB.

**FSX for Lustre**

> Amazon FSx for Lustre is a fully managed file system [...] optimised for compute-intensive workloads, such as high-performance computing, machine learning, media data processing [...] Electric Design Automation (EDA)

Used to process massive datasets, get throughput of hundreds gigabytes per second, millions of IOPS, sub-millisecond latencies.

**FSX for Lustre can store data directly on S3**

### EC2 Placement Groups

- **Clustered placement group**: A way to **place your EC2 instances very close to one other** in the same Availability in order to get **low network latency and high throughput**. Only possible with certain instances.
- **Spread placement group**: the opposite. The instances are on different hardware, on different racks. That's useful if you have a critical app running and don't want your instances to all fail at the same time because of hardware failure. Instances are isolated. It's more **resilient**.
- **Partitioned Placement Group**: A mix of the previous two. Instances are partioned in small groups of instances very close to one-other, but each partition is on a separate rack. For HDFS, HBase, Cassandra

A **clustered placement group** is in **one availability zone**. A **spread/partioned placement group** can span **multiple AZ** but in the **same region**.

**Name** of the placement group **unique in AWS account**.

**Only** available for **certain types of instances** (Compute Optimized, GPU, Memory Optimized, Storage Optimised)

AWS recommend using same instance type inside group.

**You can't merge placement groups**.

You can **move an instance in a placement group** if it's stopped (only through CLI or API for now).

**Spread placement groups** have a specific limitation that you can only  have a **maximum of 7 running instances per Availability Zone** 

### HPC in AWS

Overview of how to do High Performance Computing on AWS

#### Data transfer

- Snowball, snowmobile (for terabytes or petabytes)
- AWS DataSync to store on S3, EFS, FSx for Windows, etc...
- Direct Connect: help to **create a dedicated network** from your **premises to AWS** (dedicated line from datacenter to AWS), reduce cost, increase bandwidth, more consistent network experience

#### Compute and networking

- EC2 instances
- EC2 fleets
- Placement groups (cluster placement groups)
- Enhanced networking => ENA or FNA

#### Storage

- EBS scale up to 64, 000 Provisioned IOPS
- Instance Store: Scale to millions of IOPS, low latency 
- Network storage: S3 or EFS, Amazon Fsx for Lustre, optimised for HPC, backed by S3, millions of IOPS

#### Orchestration and automation

- AWS Batch:
  - can batch 100 000+ computing jobs on AWS
  - a job can span multiple instances
  - You can easily schedule job
- AWS ParallelCluster: Open source management tool
  - Make it easy to deploy manage HPC clusters
  - Parrallel cluster uses a simple text config file to provision AWS resources
  - Automate create of VPC, subnet, cluster type, and instances

### AWS WAF

**WAF**: Web Application Firewall.

Firewall that has access to HTTP and HTTPS requests that are forwarded to Amazon CloudFront, Appliation Load Balancer or API Gateway. Let you access OSI level 7.

You can configure which IP addresses are allowed, or what query parameter (?lang=fr&country=fr) are necessary to pass the firewall. If the request don't match a rule, it returns 403 status code

Three behavior:

- Allow all except the ones you you specify
- Refuse all except the ones you you specify
- Count the requests matching a rule

Properties you can use in your rules: **Ip** address, **country** (POPULAR EXAM QUESTION: how do you block a country ?), **request** headers, regex match request, length of request, **presence of SQL**, **presence of script**

**IN EXAM: how to filter requests ? WAF or Network ACLs**

### Additional info from the QCM

- You can add multiple volumes to an EC2 instance and then create your own  RAID 5/RAID 10/RAID 0 configurations using those volumes.

## Databases

### RDS - Relational Database Service

**VM managed by AWS running a relational database. You cannot SSH into this VM**

**Security patching of OS and DB is AWS responsability.**

**RDS is not serverless except for Aurora serverless.**

Manage many types of relational databases for you (**Postgresql, mysql, SQL, Oracle, Aurora, MariaDB**)

Two key features :

- **Multi AZ** (for resilience). If one fail, DNS address point to replica db automatically. The backup is in another AZ. **For disaster recovery only** not for performance. Available for every DB but Aurora (who don't need it, it's already fault tolerant). You can enable this in your RDS instance configuration.
  - You **can force a failover** from one AZ to another by rebooting the RDS instance with failover.
- **Read Replicas** - For Performance. **You can have 5 copies max**. They are in sync. Each replica has it's own DNS address so you need to configure your EC2 instances accordingly. They are called **Read replica** because you **read from multiple databases** but you **write** to **one only**.
  - **You can have read replicas of read replicas (but it increase latency).**
  - You can promote a read replica to an independant DB.
  - This is **Asynchronous** replication (I think it means eventual consistency. There is **NO read after write consistency**). Perfect for read-heavy.
  - Read replica are **not available for Microsoft SQL Server**.
  - **Not** used for **disaster recovery (DR)**. 
  - **YOU MUST ENABLE AUTOMATIC BACKUP TO ENABLE READ REPLICAS**. Then you can do actions > create replica.
  - You can have **read replica in another region**.
  - Your read replicas can have mulit A-Z
  - **Data transfer** from your primary RDS instance to your secondary RDS instance **is free**

You can create an Aurora replica (and then promote it to independant DB). Good way to migrate to Aurora.

**Improve performance**: Read replica or Elasticache.

**NO-SQL** database / no-relational database

**Data warehousing**: Cognos, Jaspersoft, etc...

**OLTP **(Online Transaction Processing): get data for your website ***vs*** **OLAP**(Online Analytics Processing): get stats for your back-office

It's best to have a database for your website and another for stats, because computing stats => huge workload, might affect app performance. **That's what data warehousing is about**.

Amazon has a special product for this: **Redshift**. It's a data warehouse service.

**ElastiCache**: in memory cache, using Redis or Memcached

**RDS** => OLTP

**DynamoDB** => No SQL

**Red Shift** => OLAP (Warehousing) for Business Intelligence or Data warehousing

**Elasticache** => cache. Redis or Memcached. To speed up website and reduce workload database (cache frequent queries).



⚠️ You need to **add the security group of your EC2 instance** in the **inbound rules** of the **security group of your RDS database.** ⚠️ **Otherwise your database is NOT accessible from your instances** (unless the database is public) :

> MySQL/Aurora   TCP    3306     source: sg-myinstancesg

RDS run on a VM, but you cannot log in these instances.

**RDS** is **NOT serverless**, **EXCEPT** **Aurora Serverless**

**Max 16TB for provisioned IOPS with a Microsoft SQL Server database engine**

### RDS Backup

- Automated backup: recover at any point of time. **Retention period: from 0 to 35 days max.**
  - Full snapshot per day + transaction logs = can restore at any point of time down to a second
  - **Enabled by default**
  - **To disable backup you need to set retention period to 0 days** 
  - backup data stored in S3
  - You get **free storage** space equal to db size
  - **Backup taken during defined window**. During that window db I/O might be suspended and latency elevated.
  - Deleted when you delete your instance
  - **Enabling Backup can cause some downtime**
- Snapshots
  - Done manually
  - Stored even after you delete the db

When you restore a db it will be a new RDS instance with new DNS address.

**Encryption** at rest is supported for all six db engine (mysql, postgres, etc...) using KMS service. Backups, snapshot and replica are then also encrypted.

### DynamoDB

No SQL database

-  stored on SSD Storage
- Spread across 3 geographicaly distinct datacenters
- Two consistency:
  - **Eventual consistent (default)** - it takes usually **1second** for writes to affect all db
  - **Strong consistency** (can be enabled) block reads until write applied. **Less than 1 sec**
  - Remember the 1 sec rule, it might be an exam question

DynamoDB, like serverless, can be very expensive at a certain scale. (**3340$ per month** for Aurora vs **39,995$ per month** for DynamoDB)[https://www.abhishek-tiwari.com/dynamodb-or-aurora/]

### DynamoDB Advanced

**DynamoDB Accelerator (DAX)**

- It's a fully managed, highly available, **in-memory cache**
- **10x performance** improvement
- Reduce **requests** time from milliseconds to **microseconds**
- Failover to other AZ

Usually you have to create your own cache system, and invalidate cache when write or serve outdated data. You only get read performance improvements

**DAX** is between the Application and DynamoDB and can respond in microseconds.

DynamoDB support **transactions** (transaction take more read/write than traditionnal operations).

**ON-DEMAND PRICING:**

- **Pay-per-request** pricing
- Balance cost and performance
- No mimum capacity to purchase
- No charge for idle tables
- **BUT you pay more per request ** than provisioned capacity => **best for new products**

**ON-DEMAND BACKUP AND RESTORE**

- full backups at any time
- Zero impact on table performance or availability
- consistent within seconds and **retainted until deleted**
- in the same region as the source table (can't restore across regions)

**Point-in-Time Recovery (PITR)**

- Proctects against accidental writes or deletes
- Restore to any point in the last 35 days
- Incremental backups
- NOT ENABLED BY DEFAULT
- Lastest restorable: five minutes in the past

**STEAMS**

- Time-ordered sequence of item-level changes in a table
- stored for 24 hours
- inserts, updates and deletes
- stream-records have a stream number (their order) and are grouped in `Shards`

**Use cases:**

- Cross region duplication (global table). It uses streams to sync the tables
- Messaging apps, notifications

**GLOBAL TABLES**

Global table sync your tables between multiple regions. You add your target regions and that's all

- For globally distributed applications
- Based on DynamoDB streams
- Multi-regions redundancy for DR or HA
- Replication latency under one second
- Take some time to add a region to a global table

**Database Migration Service (DMS)**

Can be used to **migrate a database from an engine** (like mysql) **to another** (like dynamodb).
The source database can be on-premise. DynamoDB is a valid destination but NOT an available source for this tool.

**DynamoDB Security**

- All user data is encrypted using KMS
- Site-to-site VPN can be used
- Direct Connect
- IAM policies and roles are available for dynamodb
- Fine-grained access (you can allow access to only certain fields in a table)
- You can monitor dynamoDB with cloudwatch and cloudtrail
- VPC endpoints

### Redshift

**Redshift** is a way to do **business intelligence and data warehousing**. You want a specific solution for data warehousing because to can be a real burden on your production database.

- Single Node (160Gb)
- Mult-node
  - Leader node (manager client connections)
  - computed node (store data and process queries)
- Advanced compression: compress data by column (a lot easier to compress)
- Don't need indexes or materialized views
- Redshift scan the tables and chose the best compression scheme
- **Massively parallel processing** (MPP)
  - redshift distribute data and query load to your nodes
  - you can easily add nodes
- **Backup by default 1 day** (max 35 days)
- **Maintain 3 copies of your data** (original + replica on compute nodes + backup on S3)
- **Can also replicate your snapshots to S3 in another region for DR**
- Priced by compute node hours (total number of hours you run across all your compute nodes). 1 unit/node/hour. Not charged for leader node. Charged for backup and data transfer
- Security
  - Encrypted in transit using SSL
  - Encrypted at rest using AES-256
  - By default, redshift takes care of key management (but you can manage your own keys through HSM or KMS)
- **Currently only in one availability zone**. No multi AZ
- Cannot restore to other AZ

### Aurora

Aurora is Amazon proprietary database. It's MySQL and Postgresql compatible.
After some research it seems that Aurora is not really that much more expensive and have a lot of befenits.

Performance 5x better than mysql and 3x better than postgresql, at much lower price point.

- Start with 10GB, scales in 10GB to 64TB (auto-scaling)
- Compute resources can scale up to 32vCPU and 244GB memory
- **2 copies of your data is contained in each availability zone, with minimum of 3 availability zones. 6 COPIES OF YOUR DATA**

**Scaling**

- Aurora is designed to handled **loss of 2 copies** of your data, **without** affecting **write performance**, and the **loss of 3 copies for read performance**.
- Storage is self-healing (autoscanning for errors and repaired automatically).
- **Three types of Aurora read replicas** are available: Aurora (15 available/database), MySQL Read Replicas (5 / database), PostgresQL (1 / database). It seems best to use Aurora unless you want high-flexibility. For instance sync take a few milliseconds, low performance impact on primary, **automated failover (only available for Aurora)**.

**Backups**

- **Enabled by default**

- **Automated backups have no performance impact !** Snapshots have no impact either.
- **you can SHARE AURORA SNAPSHOTS with other AWS accounts**

**AURORA SERVERLESS:**

- On-demand pricing: you pay per request
- Auto-scaling - very easy to setup

- Perfect for **small website**,  (infrequent, intermittent or upredictable workloads).
- Otherwise it's better to use Aurora (cheaper).

### Elasticache

A service that manage **in-memory cache** system like **redis** or **memcached**.

When to use redis or memcached

- **Memcached** is simple, **multi-threaded**, has **horizontal scaling**, but not lot of features. Best for simple, performant cache. Simple to get started.
- **Redis** feature-complete but **single-threaded**. It has **multi-AZ** support, you can persist to disk, you have pub/sub capibilities, advanced data types, ranking/sorting data types. **Backup and restores**.

**Elasticache increase database and web app performance.**

### Database Migration Service (DMS)

DMS is a cloud service that makes it easy to migrate realtional databases, data warehouses, NoSQL databases and other types of data stores.

You can migrate from AWS to AWS, on-premise to AWS, AWS to on-premise, on-premise to on-premise.

AWS DMS is a server in AWS Cloud. You tell DMS the source, the target and you schedule the operation.

You can pre-create the target tables manually, or you can use **AWS Schema Conversion Tool (SCT)** to create some or all of the tables, indexes, views, triggers, etc...

**Two types of migrations** supported by AWS DMS:

- **Homogenous **migrations: engine **A** to engine **A** (ex: mysql --> mysql)
- **Heterogenous** migrations: engine **A** to engine **B** (ex: SQL --> Amazon Aurora)

**Source**: Orable, Microsoft SQL Server, Mysql, MariaDB, postgresql, SAP, mongodb, db2; **AzureSQL**, **Amazon RDS (including Aurora)**, AWS S3

**Target**: Oracle, Microsoft SQL Server, MySQL, MariaDB, postgresQL, SAP, RDS, Redshift, DynamoDB, S3, Elasticsearch service, Kinesis Data Streams, DocumentDB

For **Homogenous** migrations: You can for instance run DMS on a EC2 instance, pull data from on-premise and put it on RDS

For **Heterogenous** migrations: You need to run **both** **DMS** and **AWS Schema Conversion Tool** (SCT) on the EC2 instance

### Caching services

The following services have caching capabilities

- **Cloudfront**: Caching HTTP responses from your origin on Edge locations (closests to the user). Low latency
- **API Gateway**: if cache miss on cloudFront, then request go through API Gateway before reaching Lambda. Second best latency
- **ElastiCache** - memcached and redis: If cache miss on API Gateway, then lambda call elastiCache. Third best latency.
- **DynamoDB Accelerator (DAX)**: lambda can also get data from DynamoDB, and them it would access DAX cache. Similar latency with elatiCache.

These four service create a stack, and you want to put as much data on the first layers, because it has lower lantency.

### Elastic Map-Reduce

**EMR is used for big data processing**

- The industry-leading cloud big data platform for processing vast amounts of data using open-source tools such as Apache Spark, Apache Hive, Flink, Hudi, Presto, and so on.
- Petabyte-scale analysis
- half-cost, 3x the speed of traditional Apache Spark
- It's a **cluster** of E2 instances, called "**node**". Each node has a role, a **node type**:
  - **Master node**: manage the cluster. Monitor health, and sub-task progression. **Each cluster must have one.**
  - **Core node**: run tasks and **store data** in HDFS (Hadoop Distributed File System). **At least one**.
  - **Task node**: only run tasks but **does not store data (optional)**.
- Nodes can communicate with each other.
- Log data on master node, if you lose your master node (at /mnt/var/log), you might lose your data. **So you can periodically archive the log files stored on the master node to Amazon S3**. **Default 5min interval**. **YOU NEED TO SETUP THIS WHEN YOU CREATE THE CLUSTER** (you can't after).

## Advanced IAM

### AWS Directory Service

So this part is a bit confusing because AWS gather some unrelated services under one name: Directory Service. But actually, Cloud Directory has nothing to do with Microsoft AD for instance.

Many companies use AD users to authenticate their employees. They want their employees to login once with AD, and use multiple services. You can have group policies. Azure directory users LDAP and DNS.  Support Kerberos, LDAP  and NTLM anthentication. Highly avaiable.

- AWS Directory service is a family of services
- Connect AWS resources with on-premise Active Directory
- Standalone directory in the cloud
- You can use those AD account to login to AWS console.

Three services: **AWS Managed Microsoft AD**, **Simple AD**, **AD Connector**

**TODO**: understand this part better

#### AWS managed Microsoft AD DC (Domain controller)

 managed microsoft servers running AD. You want multi servers in multiples AZ. Each group of domain controller is in its own VPC.

You can extend existing AD to on-premises using AD Trust.

**What AWS manage for you:**

- Multi-AZ deployment
- patch, montor, recover DC
- Instance rotation (update software)
- snapshot and restore

**What you need to manage:**

- users, groups, GPOs
- Standard AD tools
- Scale out DCs
- Trusts (resource forest)
- Certificate authorities (LDAPS)
- Federation

#### Simple AD

- standalone managed directory
- basic AD features
- small: <= 500; large <= 5,000 users
- Make it easier to manage EC2 when you want to use your existing corporate credentials
- Can't join on-premises AD

#### AD Connector

- Proxy for on-premises AD
- Avoid caching information in the cloud
- Allow on-premises users to log in to AWS using AD
- Join EC2 instances to your existing AD domain
- Scale across multiple AD Connectors

#### Cloud Directory (not AD compatible)

Not AD compatible

- Directory-based store for developers
- multiple hierarchies with hundreds of millions of objects
- Use cases: org charts, course catalogs, device, registries
- Fully managed service

#### Amazon cognito user pools

Not AD compatible

### IAM Policies

#### ARN - Amazon Resource Name

**A unique identifier for AWS resources**

All arn begin with `arn:partition:service:region:account_id:` (some parts can be omitted, like region if global resource)  and end with `resource` or `resource_type/resource` or `resource_type/resource/qualifier` (`/` can also be `:`).

Ex `arn:aws:dynamodb:eu-central-1:123456789012:table/orders` or `arn:aws:s3:::mybucket/image.png`

partition: aws has multiple infrastucture (ex: aws, the most common, vs aws-cn for china)

You can use wildcards to describe multiple resources:
`arn:aws:ec2:us-east-1:123456789012:instance/*`

#### IAM Policies

JSON document that define permissions. It's composed of:

- Identity policy. A policy you attach to a IAM user, group or role
- Resource policy. A policy you attach to a resource, like a S3 bucket.
- No effect until attached
- **List of statements**
- Each statement match an AWS API request (an action you perform in aws, like delete `mys3bucket`)

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "HumanReadblePolicyId",
      "Effect": "Allow", // Or "Deny",
      "Action": ["dynamodb:Query", "dynamodb:Get*"], // what we allow, or deny
      "Resource": "arn:aws:dynamodb:*:*:table/users" // the target of the policy
    }
  ]
}
```

- **AWS managed policies**: pre-created policies by aws (cannot be edited)
- **Customer managed policies**: created by you. You can create your own policies by writing json like above.

**Example**: You want to give your EC2 instance access to a specific bucket called test. You create a policy called TestBucketPolicy, then you create a TestBucketRole role with this policy and you assign this role to your EC2 instance.

**Inline policy**: you create a policy inside a role for instance, and it's not available outside that role, so you cannot re-use it.

- **EVERYTHING NOT EXPLICITLY ALLOWED IS FORBIDDEN**
- **If you deny something, it override any allow rule you might have**
- Only attached policies have an effect
- AWS joins all applicable policies (if you have a role with multiple policies)

#### Permission boundaries

- Used to delegate administration to other users
- Set the maximum permissions of an user. **Prevent privilege escalation** or **unnecessarily broad permissions**
- Use cases:
  - Developers creating roles for Lambda functions
  - Applications owners creating roles for EC2 instances

So if you have an user with AdministratorAccess permission access (give access to everything) but then give him an AmazonDynamoDBFullAccess permission boundary, he will only have access to dynamoDB.

### AWS Resource Access Manager (RAM)

Before we've talk about how you can create multiple root account and connect them to create an AWS organisation.

**RAM allows resource sharing between accounts**. Only a few services support this however: App Mesh, Aurora, CodeBuild, EC2, EC2 Image Builder, License Manager, Resource Groups, Route 53

So if you want to share a resource, you go in RAM service, you select the resource you want to share, then you type the id of the account you want to share it with. It will send an invitation. **You must accept the invitation in the console in the RAM service** !

### AWS Single Sign-On (SSO)

So you've spread your AWS resources into different AWS accounts for security reasons, but now you have to login in the right account everytime you want to do an action. If you have many account it's a pain.

You can you SSO to log-in once and access all the accounts you're allowed to access.

This can **integrate with external services** (ie: you can log in G suite or Office 365 with your AWS SSO), so you can use your existing corporate credentials. You can get your AWS SSO credentials from On-premise **Microsoft AD**. You can also use your AWS SSO credentials to  log in **SAML 2.0-enabled applications.**

All login through AWS SSO is recorded in CloudTrail

**YOU PROBABLY WON'T NEED MOST OF WHAT'S COVERED IN ADVANCED IAM, BUT IT'S WORTH A NUMBER OF POINTS IN THE EXAM**

## Route 53

**By default you are limited to 50 domain names.** You can increase the limit by contacting AWS.

### DNS - Domain Name System

**IPv4** -> 2^32 addresses, **IPv6** -> 2^128 addresses

**Top Level Domain**: .com, .gov, etc... Controlled by IANA (Internet Assigned Numbers Authority). [http://www.iana.org/domains/root/db](http://www.iana.org/domains/root/db)

Domains are **registered** with **InterNIC**, a service of **ICANN**, which enforces uniqueness of domain names across the Internet. Each domain name registered in a central database know as the **WhoIs** database.

A DNS domains contains the **Start of Authority Record** (SOA record) which in turn contains:

- The name of the server that supplied the data for the zone
- The administrator of the zone
- The current version of the data file
- The default number of seconds for the **time-to-live** file on resource  records

**Name Server Records** (NS records): Used by Top Level Domain servers to direct traffic to the Content DNS server. Ex: example.com,  record type: NS, value: ns1.exampleserver.com, TTL: 21600

**TTL** (time-to-live): how long DNS answer is cache. (Ex, for A records, how long the returned IP for a domain should be cached).

**Multiple types of record**:

- **"A" record** (Address record): translate domain to IP. You can put multiple IPs (with AWS, one is picked randomly, good for load balancing)
- **"CName" record** (canonical name record): translate domain to other domain. **CNAME cannot point to naked domains name** or **zone apex** (~= second level domain. example.com = naked domain, www.example.com != naked domain). Ex: `foo.example.com` points to `bar.example.net`. In that case you resolve `foo.example.com`, it returns `bar.example.net`. Then you resolve `bar.example.net` and it returns an IP.
- **Alias record**: Like CNAME, but without the limitation for naked domain. You resolve `foo.example.com`, but then the DNS server will resolve `bar.example.net` for you and returns directly the IP. So it's faster. However you will lose the geographic location of the user in the process.

The difference between CNAME record and Alias record is detailed here: [https://help.ns1.com/hc/en-us/articles/360017511293-What-is-the-difference-between-CNAME-and-ALIAS-records-](https://help.ns1.com/hc/en-us/articles/360017511293-What-is-the-difference-between-CNAME-and-ALIAS-records-)

**TIPS:**

- **Elastic load balancers** (ELBs) don't have an IPV4 by default, only a DNS name
- You must understand the difference between an Alias Record and a CNAME record.
- **Always chose Alias when given the choice**

**Common DNS types:**

- SOA records
- NS records
- A Records
- CNAMES records
- MX records for mails
- PTR Records: the inverse of A record (you have the IP and you want the domain)

**Some practical tips:**

- You can buy a domain name in AWS route 53

- A domain registration can take up to 3 days, but usually 1 or 2 hours

### Route policies

Many types of routing policies with Route53

- simple routing
- weighted routing
- latency-based routing
- failover routing
- Geolocation routing
- Geoproximity routing (traffic flow only)
- Multivalue answer routing

#### Simple routing

Simple routing is a **A record**, with **potentially multiple IP addresses**. When the address is resolved, **one IP address** is picked **randomly** amongst the listed IP addresses.

#### Weighted Routing Policy

You can assign a weight to each IP address. Ex: you can ask AWS to redirect 20% of DNS requests to IP "30.0.0.1" and 80% to IP "30.0.0.2". (Sounds great for A/B testing).

You need to create one **A record per weight with one IP only**. Then you select "Weighted routing" in policy, you chose a **weight**. You must also type a **setId**, to tell AWS the records are related. The percent of traffic redirect to that record is `(record weight / sum of record weight having the same setId) * 100`

Interesting option: Association with Health Check. If pointed resource fail health check, ignore it in DNS record

**Health Checks**: In route53 you can create a health check. It will send an HTTP query every 30 seconds (or 10 seconds) to an IP of your choice. If the query fails multiple times (you can configure the exact number), the tested server will be considered down and the health check will fail. You can get a notification if the test fail.

#### Latency-based routing

Will resolve the domain to the server with the lowest latency for the user (so probably the closest to the user, but not always). If you have a example.com domain with a server A in France and a server B in japan, a user in Japan will be redirected to server B and a user in France to server A.

For each server, you must indicate to AWS in which region it's located so that AWS can know the latency. Like for weighted routing policy, you also need to indicate a `setId`.

#### Failover routing

If one server fail, we have a fallback server. The default server is called "active", the backup server is called "passive".

#### Geolocation routing

Everything is in the title.

#### Geoproximity Routing

**DON'T CONFUSE THIS WITH GEOLOCATION ROUTING**, IT'S DIFFERENT

Don't seem very useful in practice.

It's beyond the scope of the Certified Solutions Architect Associate exam. You need to enable traffic flow to use this. Basically you can associate coordinates (lat, long) and a bias (a coefficient giving a weight to the location) with a server. The users will be routed to the server associated with the closest coordinate. The calculated distance is affected by the bias.

**Traffic flow**: Amazon has a visual diagram editor to manipulate DNS rules. It's used to create complex rules. It's best to avoid using it unless you really need it because it costs : `$50.00 per policy record / month`. Outch.

#### Multivalue answer policy

Like simple routing, however you can put health checks on every server, and only servers that pass health check are taken into accounts.

## VPC

Probably the most important part of the exam

From aws doc:

![VPC overview](https://docs.aws.amazon.com/vpc/latest/userguide/images/nat-gateway-diagram.png)

Another I found from [here](https://medium.com/an-idea/create-a-secure-aws-vpc-architecture-fd4aeb0f0b25). It was also in the slide of the udemy course.
![VPC security](https://miro.medium.com/max/1400/1*bGOy-meClkVOmcQGX8V-kw.png)

A subnet is a range of local IP addresses. You usually have a public one, with your backend, and a private one with your database. It's in the same VPC so every instance in the public subnet has access to the database in the private on, but it's not accessible from internet.

An ip range is represented like this : 10.0.0.0 / 16. In that case 10.0 is the prefix of all your ips. 16 mean that the 16 first bits are the prefix, their are fixed. So in that case the range is 10.0.0.0 to 10.0.255.254. That gives you around 2^16. **16 is the minimum number of prefix bits** (so you can have around 65 000 IPs in that subnet) and **28 is the maximum** (in that case you get 16 IPs in your subnet).

These ranges are called **CIDR**. Use [CIDR.xyz](https://cidr.xyz/) to visualize and understand better what I mean.

**There is a default VPC: All subnets are public in that VPC.**

Each EC2 instance has both a public and private IP address.

You can do **VPC peering** : you connect two VPC. It's like instances are on the same private network.

You can peer VPC's with other AWS accounts. You can peer across regions. **No transitive peering !**

**Transitive peering**: If VPC A is peered to VPC B and VPC B to VPC C, VPC A is not peered with VPC C (it can't talked to instances inside VPC C)

- **VPC = a logical datacenter in AWS**
- Consists of IGW (Or Virtual Private Gateways), Route Tables, Network Access Control Lists, Subnets, and Security Groups

- **1 subnet is in one Availability zone only**, but you can have multiple subnets in one availability zone.

- **Security Groups are Stateful; Network Access Control Lists are Stateless**

#### Creating a VPC

**When you create a VPC it automatically create:** a **Route Table**, a **Network ACL**, and a **Security Group**. It does **NOT** create a **subnet**, it does **NOT** create an **internet gateway**.

**In order to use the VPC we need to create a subnet.**

**When you create a subnet, 5 ip address are reserved**: Network IP (10.0.0.0), VPC Router (10.0.0.1),  DNS Server (10.0.0.2), Reserved by AWS for future use (10.0.0.3), Broadcast IP (10.0.0.255).

**By default**, our subnet are **private**. You can enable **public IPv4 auto-assign** (actions => Modify auto-assign IP settings).

Then you need to create an **internet gateway**. By default, it's dettached. You can attach it to a VPC (actions > Attach).

**You can only have ONE internet gateway per VPC**.

Now we need to route the traffic with **routing table.**

By default the route table **allow all communication from inside VPC** to inside VPC, by **NOT from internet**.

You have one **main route table** per VPC. **By default subnets use this main route table.**

If we modify our main route table to allow communication with internet, it would affect all the subnets of our VPC. That's a security concern.

**So we want to create a route table for our public subnet.**

So we need to create a new route in PublicRoute table, with the following parameters:

- 0.0.0.0/0 allow traffic from all IPv4 addresses
- We select our internet gateway as the target.

**The finally we need to associate our public subnet to our PublicRoute** (Select the route table, subnet assocation tab)

**Security groups belong to one VPC. You cannot use the security group from one VPC in another VPC**

#### Communication between subnets

##### Allow communication between security groups

**By default, security groups do not allow access from instances inside other security groups.**

**So what you want to do is editing the security group of your private instances and allow traffic from your public instances.** Just allow ip addresses ranges: 

Example: All ICMP (allow ping)     TCP    source: 10.0.1.0/24

#### Communication with outside world (NAT instances & NAT gateways)

 **⚠️ By default the instances in your private subnet don't have access to internet !** 

So you can't download updates or anything.

NAT: Network Address Translation.

**Most of time you would use NAT gateways** (but sometimes you might you NAT instances).

A **NAT instance is an EC2 instance** with a special AMI, **inside your public subnet**.

By default all instances have a "source and destination check". It's meaning all traffic from and to the instance must have the instance IP as the source or the destination. We need to disable that for the NAT instance.

Then you need to create a route so that the private instance can talk to the NAT instance: Destination 0.0.0.0/0  target: NAT_instance

The problem with this is that if the NAT instance crash or is overwhelmed, all the instances using.

**So NAT instance = bottleneck + single point of failure**

Of course you could create Autoscaling groups, mulitple subnets in different Azs and a script to automate failover, but It's **better** to use a **NAT Gateway**

You create your **NAT Gateway** in your public subnet

**You create a route in your Route Table**. **We want all traffic with as destination any IP address to go through our NAT gateway, which has access to internet**.

NAT gateway are:

- redundant inside AZ
- cannot span multiple AZ (one NAT gateway = one AZ)
- able to handle 5gps and scale to 45gpbs
- no need for patch
- not associated with security group
- automatically assigned a public ip address
- no need to disable source/destination checks

**You should create one NAT gateway per AZ**

### Network Access Control Lists (NACL)

NACL is a bit like Security groups, it a kind of firewall. The difference is that you can have **allow rules** and **deny rules**.

When you create a NACL, it's going to **deny everything by default**.

If you associate a subnet to your new NACL, it's going to remove your subnet from it's previous NACL because **subnets can only have ONE NACL**

When you add a rule, it's recommanded to add +100 to the rule number. This gives you space to add morme rules between to rules. 

Another **important difference with security group**: If you add an **inbound rule**, you also need to add an **outbound rule**. The **traffic is not going to be allowed to go out automatically like with security group**.

⚠️ In the outbound rules, **the port is the DESTINATION port, not the SOURCE port** !

Therefore we need to allow the range of **Ephemeral port** (1024-65535).

Basically, when a client connects to an HTTP server, it connects to port 80, but the HTTP server send the response to a random **Ephemeral port** (between 1024 and 65535).

⚠️ You also need to allow **Ephemeral ports** in **inbound** rules if you want your server to be able to make an  HTTP request to another server, because is time your serveur will be the client, and will receive the HTTP response on an ephemeral ports.

⚠️ **Rules with a higher rule number are overwritten by rules with lower rule number.**

So this means rule 100 overwrite rule 300.

**When you create a subnet, it will use the default NACL or the VPC.** 

**Summary**:

- VPC has default NACL allow all outbound and inound traffic
- You can create custom NACL, which by default denies all inbound and outbound traffic.
- Each subnet in your VPC must be associated with one NACL, and only one.
- By default, the subnet use the VPC's default NACL
- You can block specific IP using NACL, but not with security groups
- A NACL can have many subnet, but a subnet has only one NACL
- The rules with the lower rule number overwrite those with the higher rule number (100 overwrite 300)
- Each rule can allow or deny traffic
- NACL are stateless: responses to allowed inbound traffic are subject to the rules for outbound traffic (and vice versa).

### Custom VPC and ELB

You need to put the load balancer in a public subnet

**YOU NEED TWO PUBLIC SUBNETS FOR LOAD BALANCERS**

### VPC flow logs

Flow logs is a way to capture information about the IP traffic going to and from network interfaces in your VPC.

You can capture traffic at **VPC level, subnet level or network interface level**.

Select VPC, actions => create flow log

You can chose to log **accepted traffic, rejected traffic or both**

You can send the logs to CloudWatch or to S3.

If you select CloudWatch you need to **create a log group**. You also need a role. You can click on "setup permission" to ask AWS to help you create the role.

- You can only enable flow logs for VPC that are in your account.

- You can tab flow logs.

- After you created a flow log you cannot change its configuration.
- Not all traffic is logged. Here is a list of what is **NOT LOGGED**:
  - traffic when instances access Amazon DNS server.
  - traffic for Amazon Windows license activation
  - traffic for 169.254.169.254 for instance metadata
  - DHCP traffic
  - traffic to the reserved IP address for the default VPC router (ex: 10.0.0.0)

### Bastion Hosts

A bastion server is in the public subnet and is basically used as a proxy to SSH into an instance in the private subnet from the internet. It's meant to be exposed to untrusted traffic and used as a kind of guard. It need to be hardened and secured as much as possible.

- NAT Gateway or NAT instance is used to provide internet traffic to private EC2 instances.
- A Bastillon is used to securely administer EC2 instances (Using SSH or RDP). (also called Jump boxes in Australia)
- You cannot use a NAT Gateway as a Bastion host

So basically you need to create an EC2 instance with an Bastion AMI from the community in your public subnet.

### Direct connect

Help to etablish a dedicated network connection from your premises to AWS.

![Direct connect diagram](https://miro.medium.com/max/1400/1*Bf1CjyOKPeiybahOjCOMSg.png)

Taken from [this article](https://medium.com/awesome-cloud/aws-direct-connect-overview-introduction-db468ae6a582).

**What you need to remember for the exam:**

- Direct Connect connects your data center to AWS
- Useful for high throughput workloads (ie lots of network traffic).
- Or if you need a stable and reliable secure connection.

If you have scenario based question where asked what you can do when to VPC connection that keeps drop-out because of the high throughput, the answer is Direct Connect.

Apparently now you need to know the step to create a Direct Connect connection for the exam. Here is the video on [youtube](https://www.youtube.com/watch?v=dhpTTT6V1So)

**TODO: learn those steps.**

### Global accelerator

It works much like S3 transfer acceleration. It's explained [here](https://aws.amazon.com/global-accelerator).

Regular connection to EC2 instance from client on internet:

![internet traffic](https://d1.awsstatic.com/r2018/b/ubiquity/global-accelerator-before.46be83fdc7c630457bba963c7dc928cb676d9046.png)

And through Global accelerator:

![](https://d1.awsstatic.com/r2018/b/ubiquity/global-accelerator-after.2e404ac7f998e501219f2614bc048bb9c01f46d4.png)

Your client connect to the closest Edge location which act as a proxy and transfer the traffic through AWS private, optimised network.

1. Global accelerator procides you with **two static IP addresses** (you can also chose your own) that you associate with your accelerator
2. your accelerator direct traffic to optimal endpoints over the AWS global network. Each accelerator includes one or more listeners.
3. Your accelerator has a DNS address like a1234567890abcdef.awsglobalaccelerator.com. That points to the static Ip addresses that Global Accelerator assigns to you. You can use the IP, that DNS address or your own domain.
4. Network zone is a set of physical infrastructure. It's like an availability zone but for network. Each of the two static IP points to a different network zone. If one IP is unhealthy, it falls back to the second and thus the other isolated network zone.
5. Listener processes inbound connections from clients to Global Accelerator. Supports both TCP and UDP.
6. Endpoint group : associated with a specific AWS Region. End group include one or more endpoints in the Region. You can specify the percent of traffic going to a endpoint group. these percent are called **traffic dials**
7. Endpoints: EC2 instance, loadbalance. You can configure weights to determine the percent of traffic directed to a particular endpoint.

### VPC endpoints

**Problematic**: you have a EC2 instance in your private subnet. You want it to access one of your S3 buckets.

You have two ways to do this:

- Using NAT gateway (or NAT instance). The problem with this method is that your traffic will go through internet.
- The second option is to use **VPC endpoints**. The traffic will stay on the private network.

So **VPC endpoint** enables you to privately connect your VPC to supported AWS services **without going through the internet** with internet gateway, NAT gateway, etc... This way, the instances in your VPC don't need a public IP address to communicate with the AWS service. The traffic don't leave the AWS network.

VPC endpoint are virtual device. They are horizontally scaled, redundant, and highly available.

**Two types of endpoints:**

- **Interface endpoint**s: It's an elastic network interface with a private IP address that serves as an entry point for traffic desitned to a supported service. **A LOT of services are supported**. You attach this ENI to your EC2 instance and it will allow you to communicate with these services. You are charged per hour and per GB processed.
- **Gateway endpoints**: Supported only for **S3** and **Dynamodb**. Gateway endpoints are cheaper.

So can you **create an endpoint in your VPC**: you select the **service** (S3 in our case) in the service list, you select your VPC, you select your Route Table and finally you can configure an endpoint policy.

When you create the endpoint, it will create a Route in your Route Table. But it can takes 5min before it appears.

⚠️ if when you try to access your S3 bucket from the instance, it times out, it's because **you need to add the --region parameter**.

```bash
aws s3 ls #times-out
aws s3 ls --region us-east-2 # that works
```



### AWS PrivateLink

We want to open our applications up to other VPC. Two options:

- Open the VPC to internet => security issues
- VPC peering => If you want to share your VPC to many VPC you will have many peering relationships to manage. Also the whole network will be accessible.

So aws created PrivateLink.

Expose your service inside a VPC to an other VPC without making it available on internet.

- Best way to share your service with teens, hundreds or thousands of customer VPC
- No VPC peering, route tables, or NAT, IGWs required
- But: **Requires** a **Network Load Balancer on the service VPC** and an **ENI on the customer VPC**

### AWS Transit Gateway

Transit gateway allow you to simplify your network architecture. The transit gateway acts like a giant hub.

![Without transit gateway](https://d1.awsstatic.com/product-marketing/transit-gateway/tgw-before.7f287b3bf00bbc4fbdeadef3c8d5910374aec963.png)

![with transit gateway](https://d1.awsstatic.com/product-marketing/transit-gateway/tgw-after.d85d3e2cb67fd2ed1a3be645d443e9f5910409fd.png)

- Allows you o have transitive peering between thousands of VPCs and on-premises data centers.
- Works on a hub-and-spoke model
- Works on a regional basis, but you can have it across multiple regions
- You can use it across multiple AWS accounts using RAM (Resource Access Manager)
- You can use route tables to limit how VPCs talk to one another
- Works with Direct Connect as well as VPN connections
- Supports **IP multicast** (not supported by any other AWS service)

### AWS VPN CloudHub

Allow you to have a single VPN gateway for multiple private networks outside of AWS.

![aws vpn CloudHub](https://docs.aws.amazon.com/vpn/latest/s2svpn/images/AWS_VPN_CloudHub-diagram.png)

- If you have multiple sites, each with its own VPN connection , you can use AWS VPN CloudHub to connect those sites together.
- Hub-and-spoke model
- Low cost; easy to manage.
- It operates over the public internet, but all traffic between the customer gateway and the AWS VPN CloudHub is encrypted.

### Networking Costs

The **traffic** coming in **from the internet to your VPC is FREE**.

The **traffic between two instances in the same AZ is free** if you use the **private IPs**.

If you use the **public IPs, the traffic will go through the internet**, and thus it's **NOT FREE** (something like 0.02$/GB for instance (not real price, it's an example)).

The traffic between two instances in different AZ but within the same region is **NOT FREE**. But it's **cheaper than going through the internet**.

Finally traffic between regions is **NOT FREE**. It's more expensive than traffic between AZ, and cheaper than traffic through the internet.

- Use private IP addresses over public IP addresses to save money. It uses AWS own network rather than internet.
- If you want to minimize your network costs, put all of your EC2 instances in the same AZ. However that's a single point of failure, something you want to avoid.

