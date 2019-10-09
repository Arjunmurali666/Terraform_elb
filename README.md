# Terraform_elb

## Terraform -  This is a code written in terraform to create a classic load balancer in AWS including Auto scaling group and a launch configuration.

```terraform
provider "aws" {
    access_key = "AKIAYCJIVHM55R6SHVHN"
    secret_key = "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"
    region = "us-east-2"
}

resource "aws_security_group" "sg" {

  name = "terraform"
  
  tags = {
      Name = "TF-Webserver"
  }

  ingress {
     from_port = 22
     to_port = 22
     protocol = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
  }

  ingress {
     from_port = 80
     to_port = 80
     protocol = "tcp"
     cidr_blocks = ["0.0.0.0/0"]
  }

  egress {
    from_port = 0
    to_port = 0
    protocol = "-1"
    cidr_blocks = ["0.0.0.0/0"]
  }
   
}

resource "aws_launch_configuration" "my_lc" {
  name_prefix   = "terraform-lc"
  image_id      = "ami-0d8f6eb4f641ef691"
  instance_type = "t2.micro"
  key_name = "ansible"
  associate_public_ip_address = true
  security_groups = [
        "${aws_security_group.sg.id}",
    ]

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_autoscaling_group" "aws_ASG" {
  name                 = "terraform-asg-example"
  launch_configuration = "${aws_launch_configuration.my_lc.name}"
  availability_zones = ["us-east-2a", "us-east-2b", "us-east-2c"]
  min_size             = 1
  max_size             = 2

  lifecycle {
    create_before_destroy = true
  }
}

resource "aws_elb" "my_elb" {
  name               = "terraform-elb"
  availability_zones = ["us-east-2a", "us-east-2b", "us-east-2c"]

  listener {
    instance_port     = 80
    instance_protocol = "http"
    lb_port           = 80
    lb_protocol       = "http"
  }

  health_check {
    healthy_threshold   = 3
    unhealthy_threshold = 2
    timeout             = 3
    target              = "HTTP:80/"
    interval            = 10
  }
}

resource "aws_autoscaling_attachment" "asg_attachment" {
  autoscaling_group_name = "${aws_autoscaling_group.aws_ASG.id}"
  elb                    = "${aws_elb.my_elb.id}"
}
```
