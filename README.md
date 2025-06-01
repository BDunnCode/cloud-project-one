# 🔧 VPC Setup

## I. Prerequisites

*Install WSL (Windows Subsystem for Linux)*

[WSL Install Guide](https://learn.microsoft.com/en-us/windows/wsl/install)

(You will have to attach a credit card, even though you’re going to stick to free services. Using> > Search / AI to verify which services are free and which are paid can be your friend).

*Create an AWS Account*

[AWS Free Tier](https://aws.amazon.com/free)

(You’re going to be making various commands that will require specific identifiers, and you’ll want this information handy. If you’re comfortable navigating the AWS console by yourself that can work, but the fastest way is to simply aggregate the information and stick it all somewhere that you can easily grab it from, e.g.)

*Create an easily accessed document with this information:*
- Log and store:
    - VPC ID/name
    - Subnet names, CIDRs, IDs
    - Route Table ID
    - Internet Gateway ID
    - NAT instance ID
    - Security Group ID
    - Key pair name

Example Schema: (input your own names and IDs in place of 123, 123456789):

**vpc name/id** :

my-vpc-123 | vpc-123456789

**subnet name/cidr/ids** :

private-subnet-123 | 10.0.2.0/24 | subnet-123456789
public-subnet-123 | 10.0.1.0/24 | subnet-123456789

**route table name/id** :

my-route-table-123 | rtb-123456789

**internet gateway name/id** :

vpc02-internet-gateway | igw-123456789

**nat name/instance id/state** :

NATInstance  | i-123456789 |  running

**security group id** :

vpc02SecurityGroup | sg-123456789

**key pair name**:

key-pair-123

## 2. Building your VPC Components In the AWS Console

Once you’ve installed Windows Subsystem for Linux and created your AWS account, it’s time to sign-in to the AWS console.

### Create your VPC
- In the search bar, type “VPC”. Double-click on the service when it pops up.
- Select “VPC Only” 
- For the name tag, you can name it whatever you want. I called mine the default “my-vpc-01”
- For “IPv4 CIDR”, input “10.0.0.0/16” (There are other options, but this will work).
- No “IPv6 CIDR block”
- Tenancy “default”
- No tags are required.
- Press “Create VPC”

Congratulations, you’ve built your VPC. Now we’ll segment the network using subnets. 

### Create Subnets

*Subnet 1*:  

**Subnet Name**: public-subnet-01 (or whatever matches your naming scheme)  
**Availability Zone**: select any from the list  
**CIDR Block**: 10.0.1.0/24  
Left-click “Create Subnet”  

*Subnet 2*:  

**Subnet Name**: private-subnet-01 (or whatever matches your naming scheme)  
**Availability Zone**: select any from the list  
**CIDR Block**: 10.0.2.0/24  
Left-click “Create Subnet”  

With the subnets created, you’re now ready to add a gateway and route tables.  

### Create an Internet Gateway  
- Navigate to Internet Gateways  
- Select “Create Internet Gateway”  
- Name the gateway based on your naming scheme  
- Attach the gateway to your VPC  

### Create Route Tables
*Creating the Table*:  
- Select “Route tables” in the navigation bar  
- Press “Create Route Table”  
- Name it “my-route-table-01” (or whatever matches your naming scheme)  
- Press “Create”  

*Add a route for your public subnet*:
- In the Route Tables tab, select the route table you just created
- Select “Routes” -> “Edit routes” -> “Add route” 
- Add 0.0.0.0 to add your internet gateway

*Attach routing table to your public subnet*:
- Select the route table -> Subnet Associations -> Edit subnet associations
- Check the Public-Subnet and save.

## 3. Interacting with your VPC using the Linux CLI

Once you have installed the Windows Subsystem for Linux, you should have a command line interface tool that will appear to you as “Ubuntu”  

## Prerequisites

### Start the CLI  

- Type “Ubuntu” in the search bar
- Right click the application and select “Run as administrator”
- You are now ready to begin utilizing the command line interface for scripting and automation. This will dramatically increase the speed and ease with which you can work with your virtual private cloud. 

### Create your default Linux User

The CLI should simply ask you to define a username and password.

### Create an Access Key Pair within your AWS account using the AWS console

- Sign in to the AWS Management Console as an administrator
- Search “IAM” and left-click to select it
- In the left navigation pane, click on “Users.”
- Click on the username of the IAM user you want to create keys for.
- Go to the “Security credentials” tab.
- Scroll down to the “Access keys” section.
- Click “Create access key.”
- Choose the use case (e.g., "Command Line Interface (CLI)", etc.) and click Next.
- On the final screen, click “Create access key.”
- Copy or download the Access Key ID and Secret Access Key — you’ll only see the secret key once.

### Configuring your CLI

*In the Ubuntu CLI*:

- Type “aws configure”  

You will be prompted to enter:
- Your Access Key ID 
- Secret Key 
- AWS Region (You should be able to find this in the top right corner of your AWS console) 
- Default Output Format (Use “json”) 

## Interacting with AWS using the CLI

With the Ubuntu CLI configured, you can now perform all of the steps that you performed under the “Building your VPC” section using commands, ex:

**Create VPC**:

**Create a public subnet**:

```bash
aws ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.1.0/24 --availability-zone us-west-2a
```

**Create a private subnet**:

```bash
aws ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.2.0/24 --availability-zone us-west-2b
```

**Create and Attach an Internet Gateway**:

```bash
aws ec2 create-internet-gateway
```

```bash
aws ec2 attach-internet-gateway --vpc-id vpc-xxxxxx --internet-gateway-id igw-xxxxxx
```

**Create a route table for the public subnet**:


```bash
aws ec2 create-route-table --vpc-id vpc-xxxxxx
```


**Add a route for the public subnet**:

```bash
aws ec2 create-route --route-table-id rtb-xxxxxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxxxxx
```




You won’t need to do these for this project, however, because you’ve already set them up. Now we will make actual changes to the VPC using the CLI.


