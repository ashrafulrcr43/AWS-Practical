# How to start Terraform on AWS provider 
# Fast install Terraform and AWS Cli 
# Check Version 
 
# 1. making provider 
provider "aws" {
  region     = "ap-south-1"
  access_key = ""
  secret_key = ""
}
# 2. Run Comment To Connect AWS 
   1. terrafrom init
   2. terraform validate

# 3. Making Resources Create Ec2 instance 
<pre>
resource "aws_instance" "my_ec2_instance" {
  ami           = "ami-021a584b49225376d" 
  instance_type = "t2.medium"
  key_name      = "terraform" # key_name like .ppk or .pem file
  tags = {
    Name = "MyTerraformEC2"
  }
}
</pre>
# Check All Paln
terraform plan
# Apply All 
terraform apply

# 4. Multipule Instance Create 
<pre>
resource "aws_instance" "my_ec2_instance" {
  count         = 3
  ami           = "ami-021a584b49225376d" 
  instance_type = "t2.medium"
  key_name      = "terraform"
  tags = {
    Name = "Server-${count.index + 1}"
  }
}
 </pre>
# 5. Method 2 to connect terrafrom to AWS on Cli
<pre>
provider "aws" {
  region     = "ap-south-1"
}
</pre>
<pre>
resource "aws_instance" "my_ec2_instance" {
  ami           = "ami-021a584b49225376d" 
  instance_type = "t2.medium"
  key_name      = "terraform" # key_name like .ppk or .pem file
  tags = {
    Name = "MyTerraformEC2"
  }
}
 </pre>
# Run comment to connect AWS 
aws configure
# Show output public ip example 
<pre>
output "instance_public_ip" {
  value = aws_instance.my_ec2_instance.public_ip
}
 </pre>
 
# full code to show output
<pre>
provider "aws" {
  region     = "ap-south-1"
}


resource "aws_instance" "my_ec2_instance" {
  ami           = "ami-021a584b49225376d" 
  instance_type = "t2.medium"
  key_name      = "terraform" # key_name like .ppk or .pem file
  tags = {
    Name = "MyTerraformEC2"
  }
}

output "instance_public_ip" {
  value = aws_instance.my_ec2_instance.public_ip
}
 </pre>
# Terraform Variable Declaration #
variables.tf
<pre>
variable "region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "ap-south-1"
}

variable "ami" {
  description = "AMI ID for EC2"
    default     = "ami-021a584b49225376d" 
   type        = string
}
variable "instance_type" {
  description = "Type of EC2 instance"
  type        = string
  default     = "t3.micro"
  
}
variable "counting" {
  description = "Number of EC2 instances to create"
  type        = number
  default     = 3
  
}
variable "key_name" {
  description = "Name of the key pair to use for the instance"
  type        = string
  default     = "terraform"
  
}

variable "access_key" {
  description = "AWS Access Key"
  type        = string
  sensitive   = true
}

variable "secret_key" {
  description = "AWS Secret Key"
  type        = string
  sensitive   = true
}
</pre>

# making file store access-key and secret-key
terraform.tfvars
<pre>
access_key = ""
secret_key = ""
</pre>

# main.tf 
<pre>
provider "aws" {
  region = var.region
    access_key = var.access_key
    secret_key = var.secret_key
}

resource "aws_instance" "my_ec2_instance" {
   count = var.counting
   ami           = var.ami
  instance_type = var.instance_type
  key_name      = "" 
  tags = {
    Name = "MyTerra-variables"
  }
}
 </pre>
 
## Complete terraform code for Anisable 
<pre>
terraform {
  required_version = ">= 1.4.0"
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 6.0"
    }
  }
}

provider "aws" {
  region     = var.region
  access_key = var.access_key
  secret_key = var.secret_key
}

data "aws_vpc" "default" {
  default = true
}

data "aws_subnets" "default" {
  filter {
    name   = "vpc-id"
    values = [data.aws_vpc.default.id]
  }
}

data "aws_ami" "ubuntu_2204" {
  most_recent = true
  owners      = ["099720109477"] # Canonical

  filter {
    name   = "name"
    values = ["ubuntu/images/hvm-ssd/ubuntu-jammy-22.04-amd64-server-*"]
  }

  filter {
    name   = "virtualization-type"
    values = ["hvm"]
  }
}

resource "aws_security_group" "web_sg" {
  name        = "${var.project}-sg"
  description = "Allow HTTP and SSH"
  vpc_id      = data.aws_vpc.default.id

  ingress {
    description = "SSH"
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [var.ssh_ingress_cidr]
  }

  ingress {
    description = "HTTP"
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "${var.project}-sg"
  }
}

resource "aws_instance" "web" {
  ami           = data.aws_ami.ubuntu_2204.id
  instance_type = var.instance_type
  subnet_id     = element(data.aws_subnets.default.ids, 0)
  vpc_security_group_ids = [aws_security_group.web_sg.id]

  associate_public_ip_address = true
  key_name = var.key_name

  tags = {
    Name = var.project
  }
}

output "public_ip" {
  description = "Public IP of EC2"
  value       = aws_instance.web.public_ip
}

output "public_dns" {
  description = "Public DNS of EC2"
  value       = aws_instance.web.public_dns
}

 </pre>
# variables 
<pre>
variable "region" {
  description = "AWS region to deploy resources"
  type        = string
  default     = "ap-south-1"
}

variable "ami" {
  description = "AMI ID for EC2"
   default     = "ami-021a584b49225376d" 
   type        = string
}
variable "instance_type" {
  description = "Type of EC2 instance"
  type        = string
  default     = "t3.micro"
  
}
# variable "counting" {
#   description = "Number of EC2 instances to create"
#   type        = number
#   default     = 3
  
# }
variable "key_name" {
  description = "Name of the key pair to use for the instance"
  type        = string
  default     = "terraform"
  
}

variable "access_key" {
  description = "AWS Access Key"
  type        = string
  sensitive   = true
}

variable "secret_key" {
  description = "AWS Secret Key"
  type        = string
  sensitive   = true
}
variable "ssh_ingress_cidr" {
  description = "CIDR allowed to SSH"
  type        = string
  default     = "0.0.0.0/0"
}

variable "project" {
  description = "Tag/name prefix"
  type        = string
  default     = "next-japan-ec2"
}
</pre>

# terraform.tfvars
<pre>
access_key = "AKIAVKT22COW34QHQWUC"
secret_key = "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"
</pre>




