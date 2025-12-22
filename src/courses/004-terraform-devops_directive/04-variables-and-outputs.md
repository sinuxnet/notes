[TOC]

# 04: Variables and Outputs

We can make our previous code much more modular by using variables and outputs which are features of the HCL language.

## 4.1 Variables Types

- **Input Variables**

  - var.\<name\>
  - You can think of input variables as **input parameters** or **arguments** for a function.

- **Local Variables**

  - local.\<name\>
  - When we declare them we use `locals` plural.
  - These are like temporary variables within the scope of a function.
  - These just allow me to take a value which is repeated a few times throughout my configuration and pull it into a variable that can be reused, so I might say this service name is `x` or I am the owner of these resources.

- **Output Variables**

  - These are the **return value** of the function
  - These are what allow us to take multiple Terraform configurations and bundle them together

    So, as an example I might take the output of my apply and return an instance IP address, and use it to set up a firewall rule.

```terraform
variable "instance_type" {
    description = "ec2 instance type"
    type        = string
    default     = "t2.micro"
    }

locals {
    service_name = "My Service"
    owner        = "DevOps Directive"
    }

output "instance_ip_addr" {
    value = aws_instance.instance.public_ip
    }
```

## 4.2 Setting Input Variables

> :warning:**(In order of precedence // lowest :arrow_right: highest)**: First on the list is the lowest precedence and the final on the list is the highest, so if you've declared a variable in multiple ways this is the ordering in which one will be applied over the other.

- **Manual entry during plan/apply:** If you don't specify a variable anywhere and there's no default value, when you run the `terraform plan` command, the Terraform CLI will prompt you and ask you to put a value in. You generally don't want to be doing it that way because then it makes it very easy to make a typo or have a mistake such that the variables change throughout different runs.
- **Default value in declaration block:** Kind of a fallback what it will use if you don't specify a value.
- **`TF_VAR_\<name\>` environment variables:** Start with the prefix of `TF_VAR_`. This is sometimes useful in CI (Continuous Integration) environments or other environments where you would want to change the value based on different attributes.
- **`terraform.tfvars` file:** You can have different `tfvars` files like for staging and production.
- **`*.auto.tfvars` file:** This can be auto applied and these will actually be used **over top of** whatever you have in your `tfvars` files and then finally :arrow_down_small:
- **Command line `-var` or `-var-file`:** When you issue your `terraform plan` command or `terraform apply` command you can have a `-var` or `-var-file` option that will pass those in at runtime. So that is the highest precedence, so if I specify a `-var` on my `terraform apply` command that one is going to be set to that value even if I have defined it in one of these other ways.

## 4.3 Types & Validation

### 4.3.1 Primitive Types

- String
- Number
- Bool

### 4.3.2 Complex Types

- list(\<TYPE\>)
- set(\<TYPE\>)
- map(\<TYPE\>)
- object({\<ATTR NAME\>=\<TYPE\>,...})
- tuple([\<TYPE\>,...])

### 4.3.3 Validation

- **Type checking happens automatically**: When you specify a wrong type or pass wrong value type to a variable, Terraform will throw an error when you try to use that command.
- **Custom conditions can also be enforced**: You can also write your own validation rules that can be applied (a powerful technique to avoid having issues or run some automated testing against codebase to do some static checks).

## 4.4 Sensitive Data

**Make variables as sensitive:**

- Sensitive = true

**Pass to terraform apply with:**

- TF_VAR_variable
- `-var` (retrieved from secret manager at runtime)

**Can also use external secret store:**

- For example, AWS Secret Manager

## 4.5 Examples

### 4.5.1 Example 1

> `main.tf`

```terraform
terraform {
    backend "s3" {
        bucket          = "devops-directive-tf-state"
        key             = "04-variable-and-outputs/example/terraform.tfstate"
        region          = "us-east-1"
        dynamodb-table  = "terraform-state-locking"
        encrypt         = true
        }
    required_providers {
        aws = {
            source  = "hashicorp/aws"
            version = "~> 3.0"
            }
        }
    }
provider "aws" {
    region = "us-east-1"
    }
locals {
    extra_tag = "extra-tag"
    }
resource "aws_instance" "instance" {
    ami           = var.ami
    instance_type = var.instance_type

    tags = {
        Name = var.instance_name
        Extra_tag = local.extra_tag
        }
    }
resource "aws_db_instance" "db_instance" {
    allocated_storage   = 20
    storage_type        = "gp2"
    engine              = "postgres"
    engine_version      = "12"
    instance_class      = "db.t2.micro"
    name                = "mydb"
    username            = var.db_user
    password            = var.db_password
    skip_final_snapshot = true
    }
```

Local variables ~> Things that are scoped to within this project and we can't actually pass them in at runtime.

> `variables.tf`

```terraform
variable "instance_name" {
    description = "Name of ec2 instance"
    type = string
    }
variable "ami" {
    description = "Amazon machine image to use for ec2 instance"
    type        = string
    default     = "ami-01899242bb902164" # Ubuntu 20.04 LTS // us-east-1
    }
variable "instance_type" {
    description = "ec2 instance type"
    type = "string"
    default = "t2.micro"
    }
variable "db_user" {
    description = "username for database"
    type = "string"
    default = "foo"
    }
variable "db_pass" {
    description = "password for database"
    type = "string"
    sensitive = true
    }
```

These are all inputs to this Terraform configuration that can be configured and changed at **runtime.**

> `terraform.tfvars`

Where we can define variables (non-sensitive):

```terraform
instance_name = "hello-world"
ami           = "ami-01189924bb902164" # Ubuntu 20.04 LTS // us-east-1
instance_type = "t2.micro"
```

> `another-variable-files.tfvars`

We can also have more than one `tfvars` file. By default it's going to apply `terraform.tfvars`. If we have some other filename, then we have to explicitly tell it when we do `terraform apply` like this:

`terraform apply -var-file=another-variable-files.tfvars`

```terraform
instance_name = "hello-world-2"
```

> output.tf

We might want access to the IP address of the instance (or database) once its provisioned.

```terraform
output "instance_ip_addr" {
    value = aws_instance.instance.private_ip
    }

output "db_instance_addr" {
    value = aws_instance.db_instance.address
    }
```

**And Finally:**

```shell
terraform apply -var="db_user=myuser" -var="db_pass=SOMETHING_SUPER_SECURE"
```

The sensitive values can be stored in something like GitHub secrets, AWS Secret Manager and accessed within GitHub Actions, etc.

Terraform won't actually echo out sensitive variables into the terminal.

### 4.5.1 Example 2 (web app)

> variables.tf

```terraform
# General Variables

variable "region" {
  description = "Default region for provider"
  type        = string
  default     = "us-east-1"
}

# EC2 Variables

variable "ami" {
  description = "Amazon machine image to use for ec2 instance"
  type        = string
  default     = "ami-011899242bb902164" # Ubuntu 20.04 LTS // us-east-1
}

variable "instance_type" {
  description = "ec2 instance type"
  type        = string
  default     = "t2.micro"
}

# S3 Variables

variable "bucket_prefix" {
  description = "prefix of s3 bucket for app data"
  type        = string
}

# Route 53 Variables

variable "domain" {
  description = "Domain for website"
  type        = string
}

# RDS Variables

variable "db_name" {
  description = "Name of DB"
  type        = string
}

variable "db_user" {
  description = "Username for DB"
  type        = string
}

variable "db_pass" {
  description = "Password for DB"
  type        = string
  sensitive   = true
}

```

> output.tf

```terraform
output "instance_1_ip_addr" {
  value = aws_instance.instance_1.public_ip
}

output "instance_2_ip_addr" {
  value = aws_instance.instance_2.public_ip
}

output "db_instance_addr" {
  value = aws_db_instance.db_instance.address
```

> main.tf

```terraform
terraform {
  # Assumes s3 bucket and dynamo DB table already set up
  # See /code/03-basics/aws-backend
  backend "s3" {
    bucket         = "devops-directive-tf-state"
    key            = "04-variables-and-outputs/web-app/terraform.tfstate"
    region         = "us-east-1"
    dynamodb_table = "terraform-state-locking"
    encrypt        = true
  }

  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 3.0"
    }
  }
}


provider "aws" {
  region = var.region
}

resource "aws_instance" "instance_1" {
  ami             = var.ami
  instance_type   = var.instance_type
  security_groups = [aws_security_group.instances.name]
  user_data       = <<-EOF
              #!/bin/bash
              echo "Hello, World 1" > index.html
              python3 -m http.server 8080 &
              EOF
}

resource "aws_instance" "instance_2" {
  ami             = var.ami
  instance_type   = var.instance_type
  security_groups = [aws_security_group.instances.name]
  user_data       = <<-EOF
              #!/bin/bash
              echo "Hello, World 2" > index.html
              python3 -m http.server 8080 &
              EOF
}

resource "aws_s3_bucket" "bucket" {
  bucket_prefix = var.bucket_prefix
  force_destroy = true
}

resource "aws_s3_bucket_versioning" "bucket_versioning" {
  bucket = aws_s3_bucket.bucket.id
  versioning_configuration {
    status = "Enabled"
  }
}

resource "aws_s3_bucket_server_side_encryption_configuration" "bucket_crypto_conf" {
  bucket = aws_s3_bucket.bucket.bucket
  rule {
    apply_server_side_encryption_by_default {
      sse_algorithm = "AES256"
    }
  }
}

data "aws_vpc" "default_vpc" {
  default = true
}

data "aws_subnet_ids" "default_subnet" {
  vpc_id = data.aws_vpc.default_vpc.id
}

resource "aws_security_group" "instances" {
  name = "instance-security-group"
}

resource "aws_security_group_rule" "allow_http_inbound" {
  type              = "ingress"
  security_group_id = aws_security_group.instances.id

  from_port   = 8080
  to_port     = 8080
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]
}

resource "aws_lb_listener" "http" {
  load_balancer_arn = aws_lb.load_balancer.arn

  port = 80

  protocol = "HTTP"

  # By default, return a simple 404 page
  default_action {
    type = "fixed-response"

    fixed_response {
      content_type = "text/plain"
      message_body = "404: page not found"
      status_code  = 404
    }
  }
}

resource "aws_lb_target_group" "instances" {
  name     = "example-target-group"
  port     = 8080
  protocol = "HTTP"
  vpc_id   = data.aws_vpc.default_vpc.id

  health_check {
    path                = "/"
    protocol            = "HTTP"
    matcher             = "200"
    interval            = 15
    timeout             = 3
    healthy_threshold   = 2
    unhealthy_threshold = 2
  }
}

resource "aws_lb_target_group_attachment" "instance_1" {
  target_group_arn = aws_lb_target_group.instances.arn
  target_id        = aws_instance.instance_1.id
  port             = 8080
}

resource "aws_lb_target_group_attachment" "instance_2" {
  target_group_arn = aws_lb_target_group.instances.arn
  target_id        = aws_instance.instance_2.id
  port             = 8080
}

resource "aws_lb_listener_rule" "instances" {
  listener_arn = aws_lb_listener.http.arn
  priority     = 100

  condition {
    path_pattern {
      values = ["*"]
    }
  }

  action {
    type             = "forward"
    target_group_arn = aws_lb_target_group.instances.arn
  }
}


resource "aws_security_group" "alb" {
  name = "alb-security-group"
}

resource "aws_security_group_rule" "allow_alb_http_inbound" {
  type              = "ingress"
  security_group_id = aws_security_group.alb.id

  from_port   = 80
  to_port     = 80
  protocol    = "tcp"
  cidr_blocks = ["0.0.0.0/0"]

}

resource "aws_security_group_rule" "allow_alb_all_outbound" {
  type              = "egress"
  security_group_id = aws_security_group.alb.id

  from_port   = 0
  to_port     = 0
  protocol    = "-1"
  cidr_blocks = ["0.0.0.0/0"]

}


resource "aws_lb" "load_balancer" {
  name               = "web-app-lb"
  load_balancer_type = "application"
  subnets            = data.aws_subnet_ids.default_subnet.ids
  security_groups    = [aws_security_group.alb.id]

}

resource "aws_route53_zone" "primary" {
  name = var.domain
}

resource "aws_route53_record" "root" {
  zone_id = aws_route53_zone.primary.zone_id
  name    = var.domain
  type    = "A"

  alias {
    name                   = aws_lb.load_balancer.dns_name
    zone_id                = aws_lb.load_balancer.zone_id
    evaluate_target_health = true
  }
}

resource "aws_db_instance" "db_instance" {
  allocated_storage   = 20
  storage_type        = "standard"
  engine              = "postgres"
  engine_version      = "12"
  instance_class      = "db.t2.micro"
  name                = var.db_name
  username            = var.db_user
  password            = var.db_pass
  skip_final_snapshot = true
}
```
