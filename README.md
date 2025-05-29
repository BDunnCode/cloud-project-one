# Custom AWS VPC Network from Scratch (Manual CLI Setup + Diagram)

## Project Overview

An AWS VPC complete with step-by-step instructions on the cost/budget management and building out process of public and private subnets, routing tables, security groups, and a NAT instance to mask the IPs of sensitive hosts in the private subnet when they communicate with the public internet.

## Architecture Diagram

[VPC Architecture Text Diagram](./images/vpc-text-diagram.png)


[VPC Architecture Visual Diagram](./images/vpc-visual-diagram.png)

## Step-by-Step Setup

### Prerequisites 

-Download and install Windows subsystem for Linux 

https://learn.microsoft.com/en-us/windows/wsl/install

-Create an AWS account

https://aws.amazon.com/free

(You will have to attach a credit card, even though you’re going to stick to free services. Using Search / AI to verify which services are free and which are paid can be your friend).

-Create a document to store key information

You’re going to be making various commands that will require specific identifiers, and you’ll want this information handy. If you’re comfortable navigating the AWS console by yourself that can work, but the fastest way is to simply aggregate the information and stick it all somewhere that you can easily grab it from, e.g. 

vpc name/id :

my-vpc-xx | vpc-xxxxxxxxxx

subnet name/cidr/ids :

private-subnet-xx | X.X.X.X/XX | subnet-xxxxxxxxxx
public-subnet-xx | X.X.X.X/XX | subnet-xxxxxxxxxx

route table name/id :

my-route-table-xx | rtb-xxxxxxxxxx

internet gateway name/id :

xx-internet-gateway | igw-xxxxxxxxxx

nat name/instance id/state :

NATInstance  | i-xxxxxxxxxx |  running

security group id :

vpc02SecurityGroup | sg-xxxxxxxxxx

key pair name:

key-pair-xx

### Building your VPC

Once you’ve installed Windows Subsystem for Linux and created your AWS account, it’s time to sign-in to the AWS console.

Create your VPC
In the search bar, type “VPC”. Double-click on the service when it pops up.
Select “VPC Only” 
For the name tag, you can name it whatever you want. I called mine the default “my-vpc-01”
For “IPv4 CIDR”, input “10.0.0.0/16” (There are other options, but this will work).
No “IPv6 CIDR block”
Tenancy “default”
No tags are required.
	Press “Create VPC”

Congratulations, you’ve built your VPC. Now we’ll segment the network using subnets. 

### Create Subnets
	
1) Subnet 1:

Subnet Name: public-subnet-01 (or whatever matches your naming scheme)
Availability Zone: select any from the list
CIDR Block: 10.0.1.0/24
Left-click “Create Subnet”

2) Subnet 2: 

Subnet Name: private-subnet-01 (or whatever matches your naming scheme)
Availability Zone: select any from the list
CIDR Block: 10.0.2.0/24
Left-click “Create Subnet”

With the subnets created, you’re now ready to add route tables.

3) Create an Internet Gateway

Navigate to Internet Gateways
Select “Create Internet Gateway”
Name the gateway based on your naming scheme
Attach the gateway to your VPC

4) Create Route Tables

Creating the Table:
Select “Route tables” in the navigation bar
Press “Create Route Table”
Name it “my-route-table-01” (or whatever matches your naming scheme)
Press “Create”

Add a route for your public subnet:
In the Route Tables tab, select the route table you just created.
Select “Routes” -> “Edit routes” -> “Add route” 
Add 0.0.0.0 to add your internet gateway

Attach routing table to your public subnet:
Select the route table -> Subnet Associations -> Edit subnet associations.
Check the Public-Subnet and save.
	
### Interacting with your VPC using the Linux CLI

Once you have installed the Windows Subsystem for Linux, you should have a command line interface tool that will appear to you as “Ubuntu”

Type “Ubuntu” in the search bar

Right click the application and select “Run as administrator”

You are now ready to begin utilizing the command line interface for scripting and automation. This will dramatically increase the speed and ease with which you can work with your virtual private cloud. 

Prerequisites

Create your default Linux User

The CLI should simply ask you to define a username and password.

Create an Access Key Pair within your AWS account using the AWS console

1) Sign in to the AWS Management Console as an administrator

2) Search “IAM” and left-click to select it

3) In the left navigation pane, click on “Users.”

4) Click on the username of the IAM user you want to create keys for.

