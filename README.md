## Terraform Homework 1
## Description: 
#### In this homework 1 we are going to create the below infrastructure as a code using Terraform and AWS as a provider.
## Terraform Template
*  VPC
*  Internet Gateway
*  3 Public Subnets
*  3 Private Subnets
*  2 RouteTables (1 Public, 1 Private)
*  NAT Gateway
*  Elastic IP

#### The Terraform Template only has a terraform_hw1.tf file.
In the terraform_hw1.tf file you are going to find the Terraform code to create an AWS VPC, internet_gateway, 3 public subnets, 3 private subnets, 2 RouteTables (1 public and 1 private), an Elastic IP, NAT Gateway and the respective associations between the resources.

Just to show some code from the terraform_hw1.tf file. 

In the code below we have the provider in this case AWS, we are also adding Oregon as the region that we are using for this project (us-west-2).

```bash
#Adding the provider and version
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}

# Configure the AWS Provider/Region
provider "aws" {
  region = "us-west-2"
}
```

The AWS VPC can be created using the code below, "terraform-vpc" is going to be used as the VPC id, can be any other name. Make sure to add also the cidr_block, tags can be added as well to easly identify the VPC in the AWS account.
```bash
# Create a VPC using the cidir_block 172.30.0.0/16
resource "aws_vpc" "terraform-vpc" {
  cidr_block ="172.30.0.0/16"
  instance_tenancy ="default"

  tags = {
    "Name" = "israelalonzo-terraform-vpc"
  }
}
```

Using the code below an Internet Gateway can be created, make sure to include the vpc_id to associate the Internet Gateway to the VPC as shown below.
```bash
#Code to Create a Internet Gateway
resource "aws_internet_gateway" "terraform-internet-gw" {
  vpc_id = aws_vpc.terraform-vpc.id

  tags = {
    "Name" = "israelalonzo-terraform-internet-gw"
  }
}
```

Command to show how to create an AWS subnet. When creating subnets is recommended to add tags to identify easily a subnet cause most of the time we need to create more than 1 or 2 subnets, for example here we are using 6 subnets, 3 public subnets, and 3 privates subnets,  is recommended to create the subnets in different availability zones if possible, this helps to have a high availability infrastructure, using other resources too but this is a part of it. In the below subnet we are using availability zone "us-west-2a", for the public subnet 2 "us-west-2b" can be use, there is also "us-west-2c" and "us-west-2d", take in consideration that the Oregon region "us-west-2" has 4 availability zones.

```bash
#Create israelalonzo-terraform-public-subnet-1
resource "aws_subnet" "terraform-public-subnet-1" {
  vpc_id                  = aws_vpc.terraform-vpc.id
  cidr_block              = "172.30.0.0/24"
  availability_zone       = "us-west-2a"
  map_public_ip_on_launch = true

  tags = {
    "Name" = "israelalonzo-terraform-public-subnet-1"
  }
}


```

Below the Terraform code to create an AWS Route Table.
Make sure to associte the Route Table to the VPC_id and based that this is the Public Route Table, please associate it to the Internet Gateway as described below, the route cidr_block leave it as "0.0.0.0/0" to allow IPv4 communication between all the EC2 instances for example the are inside the VPC.
```bash
#Create israelalonzo-terraform-public-route-table-1
resource "aws_route_table" "terraform-public-route-table-1" {
  vpc_id = aws_vpc.terraform-vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    gateway_id = aws_internet_gateway.terraform-internet-gw.id

  }

  tags = {
    Name = "israelalonzo-terraform-public-route-table-1"
  }
}

```
This is how we associate a Subnet to a Route Table, here we have to add the id of the Public Route Table and the id of the Public Subnet. Same process apply to associate Private Route Table to Private Subnets as well.
```bash
#Create israelalonzo-terraform-public-route-table-1 with israelalonzo-terraform-public-subnet-1
resource "aws_route_table_association" "terraform-public-route-table-1-with-public-subnet-1" {
  subnet_id      = aws_subnet.terraform-public-subnet-1.id
  route_table_id = aws_route_table.terraform-public-route-table-1.id

}
```

