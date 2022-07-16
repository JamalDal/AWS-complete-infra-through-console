
# AWS complete setup through console

We will start from creating a VPC, SG, IAM role, LC, LB, AS, Code deploy, RDS, Redis, S3 bucket, CloudFront, SNS, CloudWatch, VPN, and to Elastic IP. 

We will be taking us-east-1 (N. Virginia) region to deploy ot infra and name would be dev-env.

https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/console-help-gsg.pdf




## Create a VPC 
Name: vpc-dev-env 

CIDR range: 172.20.0.0/16

![vpc-image](https://user-images.githubusercontent.com/97054844/179372660-ea0ab91b-d649-477f-9f57-a3e14aa1671b.png)


### Create Subnet
Create four subnets: one public and one private subnet in the availability zone us-east-1a, and one public and one private subnet in the availability zone us-east-1b.


Subnets CIDR-Ranges:

•	public-subnet1-dev-env = 172.20.0.0/24 us-east-1a

•	public-subnet2-dev-env = 172.20.16.0/24 us-east-1b

•	private-subnet1-dev-env = 172.20.32.0/24 us-east-1a

•	private-subnet2-dev-env =172.20.64.0/24 us-east-1b

![subnet-image](https://user-images.githubusercontent.com/97054844/179372673-2d6481aa-185f-4a56-a99b-67bbfa220edc.png)

Now, we create two route table, one for public (RT-public-dev-env) and other one for private (RT-private-dev-env).
Then add them in VPC and after that associate subnets in them.


![RT-public-dev-env](https://user-images.githubusercontent.com/97054844/179372674-a7b76f5d-213b-4034-a0cd-9b168b1af0ea.png)



![RT-private-dev-env](https://user-images.githubusercontent.com/97054844/179372676-ee5c89bd-0541-40f3-a403-f5208df1105d.png)


Now, create an internet gateway (name: igw-dev-env) attach it with vpc-dev-env & attatch the public router table (RT-public-dev-env) with it.


Then create a NAT gateway (NAT-dev-env) in public subnet with static IP (Elastic IP) and attached with RT-private-dev-env . This is to provide internet accessibility to private server.


![igw](https://user-images.githubusercontent.com/97054844/179372678-21a31d0c-0287-4e8d-a993-c862831beb2e.png)



![nat](https://user-images.githubusercontent.com/97054844/179372679-c0d3e5c3-bfdc-4bf5-9f54-dad3ecc0b6d9.png)



## Security Group
We may set up multiple security groups for various resources. Mention the name of the security group to distinguish it.

### Security group for load balancer (name: SG-LB)
Select vpc-dev-env and open port HTTP (80) and HTTPS (443) for inbound traffic to all.

![SG-LB](https://user-images.githubusercontent.com/97054844/179372683-e9ae1c5c-c3b9-4371-b60f-56f5013c5aec.png)


### Security group for Redis (name: SG-Redis)
Select vpc-dev-env and and open port 6379 from Webserver- security group.

#### SG-Redis image take later and put


### Security group for VPN (name: SG-VPN)
Select vpc-dev-env and open ports 22, 80, 443, 1194, 12320, and 12321.

![SG-VPN](https://user-images.githubusercontent.com/97054844/179372684-d8b09f2a-f21d-4b07-85b0-b29ffc733e14.png)


NOTE: After completing the infrastructure, we should remove ports 80, 22 and 443 from the SG-VPN.


### Security group for launch configuration(web-server) (name: SG-LC)
Select vpc-dev-env add SG-LB and SG-LB at port 80 and 443, SG-VPN at  port 22 allowed.

![SG-LC](https://user-images.githubusercontent.com/97054844/179372739-f544b584-7699-4a98-9d41-210616f7bfc4.png)

### Security group for database (name SG-DB)

Select vpc-dev-env and SG-LC and MYSQL/Aurora port 3306 allowed.

![sg-db](https://user-images.githubusercontent.com/97054844/179372741-c98cbf9e-9167-4f90-8bdd-bf433bf8ab01.png)


## Create IAM Role for EC2 service (name: s3FullAccessRoleForEC2)
To provide s3 full-access to EC2

![s3FullAccessRoleForEC2](https://user-images.githubusercontent.com/97054844/179372742-c8f37560-fcd1-4c83-944c-1ecdcea9e725.png)


## Create IAM Role for CodeDeploy service (name: CodeDeployRoleforEC2)
To provice CodeDeploy access

![cdrole](https://user-images.githubusercontent.com/97054844/179372769-5d2e2f1c-6fe6-4938-8f45-57262d470dce.png)


## Create Launch Configuration 
Take the Amazon Linux AMI with the instance type t2.small. Attach the ec2-iam role (s3FullAccessRoleForEC2) and SG-LC along with the following preinstalled on user data to install the codedeploy agent on servers at the time of boot.

```bash
#!/bin/bash
yum -y update
yum install -y ruby
cd /home/ec2-user
curl -O https://aws-codedeploy-us-east-1.s3.amazonaws.com/latest/install
chmod +x ./install
./install auto
```

## Create LoadBalancer (name: alb-dev-env) 
Select vpc-dev-env and select public subnet to both availability zones.
ALso attach SG-LB
Application LB and Internet facing

Moreover, In configure routing create new target group and name is tg-dev-env.

![alb-dev-env](https://user-images.githubusercontent.com/97054844/179372772-22a281cf-058e-44bb-ad57-a1081abb0c67.png)


## Create AutoScaling group (name: asg-dev-env)
Attach with vpc-dev-env and both private subnets. Also attach launch configuration profile
Select tg-dev-env as target group to forward the traffic to specific targets.

Here we can define Desired, Minimum and Maximum capacity.

Note: In new console, AWS has removed ASG console

![asg-dev-env image](https://user-images.githubusercontent.com/97054844/179372774-b2e8801e-a82e-4a31-9dca-52cecfa83307.png)


## Create CodeDeploy 
Create an application first and select Ec2/On-premises on the compute platform.

![application-dev-env](https://user-images.githubusercontent.com/97054844/179372777-d3154881-48a7-4226-8382-41e562a97bed.png)

Then create a deployment group (name: app-deploy-group-dev-env) and attach code to it to deploy an IAM role.

For now, the deployment type is In-place deployment, with the deployment setting of OneAtAtime. and attached to the tg-dev-env target group of your application load balancer.


### Create an RDS
Take the Amazon Aurora engine with a production template. The DB cluster identifier is dev-env, and the classes are t2.micro and vpc-dev-env with NO public access. Select the SG-DB-dev-env security group, enter the RDS user name and password, and create it. (accessible only through its security group).

### Create Redis
Put it in private subnet and attach SG-Redis

### Create S3 Bucket
Create four of them: Codedeploy, store logs, media,and for static data, one for each.

### Create cloudfront
In origin domain name select dev-env-static bucket and create cloudfront distribution.

Then we can create SNS topic for notification. In this case, we can select cpu-alert type.

Further, we can crate Cloudwatch dashboard for monitoring purpose as per the requirements, also we cna set alram and attach SNS topic to that alarm.

### Final step to create a VPN server
Take a look at the AWS market place for turnkey ami (for VPN).Attach it to vpc-dev-env with a size of t2.micro and the SG-VPN security group.


```bash
  The End. Thank you!
```





### References:

https://docs.aws.amazon.com/vpc/latest/userguide/VPC_Internet_Gateway.html#Add_IGW_Attach_Gateway

https://docs.aws.amazon.com/IAM/latest/UserGuide/id.html

https://aws.amazon.com/getting-started/hands-on/getting-started-with-aws-management-console/

https://aws.amazon.com/blogs/architecture/

https://docs.aws.amazon.com/cli/latest/reference/autoscaling/index.html

https://docs.aws.amazon.com/elasticloadbalancing/latest/application/introduction.html

https://docs.aws.amazon.com/autoscaling/ec2/userguide/create-launch-config.html


