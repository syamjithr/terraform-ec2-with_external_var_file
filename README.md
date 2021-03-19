# Terraform-Ec2-Loading_Variables_From_External_File
## Provisioning an ec2 instance with external variable file using Terraform
### Prerequisite:
Download Terraform from link and set up it in a linux machine. ( Here I'm using a ec2 instance) Configure aws cli in the instance.
### First we need to create a directory 
##### mkdir ec2-var
##### cd ec2-var/
##### terraform init
##### vim ec2-infra.tf
```
##############################################################################
# Uploading Keypair
##############################################################################
resource "aws_key_pair" "mykey" {

  key_name   = "mykey"
  public_key = "--------------------replace with public key---------------------------"
  tags = {
      Name = "mykey"
  }
    
}

##############################################################################
# Security Group webserver
##############################################################################

resource "aws_security_group" "webserver" {
    
  name        = "webserver-firewall"
  description = "allows 22,80,443 only"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
 
  ingress {
    from_port   = 80
    to_port     = 80
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
    
  ingress {
    from_port   = 443
    to_port     = 443
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
    
    
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "webserver-firewall"  
  }
}


##############################################################################
# Security Group database
##############################################################################

resource "aws_security_group" "database" {
    
  name        = "database-firewall"
  description = "allows 22,3306 only"

  ingress {
    from_port   = 22
    to_port     = 22
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
 
  ingress {
    from_port   = 3306
    to_port     = 3306
    protocol    = "tcp"
    cidr_blocks = [ "0.0.0.0/0" ]
  }
    
  egress {
    from_port   = 0
    to_port     = 0
    protocol    = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }

  tags = {
    Name = "database-firewall"  
  }
}

##############################################################################
# Ec2 Instance
##############################################################################

resource "aws_instance" "webeserver" {
    
  ami                         = var.instance_ami
  instance_type               = var.instance_type
    
  associate_public_ip_address = true
  key_name                    = aws_key_pair.mykey.key_name
  vpc_security_group_ids      = [ aws_security_group.webserver.id , aws_security_group.database.id ]
  availability_zone           = "ap-south-1a"
  user_data                   = file("setup.sh")
  tags = {
    Name = "webserver"
  }
    
  lifecycle {
    create_before_destroy = true
  }
}
```
##### External variables file
vim variables.tf
```
variable "instance_ami" {
    
    description = "ami id of the ec2 instance"
    default = "ami-0eeb03e72075b9bcc"
    
}

variable "instance_type" {
    
    description = "Type of the ec2 instance"
    default = "t2.micro"
    
}
```
##### Execution
```
#terraform validate   - syntax check 

#terraform plan - Creating an execution plan ( to check what will get installed before running it)

# terraform apply - Applying

# terraform destroy - Destroying what we have applied through terrafrom apply

We can also use -auto-approve while applying.

# terraform apply -auto-approve
```
