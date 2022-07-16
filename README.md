
# AWS complete setup through console

We will start from creating a VPC, SG, IAM role, LC, LB, AS, Code deploy, RDS, Redis, S3 bucket, CloudFront, SNS, CloudWatch, VPN, and to Elastic IP. 

We will be taking us-east-1 (N. Virginia) region to deploy ot infra and name would be dev-env.

https://docs.aws.amazon.com/awsconsolehelpdocs/latest/gsg/console-help-gsg.pdf




## Create a VPC 
Name: vpc-dev-env 

CIDR range: 172.20.0.0/16

### vpc-image

### Create Subnet
Create four subnets: one public and one private subnet in the availability zone us-east-1a, and one public and one private subnet in the availability zone us-east-1b.


Subnets CIDR-Ranges:

•	public-subnet1-dev-env = 172.20.0.0/24 us-east-1a

•	public-subnet2-dev-env = 172.20.16.0/24 us-east-1b

•	private-subnet1-dev-env = 172.20.32.0/24 us-east-1a

•	private-subnet2-dev-env =172.20.64.0/24 us-east-1b

### subnet image

Now, we create two route table, one for public (RT-public-dev-env) and other one for private (RT-private-dev-env).
Then add them in VPC and after that associate subnets in them.


### RT-public-dev-env--- image

### RT-private-dev-env-- image


Now, create an internet gateway (name: igw-dev-env) attach it with vpc-dev-env & attatch the public router table (RT-public-dev-env) with it.


Then create a NAT gateway (NAT-dev-env) in public subnet with static IP (Elastic IP) and attached with RT-private-dev-env . This is to provide internet accessibility to private server.


### igw image

### nat image


## Security Group
We may set up multiple security groups for various resources. Mention the name of the security group to distinguish it.

### Security group for load balancer (name: SG-LB)
Select vpc-dev-env and open port HTTP (80) and HTTPS (443) for inbound traffic to all.

### SG-LB image

### Security group for Redis (name: SG-Redis)
Select vpc-dev-env and and open port 6379 from Webserver- security group.

#### SG-Redis image take later and put


### Security group for VPN (name: SG-VPN)
Select vpc-dev-env and open ports 22, 80, 443, 1194, 12320, and 12321.

#### SG-VPN image

NOTE: After completing the infrastructure, we should remove ports 80, 22 and 443 from the SG-VPN.


### Security group for launch configuration(web-server) (name: SG-LC)
Select vpc-dev-env add SG-LB and SG-LB at port 80 and 443, SG-VPN at  port 22 allowed.

### sg-lc image

### Security group for database (name SG-DB)

Select vpc-dev-env and SG-LC and MYSQL/Aurora port 3306 allowed.

### sg-db image


## Create IAM Role for EC2 service (name: s3FullAccessRoleForEC2)
To provide s3 full-access to EC2

### s3FullAccessRoleForEC2 image


## Create IAM Role for CodeDeploy service (name: CodeDeployRoleforEC2)
To provice CodeDeploy access

###CodeDeployRoleforEC2 image


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

#### alb-dev-env image


## Create AutoScaling group (name: asg-dev-env)
Attach with vpc-dev-env and both private subnets. Also attach launch configuration profile
Select tg-dev-env as target group to forward the traffic to specific targets.

Here we can define Desired, Minimum and Maximum capacity.

Note: In new console, AWS has removed ASG console
### asg-dev-env image


## Create CodeDeploy 
Create an application first and select Ec2/On-premises on the compute platform.

### application-dev-env image

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