5) Go to the “Security credentials” tab.

6) Scroll down to the “Access keys” section.

7) Click “Create access key.”

8) Choose the use case (e.g., "Command Line Interface (CLI)", etc.) and click Next.

9) On the final screen, click “Create access key.”

10) Copy or download the Access Key ID and Secret Access Key — you’ll only see the secret key once.

### Configuring your CLI

In the Ubuntu CLI:

Type “aws configure” 

You will be prompted to enter:

Your Access Key ID 

Secret Key 

AWS Region (You should be able to find this in the top right corner of your AWS console) 

Default Output Format (Use “json”) 

Interacting with AWS using your CLI:

You could now perform all of the steps that you performed under the “Building your VPC” section using commands, ex:

Create VPC:

	aws ec2 create-vpc --cidr-block 10.0.0.0/16 --region us-west-2

	Create a public Subnet:

	aws ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.1.0/24 --availability-zone us-west-2a

Create a private Subnet:

aws ec2 create-subnet --vpc-id vpc-xxxxxx --cidr-block 10.0.2.0/24 --availability-zone us-west-2b

Create and Attach an Internet Gateway:

aws ec2 create-internet-gateway
aws ec2 attach-internet-gateway --vpc-id vpc-xxxxxx --internet-gateway-id igw-xxxxxx

Creating a Route Table
	
	Create a route table for the public subnet:

	aws ec2 create-route-table --vpc-id vpc-xxxxxx

	Add a route for the public subnet:

	aws ec2 create-route --route-table-id rtb-xxxxxx --destination-cidr-block 0.0.0.0/0 --gateway-id igw-xxxxxx

	—
You won’t need to do these for this project, however, because you’ve already set them up. Now we will make actual changes to the VPC using the CLI.
	—
	Adding to the VPC using the CLI

	We’ll now add a Security Group (AWS version of a virtual firewall) to our VPC.

	Input (into your CLI): 

“aws ec2 create-security-group --group-name MySecurityGroup --description "Allow SSH" --vpc-id vpc-xxxxxx”

You can optionally change the orange highlighted portion to another name to match the naming scheme that you’ve used throughout the project.

Change the “xxxxxx” yellow highlighted portion to VPC id (Can be found in the console by selecting “Your VPCs”)

	and 

	“aws ec2 authorize-security-group-ingress --group-id sg-xxxxxx --protocol tcp --port 22 --cidr 0.0.0.0/0”

Again change the “xxxxxx” portion to your security group id. That id should have been output to you by the CLI after you created it. If not, you can search “Security Groups” in the console and find it there.




Adding EC2 instances and NAT functionality to your VPC
Now we’ll add Network Address Translation functionality to our VPC, all while in the free tier, by using a free EC2 instance. By using Network Address Translation, we allow sensitive and private parts of the network to be able to interact with the internet without being identified by its real IP address. 

Here’s why we’re doing this:


Example: Billing Department Accessing a Third-Party Payment Gateway

Business Context:
A large corporation’s billing department needs to interact with an external payment gateway API to verify customer payment status and process transactions.

Challenge:
The billing department’s internal systems contain sensitive customer data.


For security and compliance, the company does not want the external payment gateway to see the internal IP addresses or direct source of the requests.


Also, the internal IP scheme is private and non-routable on the internet.


The company wants to limit inbound exposure of their internal network while still enabling outbound communication.



How NAT Helps:
Internal Billing System:
 The billing department’s application server has a private IP (e.g., 10.20.30.5) on the company’s internal network.


Outbound Request:
 When the billing system sends API requests to the payment gateway over the internet, the request first goes through a NAT gateway or firewall device at the company’s network edge.


NAT Translation:
 The NAT gateway replaces the internal private IP (10.20.30.5) with a public IP address assigned to the company (e.g., 198.51.100.10) before forwarding the request to the payment gateway.


Payment Gateway Sees Only the Public IP:
 The payment gateway’s servers only see requests coming from 198.51.100.10 — the public IP of the company’s NAT device — not the internal IP or any details about the originating billing system.


Security & Privacy:


This prevents the payment gateway from identifying the exact internal source machine, improving privacy.


It also prevents inbound unsolicited traffic from reaching internal systems since NAT only allows established connections.


Return Traffic:
 Responses from the payment gateway go back to the NAT device at 198.51.100.10, which translates the IP back to the internal billing system’s IP, ensuring seamless communication.



