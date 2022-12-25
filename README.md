# 3-Tier_Architecture
How to design a 3- Tier Architecture in AWS

What is 3-tier Architecture?

The three-tier architecture is the most popular implementation of a multi-tier architecture and consists of a single presentation tier, logic tier, and data tier.

Each of these layers or tiers does a specific task and can be managed independently of each other. This a shift from the monolithic way of building an application where the frontend, the backend and the database are both sitting in one place.

Here in this section I'll be making use of the following AWS services to design and build a three-tier cloud infrastructure: Elastic Compute Cloud (EC2), Auto Scaling Group, Virtual Private Cloud(VPC), Elastic Load Balancer (ELB), Security Groups and Internet Gateway.

Using above services I'll design a highly available and fault tolerant architecture.

![image](https://user-images.githubusercontent.com/67903736/209460566-c375b640-84e3-49b5-9669-f690a89df6bb.png)


Benefits:
scalability — each item can scale horizontally, application servers can be deployed on multiple machines database integrity and security— the client cannot directly access the data with an application layer between them and the database improved performance — the presentation tier can cache requests, minimizing network utilization and load easier to maintain and modify — modifications or replacement of one tier does not affect the other tiers (decoupling), multiple developers can work on the different layers, reducing cost and time needed to integrate changes

Overview of Steps
Create a VPC, subnets, internet gateway, and edit route tables.

Create an application load balancer for the web tier (Internet-facing) and application tiers.

Create the application and web tiers with EC2 auto-scaling groups.

Configure security groups so the web tier only accepts traffic from the ALB (Application Load Balancer), and the application tier only accepts traffic from the web tier security group.

Create the database tier using RDS (free tier).

Verify the web tier can be accessed from the Internet and that it can ping the application tier.

Before we get started

To follow along, you need to have an AWS account. We shall be making use of the AWS free-tier resources so we do not incur charges while learning.

Note: At the end of this tutorial, you need to stop and delete all the resources such as the EC2 instances, Auto Scaling Group, Elastic Load Balancer etc you set up. Otherwise, you get charged for it when you keep them running for a long.

Let’s Begin
Setup the Virtual Private Cloud (VPC): VPC stands for Virtual Private Cloud (VPC). It is a virtual network where you create and manage your AWS resource in a more secure and scalable manner. Go to the VPC section of the AWS services, and click on the Create VPC button.
Give your VPC a name and a CIDR block of 10.0.0.0/16

![image](https://user-images.githubusercontent.com/67903736/209460583-84435a17-6b80-404b-83aa-69198f3ac484.png)


Create VPC

![image](https://user-images.githubusercontent.com/67903736/209460587-8727fb99-aa0c-4614-b2f5-7bd0535acccc.png)


2. Setup the Internet Gateway: The Internet Gateway allows communication between the EC2 instances in the VPC and the internet. To create the Internet Gateway, navigate to the Internet Gateways page and then click on Create internet gateway button.

![image](https://user-images.githubusercontent.com/67903736/209460598-9bb760a7-225d-430a-9eda-1cef9ae0b088.png)

![image](https://user-images.githubusercontent.com/67903736/209460605-e91063af-07c8-4ff7-92b2-e4d769c52324.png)


We need to attach our VPC to the internet gateway. To do that:

a. we select the internet gateway

b. Click on the Actions button and then select Attach to VPC.

c. Select the VPC to attach the internet gateway and click Attach

![image](https://user-images.githubusercontent.com/67903736/209460610-621f024e-fa7e-40fa-8859-c83191c208a6.png)


3. Create 4 Subnets: The subnet is a way for us to group our resources within the VPC with their IP range. A subnet can be public or private. EC2 instances within a public subnet have public IPs and can directly access the internet while those in the private subnet does not have public IPs and can only access the internet through a NAT gateway.

For our setup, we shall be creating the following subnets with the corresponding IP ranges.

· demo-public-subnet-1 | CIDR (10.0.1.0/24) | Availability Zone (us-east-1a)

· demo-public-subnet-2 | CIDR (10.0.2.0/24) | Availability Zone (us-east-1b)

· demo-private-subnet-3 | CIDR (10.0.3.0/24) | Availability Zone (us-east-1a)

· demo-private-subnet-4 | CIDR(10.0.4.0/24) | Availability Zone (us-east-1b)

create subnets

![image](https://user-images.githubusercontent.com/67903736/209460621-5389d221-f427-439f-863c-fadce7467d6a.png)

![image](https://user-images.githubusercontent.com/67903736/209460625-6c8fdf42-8db6-4543-b455-052a15c21e8a.png)


Four subnets got sreated

4. Create Two Route Tables: Route tables is a set of rule that determines how data moves within our network. We need two route tables; private route table and public route table. The public route table will define which subnets that will have direct access to the internet ( ie public subnets) while the private route table will define which subnet goes through the NAT gateway (ie private subnet).

To create route tables, navigate over to the Route Tables page and click on Create route table button.

![image](https://user-images.githubusercontent.com/67903736/209460628-c895717f-83a1-4abe-9e0f-4a7eafce39a0.png)


Route Table Created

![image](https://user-images.githubusercontent.com/67903736/209460631-7cefeb83-8713-437b-8cf4-824ac730cece.png)


Private and Public Route Tables

The public and the private subnet needs to be associated with the public and the private route table respectively.

To do that, we select the route table and then choose the Subnet Association tab.

![image](https://user-images.githubusercontent.com/67903736/209460633-1e7240c1-e5ef-4b63-bc2c-ac467f08f377.png)


Subnet Associations

![image](https://user-images.githubusercontent.com/67903736/209460638-90027b13-51e5-4de8-9490-4137931faf6e.png)


Select the public subnet for the public route table

We also need to route the traffic to the internet through the internet gateway for our public route table.

To do that we select the public route table and then choose the Routes tab. The rule should be similar to the one shown below:

![image](https://user-images.githubusercontent.com/67903736/209460647-d7d65c36-edfa-4e5c-a237-72ca37253755.png)


Edit Route for the public route table

5. Create the NAT Gateway: The NAT gateway enables the EC2 instances in the private subnet to access the internet. The NAT Gateway is an AWS managed service for the NAT instance. To create the NAT gateway, navigate to the NAT Gateways page, and then click on the Create NAT Gateway.

Please ensure that you know the Subnet ID for the demo-public-subnet-2. This will be needed when creating the NAT gateway.

![image](https://user-images.githubusercontent.com/67903736/209460648-a8bd27d2-22db-4daa-a1b6-e0f769705074.png)


Create NAT Gateway

Now that we have the NAT gateway, we are going to edit the private route table to make use of the NAT gateway to access the internet.

![image](https://user-images.githubusercontent.com/67903736/209460655-68a4222c-649d-4fb5-8ab8-62f7fd7a0fd7.png)


![image](https://user-images.githubusercontent.com/67903736/209460659-92b5bec1-843a-4d50-b1a3-f6a5787fa071.png)


Edit Private Route Table to use NAT Gateway for private EC2 instances

6. Create Elastic Load Balancer: From our architecture, our frontend tier can only accept traffic from the elastic load balancer which connects directly with the internet gateway while our backend tier will receive traffic through the internal load balancer. The essence of the load balancer is to distribute load across the EC2 instances serving that application. If however, the application is using sessions, then the application needs to be rewritten such that sessions can be stored in either the Elastic Cache or the DynamoDB. To create the two load balancers needed in our architecture, we navigate to the Load Balancer page and click on Create Load Balancer.

a. Select the Application Load Balancer.

![image](https://user-images.githubusercontent.com/67903736/209460669-044714f7-b110-4097-8b0f-ac7fb6835e86.png)


Select Application Load Balancer

b. Click on the Create button

c. Configure the Load Balancer with a name. Select internet facing for the load balancer that we will use to communicate with the frontend and internal for the one we will use for our backend.

![image](https://user-images.githubusercontent.com/67903736/209460679-896ced3b-5007-463f-874c-2af7eb0abf20.png)


Internet Facing Load Balancer for the Frontend tier

![image](https://user-images.githubusercontent.com/67903736/209460681-ab877126-f08b-4945-ac21-3c0c181c32ef.png)


Internal Load Balancer for the Backend Tier

![image](https://user-images.githubusercontent.com/67903736/209460683-e54da3db-1784-4b24-8934-27cc11da51bf.png)


d. Under the Availability Zone, for the internet facing Load Balancer, we will select the two public subnets while for our internal Load Balancer, we will select the two private subnet.

Availability Zone for the Internet Facing Load Balancer

![image](https://user-images.githubusercontent.com/67903736/209460694-951f0b36-22b1-4cc0-b4b9-93976b9a0fde.png)


Availability Zone for the internal Load Balancer

![image](https://user-images.githubusercontent.com/67903736/209460697-9678391f-9266-43af-938e-a473a377cd4c.png)


e. Under the Security Group, we only need to allow ports that the application needs. For instance, we need to allow HTTP port 80 and/or HTTPS port 443 on our internet facing load balancer. For the internal load balancer, we only open the port that the backend runs on (eg: port 3000) and the make such port only open to the security group of the frontend. This will allow only the frontend to have access to that port within our architecture.

f. Under the Configure Routing, we need to configure our Target Group to have the Target type of instance. We will give the Target Group a name that will enable us to identify it. This is will be needed when we will create our Auto Scaling Group. For example, we can name the Target Group of our frontend to be Demo-Frontend-TG

Skip the Register Targets and then go ahead and review the configuration and then click on the Create button.

7. Auto Scaling Group: We can simply create like two EC2 instances and directly attach these EC2 instances to our load balancer. The problem with that is that our application will no longer scale to accommodate traffic or shrink when there is no traffic to save cost. With Auto Scaling Group, we can achieve this feat. Auto Scaling Group is can automatically adjust the size of the EC2 instances serving the application based on need. This is what makes it a good approach rather than directly attaching the EC2 instances to the load balancer.

To create an Auto Scaling Group, navigate to the Auto Scaling Group page, Click on the Create Auto Scaling Group button.

a. Auto Scaling Group needs to have a common configuration that instances within it MUST have. This common configuration is made possible with the help of the Launch Configuration. In our Launch configuration, under the Choose AMI, the best practice is to choose the AMI which contains the application and its dependencies bundled together. You can also create your custom AMI in AWS.

Custom AMI for each tier of our application

![image](https://user-images.githubusercontent.com/67903736/209460703-cf21510c-52e3-40d9-bc26-0b91ee455a94.png)


b. Choose the appropriate instance type. For a demo, I recommend you choose t2.micro (free tier eligible) so that you do not incur charges.

c. Under the Configure details, give the Launch Configuration a name, eg Demo-Frontend-LC. Also, under the Advance Details dropdown, the User data is provided for you to type in a command that is needed to install dependencies and start the application.

![image](https://user-images.githubusercontent.com/67903736/209460707-a79e0c72-9f7c-4e06-8635-97943828b4e7.png)


d. Again under the security group, we want to only allow the ports that are necessary for our application.

e. Review the Configuration and Click on Create Launch Configuration button. Go ahead and create a new key pair. Ensure you download it before proceeding.

f. Now we have our Launch Configuration, we can finish up with the creating our Auto Scaling Group. Use the below image as a template for setting up yours.

Auto Scaling Group 1

![image](https://user-images.githubusercontent.com/67903736/209460715-90c71ed1-1d02-4552-ac6d-b4c9921b16f2.png)


Auto Scaling Group 2

![image](https://user-images.githubusercontent.com/67903736/209460723-2727343a-3739-4a5b-b0b7-f128a8fd4c4e.png)


g. Under the Configure scaling policies, we want to add one instance when the CPU is greater than or equal to 80% and to scale down when the CPU is less than or equal to 50%. Use the image as a template.

Scale-up

![image](https://user-images.githubusercontent.com/67903736/209460727-e2d8b50c-cfcf-4b1a-883d-194e4c6fc71e.png)


Scale Down

![image](https://user-images.githubusercontent.com/67903736/209460734-78adafbe-eaeb-4116-b2a9-218a6973e4b9.png)


h. We can now go straight to Review and then Click on the Create Auto Scaling group button. This process is to be done for both the frontend tier and the backend tier but not the data storage tier.

We have almost setup or architecture. However, we cannot SSH into the EC2 instances in the private subnet. This is because have not created our bastion host. So the last part of this article will show how to create the bastion host.

8. Bastion Host: The bastion host is just an EC2 instance that sits in the public subnet. The best practice is to only allow SSH to this instance from your trusted IP. To create a bastion host, navigate to the EC2 instance page and create an EC2 instance in the demo-public-subnet-1 subnet within our VPC. Also, ensure that it has public IP.

Bastion Host EC2 instance in public subnet

![image](https://user-images.githubusercontent.com/67903736/209460744-58f6abb5-cb4c-4aa1-8025-85b7afcdd956.png)


Security Group of the Bastion Host

![image](https://user-images.githubusercontent.com/67903736/209460749-968fd4a0-9c10-40a6-b2e0-f075e97189c2.png)


We also need to allow SSH from our private instances from the Bastion Host.

Ok, that's it !!
