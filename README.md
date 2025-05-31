🔧 VPC Setup: Step-by-Step Breakdown

I. Prerequisites

    Install WSL (Windows Subsystem for Linux)
    WSL Install Guide

    Create an AWS Account
    AWS Free Tier

    Create a Key Info Document
    Log and store:

        VPC ID/name

        Subnet names, CIDRs, IDs

        Route Table ID

        Internet Gateway ID

        NAT instance ID

        Security Group ID

        Key pair name

II. Create VPC Components in AWS Console

    Create a VPC

        CIDR: 10.0.0.0/16

        No IPv6

        Tenancy: Default

    Create Two Subnets

        public-subnet-01: 10.0.1.0/24

        private-subnet-01: 10.0.2.0/24

    Create an Internet Gateway

        Name it

        Attach to VPC

    Create a Route Table

        Name it

        Add route: 0.0.0.0/0 → internet gateway

        Associate with public subnet

III. Set Up CLI with Ubuntu on WSL

    Create Linux User

    Create Access Key Pair in IAM

    Run aws configure in Ubuntu CLI

        Access Key

        Secret Key

        Region

        Output: json

IV. (Optional) CLI-Based VPC Component Creation

(You already created resources in the console, but here are CLI commands for practice)

    Create VPC
    aws ec2 create-vpc --cidr-block 10.0.0.0/16

    Create Subnets
    aws ec2 create-subnet ...

    Create Internet Gateway & Attach
    aws ec2 create-internet-gateway
    aws ec2 attach-internet-gateway ...

    Create Route Table & Add Route
    aws ec2 create-route-table ...
    aws ec2 create-route ...

V. Add a Security Group via CLI

    Create Security Group
    aws ec2 create-security-group --group-name MySecurityGroup ...

    Allow SSH Access (port 22)
    aws ec2 authorize-security-group-ingress --group-id sg-xxxx --protocol tcp --port 22 --cidr 0.0.0.0/0

VI. Add NAT Functionality Using EC2

    Find a NAT-Compatible AMI

aws ec2 describe-images \
--owners amazon \
--filters "Name=name,Values=amzn2-ami-hvm-*-x86_64-gp2" "Name=state,Values=available" \
--query 'Images[*].[ImageId,Name]' --output table

Launch EC2 Instance as NAT

    Use t2.micro and public subnet

    Associate public IP

    Add security group

aws ec2 run-instances --image-id ami-xxxx \
--count 1 --instance-type t2.micro \
--key-name key-pair-xx --subnet-id subnet-xx \
--associate-public-ip-address \
--security-group-ids sg-xx \
--tag-specifications 'ResourceType=instance,Tags=[{Key=Name,Value=NATInstance}]'

Disable Source/Destination Check

aws ec2 modify-instance-attribute --instance-id i-xxxx --no-source-dest-check

SSH & Configure NAT

    SSH into NAT Instance

    Enable IP Forwarding:

sudo sysctl -w net.ipv4.ip_forward=1
echo "net.ipv4.ip_forward = 1" | sudo tee -a /etc/sysctl.conf
sudo sysctl -p

Enable NAT Masquerading:

    sudo iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    sudo yum install iptables-services -y
    sudo service iptables save
    sudo systemctl enable iptables

Update Route Table for Private Subnet

    Route outbound traffic via NAT instance:

aws ec2 create-route \
--route-table-id rtb-xxxx \
--destination-cidr-block 0.0.0.0/0 \
--instance-id i-xxxx