Summary
Role
Internal Network IP
Public IP (NAT)
External Server Sees
Billing System
10.20.30.5 (private)
198.51.100.10 (public)
198.51.100.10 (company NAT)

: Billing Department Accessing a Third-Party Payment Gateway



Now that we know why we’re setting up the NAT Instance to use network address translation, let’s get to it:



Building the NAT Instance
AWS doesn’t maintain official NAT instance AMIs anymore, so you have to configure one manually using a traditional EC2 instance.

Step 1 - Find an AMI
Input:
-
aws ec2 describe-images \
  --owners amazon \
  --filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
  --query 'Images[*].[ImageId,Name]' --output table
-
Grab the id of the top most AMI (This is the latest version),
Step 2 - Launch the NAT Instance
You’ll need various information about the components of your VPC so grab your file from the prerequisites and make sure to enable: 
Input: 
-
aws ec2 run-instances \
  --image-id ami-0abcdef1234567890 \
  --count 1 \
  --instance-type t2.micro \
  --key-name my-key-pair \
  --subnet-id subnet-xxxxxxxx \
  --associate-public-ip-address \
  --security-group-ids sg-xxxxxxxx \
  --tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=NATInstance}]'
-
Be sure to fill in the key pair, subnet id, and security group with your specific credentials
Step 3 - Disable Source Destination Check
- 
Input:
-
aws ec2 modify-instance-attribute \
  --instance-id i-xxxxxxxxxxxx \
  --no-source-dest-check
-
This will allow the NAT instance and functionality to work as intended
-
Step 4 - Connect to the Instance and Setup IP Forwarding
a) Get the public IP of (what will be) your NAT instance
Input:
-
aws ec2 describe-instances \
  --instance-ids i-0123456789abcdef0 \
  --query "Reservations[0].Instances[0].PublicIpAddress" \
  --output text
-
Make sure you swap in your instance ID in the top line
-
b) Make sure your key file has the proper permissions
Input:
-
chmod 400 /path/to/your-key.pem
-
c) SSH into the instance
Input:
-
ssh -i /path/to/your-key.pem ec2-user@<public-ip>
-
Be sure to put the correct full path for the key file, and swap in the public IP for what will be the NAT instance
-
d) Setup IP Forwarding
Input:
-
sudo sysctl -w net.ipv4.ip_forward=1
-
Then to make that change permanent: 
Input: 
-
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p
-
e) Enable NAT masquerading with IPTables
Input: 
-
sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
-
Make that permanent by inputting the following
Input: 
-
sudo yum install iptables-services -y
sudo service iptables save
sudo systemctl enable iptables
-
Step 5 - Update the Private Subnet's Route Table
Now we’ll route outbound internet traffic (0.0.0.0/0) to the NAT instance’s private IP (not public IP).

Input: 
-
aws ec2 create-route \
  --route-table-id rtb-xxxxxxxx \
  --destination-cidr-block 0.0.0.0/0 \
  --instance-id i-xxxxxxxxxxxx
-
Design Rationale & Cost Saving

The intent with this project was to immediately jump in and create a functioning VPC that would resemble a professional network.

Subnetting / NAT Instance 

The VPC was divided into a private and public subnet in order to emulate a modern, professional networking environment. 

The NAT Instance resides on the public while accepting traffic from EC2 instances on the private subnet and translating the IP addresses on what would be potentially sensitive host units (data, financials) before allowing them to communicate with the internet.
t2.micro Instances 

t2.micro instances were used exclusively allowing free hands-on usage of VM instances. Both for the host units and NAT. 

The NAT instance required manual configuration, and was used instead of the standard NAT Gateway, an AWS managed service that is a paid service.

Billing / Budget

Billing and Cost Management and Budget tools were utilized to avoid and monitor for any charges due to potential human error.

## Post Project Reflection

Setting up the VPC manually for the first time was surprisingly quick and straightforward. The entire process—from initial setup to gaining a working SSH connection into the EC2 instance was very easy when compared to setting up communication via documentation and diagrams.

While working alongside an AI LLM provided valuable support, it also introduced a challenge: keeping track of the many back-and-forth prompts and instructions. Ensuring coherence in the workflow required extra effort, especially as the process stretched across multiple working sessions. There are also hurdles in terms of finding the balance of utilizing the LLM to assist in productivity, while still maintaining a genuine understanding of the process.
