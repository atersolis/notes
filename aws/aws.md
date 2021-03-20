# AWS Associate Architect traning notes

Notes and cheat sheets made while reading [this udemy training](https://www.udemy.com/course/aws-certified-solutions-architect-associate)

## IAM (Identity and Access Management)

When you create an account on AWS, it's a **root account**. The **root account** is a super admin account, it have all right on everything.

So you need to enable **MFA** (multi-factor auth), for instance by installing Google Authenticator on your phone. Make sure you have a **backup** for your **MFA**, because if you don't, when you lose your phone, you lose your account, forever.

This root account can create other account with restricted access. These accounts can login through a special url, something like https://myalias.signin.aws.amazon.com/console

AWS has a very powerfull access control system.
You need to understand four concept : **users**, **groups**, **roles** et **policies**

- **policies**: are just permissions basically. AWS has a long list of are-made policies like **AdministratorAccess** which give you all right on everything, or **AmazonS3ReadOnlyAccess**. But you can also make your own **policies**. For instance, **AmazonS3ReadOnlyAccess** allow you to access all S3 buckets. But might want create a policy to access only one specific S3 bucket.
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

- **roles**: are sets of policies. You can assign a role to an user, **but also, to an AWS ressource, like a EC2 instance**. You can for instance create a role so that myEc2Instance, and only myEc2Instance can access myS3Bucket.
  From AWS IAM FAQ:

  > **Q: How do I assume an IAM role?**
  > You assume an IAM role by calling the AWS Security Token Service (STS) AssumeRole APIs (in other words, AssumeRole, AssumeRoleWithWebIdentity, and AssumeRoleWithSAML). These APIs return a set of temporary security credentials that applications can then use to sign requests to AWS service APIs.

- **Groups**: they contain users. You can assign them policies directly, and every user in the group will have these permissions.

- **Users**: an account created by the root account. You can put the user in a group, or directly assign policies to the user.

When you create an user, if you check "programmatic access", it will generate an **access Id** and a **secret key**. YOU NEED TO SAVE THE SECRET KEY RIGHT AWAY, OR IT WILL BE LOST FOREVER.

Five things you can after creating your root account, in IAM:

- Create an alias for the signin url. By default it's something like (https://123456789012.signin.aws.amazon.com/console), you can make it easier to remember (like https://myalias.signin.aws.amazon.com/console)
- Enable MFA for the root account
- Create at least one user and avoid using the root account
- Create at least a group
- Create a IAM password poilcy (how often should user change their password, minimum length, etc...)



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
- **S3 One zone IA**: same thing but even cheaper. It's only in one Availibility zone so the availibility is lower.
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

**Glacier Vault Lock**: similar to S3 Object lock



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

