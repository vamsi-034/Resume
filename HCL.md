# HCL Terraform Code to create EC2 instance and Security group
### **The following Terraform code sets up an AWS EC2 instance with Docker and Jenkins installed on it. It also creates a security group where we will define the incoming and outgoing traffics needed**

## PROVIDER CONFIGURATION

```
  provider "aws" {
  region     = "us-east-1"
  access_key = "USER ACCESS KEY"
  secret_key = "USER SECRET KEY"
}
```
**Provider Configuration Overview**
* `provider "aws"` Configures that the AWS is provider.
* `region= "us-east-1"` specifies the region to create the EC2 Instance, This can be changed based on the user preference.
* `access_key` and `secret_key` are used to provide authentication.

## EC2 Instance Resource

```
  resource "aws_instance" "ec2_with_docker_jenkins" {
  ami           = "ami-0bb84b8ffd87024d8"  # Replace with your preferred AMI
  instance_type = "t2.micro"
  key_name      = "Keypair_name"                   # Replace with your key name

  vpc_security_group_ids = [aws_security_group.allow_ssh_docker.id]

  user_data = <<-EOF
              #!/bin/bash
              sudo -i
              yum update -y
              yum install httpd -y
              systemctl start httpd
              systemctl enable httpd
              yum install docker -y
              systemctl start docker
              systemctl enable docker
              docker pull jenkins/jenkins
              docker run -dit --name jenkins -p 8080:8080 jenkins/jenkins
              EOF

  tags = {
    Name = "EC2-with-docker and Jenkins"
  }
}
```
**EC2 Instance resource Overview.**
* `resource "aws_instance" "ec2_with_docker_jenkins"` Defines an EC2 instance resource named ec2_with_docker_jenkins.
* `ami` Specifies the Amazon Machine Image (AMI) ID for the instance. This can Replaced with the AMI ID of user preference.
* `instance_type` Sets the instance type.
* `key_name` Specifies the name of the key pair to use for SSH access. Replace "test" with your actual key pair name.
* `vpc_security_group_ids` Associates the instance with the security group defined later in the script (aws_security_group.allow_ssh_docker.id).
* `user_data` Provides a shell script to configure the instance upon launch.
> `yum update -y`: Updates all packages/
`yum install httpd -y`: installs Apache HTTP Server./
`systemctl start httpd` and `systemctl enable httpd`:Starts and enables httpd to run on boot./
`yum install docker`: installs docker service./
`systemctl start docker` and `systemctl enable docker`:Starts and enables docker to run./
`docker pull jenkins/jenkins`: Pulls the Jenkins Docker image from docker hub./
`dcoerk run -dit --name jenkins -p 8080:8080 jenkins/jenkins`: Runs a Jenkins container mapping port 8080 on the host to port 8080 on the container.

## Security Group Resource

```resource "aws_security_group" "allow_ssh_docker" {
  name        = "allow_ssh_docker"
  description = "Allow SSH and docker"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 2376
    to_port     = 2376
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
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
    Name = "allow_ssh_docker_jenkins"
  }
}
```

## Security Group Resource overview
* `resource "aws_security_group" "allow_ssh_docker"` Defines a security group named allow_ssh_docker
* ingress rules: Allow inbound traffic.
* Ports added :
> 22: SSH.

> 80: HTTP.

>443: HTTPS.

>2376: Docker daemon (encrypted) for secure Docker communication.

>8080: Jenkins web interface.
* egress rules: Allow all outbound traffic (from_port = 0, to_port = 0, protocol = "-1").

# Full code
```
provider "aws" {
  region     = "us-east-1"
  access_key = "AKIAX37EYSYJOHNZBAO6"
  secret_key = "kA3KVxMin6E1662qPoVSzAaT7+owHUEllfyIrhOt"
}

resource "aws_instance" "ec2_with_docker_jenkins" {
  ami           = "ami-0bb84b8ffd87024d8"  # Replace with your preferred AMI
  instance_type = "t2.micro"
  key_name      = "test"                   # Replace with your key name

  vpc_security_group_ids = [aws_security_group.allow_ssh_docker.id]

  user_data = <<-EOF
              #!/bin/bash
              sudo -i
              yum update -y
              yum install git -y
              yum install httpd -y
              systemctl start httpd
              systemctl enable httpd
              yum install docker -y
              systemctl start docker
              systemctl enable docker
              docker pull jenkins/jenkins
              docker run -dit --name jenkins -p 8080:8080 jenkins/jenkins
              EOF

  tags = {
    Name = "EC2-with-docker and Jenkins"
  }
}
resource "aws_security_group" "allow_ssh_docker" {
  name        = "allow_ssh_docker"
  description = "Allow SSH and docker"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 2376
    to_port     = 2376
    protocol    = "tcp"
    cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
    from_port   = 8080
    to_port     = 8080
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
    Name = "allow_ssh_docker_jenkins"
  }
}
```