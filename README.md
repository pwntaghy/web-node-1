terraform {  
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 5.0"
    }
  }
}

provider "aws" {
  region     = "us-east-1"
  # It's recommended to use IAM roles or AWS CLI configuration instead of hardcoding access keys.
  access_key = "AKIAZBKNEFV4TO4YRBXI"
  secret_key = "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"
}

# Create a VPC
resource "aws_vpc" "web_vpc" {
  cidr_block = "172.31.0.0/16"

  tags = {
    Name = "web-vpc"
  }
}

# Create a public subnet
resource "aws_subnet" "web_subnet" {
  vpc_id            = aws_vpc.web_vpc.id
  cidr_block        = "172.31.1.0/24"
  availability_zone = "us-east-1a"

  tags = {
    Name = "web-subnet"
  }
}

# Create a security group
resource "aws_security_group" "web_security_group" {
  vpc_id = aws_vpc.web_vpc.id

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]  # Allows SSH from anywhere (change for production)
  }

  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]  # Allows all outbound traffic
  }

  tags = {
    Name = "web-security-group"
  }
}

# Create a key pair for SSH access
resource "tls_private_key" "web_key" {
  algorithm = "RSA"
  rsa_bits  = 4096
}

# Create an EC2 instance
resource "aws_instance" "web_instance" {
  ami                    = "ami-0866a3c8686eaeeba"  # Ensure this AMI ID is valid in the us-east-1 region
  instance_type          = "t2.micro"
  subnet_id              = aws_subnet.web_subnet.id  # Use the created subnet reference
  key_name               = "web-key"  # Ensure this key pair exists in your account

  vpc_security_group_ids = [aws_security_group.web_security_group.id]  # Associate the security group

  tags = {
    Name = "webnitz"
  }
}

# Outputs
output "instance_public_ip" {
  value = aws_instance.web_instance.public_ip
}

output "ssh_private_key" {
  value     = tls_private_key.web_key.private_key_pem
  sensitive = true
}
