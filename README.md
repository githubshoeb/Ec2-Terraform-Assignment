# Ec2-Terraform-Assignment
terraform code

provider "aws" {
  region = "us-east-1"
}

resource "aws_instance" "my_ec2_instance" {
  ami           = "ami-0dba2cb6798deb6d8"
  instance_type = "t2.micro"
  vpc_security_group_ids = [
    aws_security_group.my_ec2_instance_security_group.id,
  ]
  tags = {
    Name = "My EC2 Instance"
  }

  connection {
    type        = "ssh"
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")
    timeout     = "2m"
    host        = self.public_ip
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx",
      "sudo service nginx start",
    ]
  }
}

resource "aws_security_group" "my_ec2_instance_security_group" {
  name        = "my-ec2-instance-security-group"
  description = "Allow SSH access from anywhere"
  vpc_id      = aws_default_vpc.default.id

  ingress {
    from_port = 22
    to_port   = 22
    protocol  = "tcp"
    cidr_blocks = [
      "0.0.0.0/0",
    ]
  }
}

resource "aws_default_vpc" "default" {
}
