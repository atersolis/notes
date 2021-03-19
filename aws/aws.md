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

