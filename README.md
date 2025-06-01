# 🔧 VPC Setup

## I. Prerequisites

1. Install WSL (Windows Subsystem for Linux)

    [WSL Install Guide](https://learn.microsoft.com/en-us/windows/wsl/install)

    (You will have to attach a credit card, even though you’re going to stick to free services. Using Search / AI to verify which services are free and which are paid can be your friend).

2. Create an AWS Account 

    [AWS Free Tier](https://aws.amazon.com/free)

    (You’re going to be making various commands that will require specific identifiers, and you’ll want this information handy. If you’re comfortable navigating the AWS console by yourself that can work, but the fastest way is to simply aggregate the information and stick it all somewhere that you can easily grab it from, e.g.)

3. Create a Key Info Document
Log and store:

    - VPC ID/name

    - Subnet names, CIDRs, IDs

    - Route Table ID

    - Internet Gateway ID

    - NAT instance ID

    - Security Group ID

    - Key pair name

See Example Scheme: (input your own names and IDs in place of 123, 123456789):

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**vpc name/id** :

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;my-vpc-123 | vpc-123456789

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**subnet name/cidr/ids** :

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;private-subnet-123 | 10.0.2.0/24 | subnet-123456789
&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;public-subnet-123 | 10.0.1.0/24 | subnet-123456789

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**route table name/id** :

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;my-route-table-123 | rtb-123456789

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**internet gateway name/id** :

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vpc02-internet-gateway | igw-123456789

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**nat name/instance id/state** :

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;NATInstance  | i-123456789 |  running

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**security group id** :

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;vpc02SecurityGroup | sg-123456789

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;**key pair name**:

&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;key-pair-123

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