The code below is used to create an Elastic IP that is required to create a NAT Gateway.
```bash
#Allocate Elastic IP Address
resource "aws_eip" "terraform-eip-for-nat-gw" {
  vpc = true
  tags = {
    "Name" = "israelalonzo-terraform-eip-for-nat-gw"
  }
}

```
To create a NAT Gateway, use the below code, this NAT Gateway needs to be associated to the Private Route Table and to one of the Public Subnets, cuase this NAT Gateway is going to allow the EC2 instances for example to be able to communicate to the Internet. As shown below is associated with the Public Subnet 1, is also associated to an Elastic IP.
```bash
#Create a NAT Gateway
resource "aws_nat_gateway" "terraform-nat-gw" {
  allocation_id = aws_eip.terraform-eip-for-nat-gw.id
  subnet_id     = aws_subnet.terraform-public-subnet-1.id
  depends_on    = [aws_internet_gateway.terraform-internet-gw]
}

```
And last with the code below we associate the Private Route Table to the NAT Gateway, this communication will help to allow sharing information between public an private subnets, communication like SSH communication, Internet communication from the EC2 instances that are going to be created inside the private subnets.
```bash
#Create israelalonzo-terraform-private-route-table-1
resource "aws_route_table" "terraform-private-route-table-1" {
  vpc_id = aws_vpc.terraform-vpc.id

  route {
    cidr_block = "0.0.0.0/0"
    nat_gateway_id = aws_nat_gateway.terraform-nat-gw.id
  }

  tags = {
    Name = "israelalonzo-terraform-private-route-table-1"
  }
}

```

### Finally to test the Terraform Template use the below commands.
#
While inside the project directory "terraform_hw1.tf" file:

Type terraform init, terraform init command is used to initialize a working directory containing Terraform configuration files.
```bash
terraform init
```
Once the terraform init is completed, then Type terraform apply, terraform apply command executes the actions proposed, in this case action to create the AWS infrastructure based on the Terraform Template.

```bash
terraform apply
```

And last if you would like to destroy the AWS infrastructure you just created, run terraform destroy.

```bash
terraform destroy
```

## Below more information about this project
Steps that can help to prepare the host machine to be able to run the Terraform Template.

Step 1: Download the Terraform package for the OS(this terraform code was tested in Ubuntu-20.04.4-amd64 64-bit) in the following link: https://www.terraform.io/downloads

Step 2: Unzip the downloaded terraform package(The terraform package will look like "terraform_1.1.9_linux_amd64.zip", unzip it).

Step 3: Once the package is unzipped "terraform_1.1.9_linux_amd64", open it and copy the binary called "terraform", then paste the binary called "terraform" in the host machine path /usr/local/bin.

Step 4: Open the host machine Linux terminal and type $terraform --version (it will show the terraform version).

```bash
terraform --version
```

Step 5: Open the host machine Linux terminal and type $aws configure(enter the AWS_ACCESS_KEY_ID, then the AWS_SECRET_ACCESS_KEY, hit enter if the region is correct otherwise type(us-west-2), then hit enter again for the output format(these steps will help to test that access to the AWS account is allow, cause to create an aws vpc and the rest of the vpc resources access to the aws account with the right permissions to create resources is required, otherwise create your AWS account with these permissions).

```bash
aws configure
```

Step 6: Clone the terraform_hw2 to your local machine.

```bash
git clone https://github.com/IsraelAlonzoMoran/terraform_hw1.git

```
Step 7: Once the "terraform_hw1" project is downloaded to the host machine, with $cd go to the directory that has the downloaded folder, example "$cd /home/alonzo/Downloads/terraform_hw1".

Step 8: Once in the directory that has the "terraform_hw1.tf" file:

Type terraform init, terraform init command is used to initialize a working directory containing Terraform configuration files
```bash
terraform init
```
Once the terraform init is completed, then Type terraform apply, terraform apply command executes the actions proposed, in this case action to create the AWS infrastructure based on the Terraform Template.

```bash
terraform apply
```

And last if you would like to destroy the AWS infrastructure you just created, run terraform destroy.

```bash
terraform destroy
```

Step 9: Go to the AWS Console(https://aws.amazon.com/) select Sing in, enter your credentinals, type vpc in the search option and review that the aws vpc and rest of resources have been created.

That's all. Thank you.
