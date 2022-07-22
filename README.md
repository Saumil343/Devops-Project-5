# Devops-Project-5
Project Life Cycle:
Test Repo used for web-app deployment and ansible docker files for this project is : https://github.com/Saumil343/test4.git
![devops project 5 drawio](https://user-images.githubusercontent.com/53990452/180130006-8fbf79ce-c3f3-427b-a149-f3e7784ead03.png)
- <b> Tool's Used: </b>
- Terraform
- Jenkins
- Ansible
- Github
- Docker Containers 
- Aws Load Balancer

<!-- 
- <b> Steps Followed </b>
 -->
- <b> Project Descrption </b>
- The project uses the concept of Infrastructre as code and uses terraform for creating a architecture on the AWS, Also I have used a jenkins pipeline to execute Ansible Playbooks over two webservers which i created using terraform, after that the ansible playbook also pulls the latest code from the github and creates a docker image and also runs it on port 3100, Later on i have created a Application Load Balancer and target group with terraform which also adds this two web server to the target group of the load balancer which health checks the port 3100 over web server





- <b> STEPS PERFORMED </b>
1) Created two ec2 instance along with security groups using terraform:
- Terraform File :
```

provider "aws" {
    profile = "Saumil1"
    region = "ap-south-1"
}



resource "aws_security_group" "terraform-prac" {
  name        = "terraform-prac"
  description = "All_traffic_tf"
  vpc_id      = "vpc-e970be82"

  ingress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_tls"
  }
}


resource "aws_instance" "Terraform-instance-1" {
  ami           = "${var.amiid_aws-linux}"
  instance_type = "t2.micro"
  key_name = "terraform-prac"
  vpc_security_group_ids =  ["${aws_security_group.terraform-prac.id}"]
  associate_public_ip_address = true
  tags = {
    Name = "Terraform-Jenkins"
    
  }
}


resource "aws_instance" "Terraform-instance-2" {
  ami           = "${var.amiid_aws-linux}"
  instance_type = "t2.micro"
  key_name = "terraform-prac"
  vpc_security_group_ids =  ["${aws_security_group.terraform-prac.id}"]
  associate_public_ip_address = true
  tags = {
    Name = "Terraform-webserver"
    
  }
}

```

2) Created Jenkins Pipeline for executing the ansible playbook and Docker file on both the instance i created
- Here is my Jenkins Pipeline code:
```
pipeline{
   agent any
   stages{
       stage("git checks"){
           steps{
               git branch: 'main', url: 'https://github.com/Saumil343/test4.git'
           }
       }
       stage("Ansible playbook"){
           steps{
               ansiblePlaybook credentialsId: '91fda46f-c50f-40ce-bab7-8a965994c32b', disableHostKeyChecking: true, installation: 'ansible1', inventory: 'hosts', playbook: 'terraform-ansibe.yml'
           }
       }
   }
}
```
- Also here is the DockerFile for the same where i used apache image which clones all the content from the git repo to our instance  :
```
FROM httpd:2.4
COPY . /usr/local/apache2/htdocs/
```

3) Load Balancer to my both instance using terraform:
- Here is the code which attaches the both instance to a load balancer
- The terraform code for creating a load balancer,target group,listner, and instance attachement and security group is here:
```

provider "aws" {
  profile = "Saumil1"
  region = "ap-south-1"
}
output "vpcid" {
  value = "${local.vpcid}"
}
output "publicsubnetid" {
  value = "${local.publicsubnetid}"
}
output "publicsubnetid2" {
  value = "${local.publicsubnetid2}"
}

resource "aws_security_group" "publicsg" {
  name        = "publicsg1"
  description = "publicsg for lb"
  vpc_id      = "${local.vpcid}"

  ingress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  egress {
    from_port        = 0
    to_port          = 0
    protocol         = "-1"
    cidr_blocks      = ["0.0.0.0/0"]
    ipv6_cidr_blocks = ["::/0"]
  }

  tags = {
    Name = "allow_tls"
  }
}
output "publicsg_id" {
  value = "${aws_security_group.publicsg.id}"
}

locals {
  env = "${terraform.workspace}"

  instanceid1_env = {
        default = "i-085c320fd643d6d73"
      stagging = "i-085c320fd643d6d73"
      production = "i-085c320fd643d6d73"
  }

  instanceid2_env = {
        default = "i-04ba75971ac6955f7"
      stagging = "i-04ba75971ac6955f7"
      production = "i-04ba75971ac6955f7"
  }
instanceid1 = "${lookup(local.instanceid1_env,local.env)}"
instanceid2 = "${lookup(local.instanceid2_env,local.env)}"

  vpcid_env = {
      default = "vpc-e970be82"
      stagging = "vpc-e970be82"
      production = "vpc-e970be82"
  }

  vpcid = "${lookup(local.vpcid_env,local.env)}"

  publicsubnetid_env = {
      default = "subnet-3b153177"
      stagging = "subnet-3b153177"
      production = "subnet-3b153177"
  }

  publicsubnetid = "${lookup(local.publicsubnetid_env,local.env)}"
  
  publicsubnetid2_env = {
      default = "subnet-dc0668a7"
      stagging = "subnet-dc0668a7"
      production = "subnet-dc0668a7"
  }

  publicsubnetid2 = "${lookup(local.publicsubnetid2_env,local.env)}"
   privatesubnetid_env = {
      default = "subnet-8c01f6e7"
      stagging = "subnet-8c01f6e7"
      production = "subnet-8c01f6e7"
  }

  privatesubnetid = "${lookup(local.privatesubnetid_env,local.env)}"
  

}




resource "aws_lb" "aws_lb_1" {
  name               = "test-lb-tf"
  internal           = false
  load_balancer_type = "application"
  security_groups    = [aws_security_group.publicsg.id]
  subnets            = [local.publicsubnetid,local.publicsubnetid2,local.privatesubnetid]

  enable_deletion_protection = false

  tags = {
    Environment = "production"
  }
}

resource "aws_lb_target_group" "aws_lb_1_tg" {
  name     = "gp2"
  port     = 3100
  protocol = "HTTP"
  vpc_id   = local.vpcid
  
  health_check {
    path = "/"
  }

}

resource "aws_vpc" "main" {
  cidr_block = "172.31.32.0/20"
  
}


resource "aws_lb_listener" "test33" {
  load_balancer_arn = aws_lb.aws_lb_1.arn
  port              = "3100"
  protocol          = "HTTP"
 
  default_action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.aws_lb_1_tg.arn
  }
}

resource "aws_lb_target_group_attachment" "test1" {
  target_group_arn = aws_lb_target_group.aws_lb_1_tg.arn
  target_id        = local.instanceid1
  port             = 3100
}


resource "aws_lb_target_group_attachment" "test2" {
  target_group_arn =  aws_lb_target_group.aws_lb_1_tg.arn
  target_id        = local.instanceid2
  port             = 3100
}


```
4) Now i can access the docker container which are hosting the web app using the load balancer DNS 
