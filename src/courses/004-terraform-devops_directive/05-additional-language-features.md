[TOC]

# 05: Additional Language Features

## 5.1 Expressions + Functions

**_USE THE DOCS!_**

### 5.1.1 Expressions

- Template Strings: If you are familiar with how you can have a string in JavaScript with curly braces inside and reference a variable from within that string (build strings dynamically)
- Operators (!, -, \*, ', %, >, ==, etc...)
- Conditionals ( cond ? true : false)
- For ([**for** o in **var**.list : o.id])
- Splat (var.list[*].id): Take the values in a list and expand them out over some way in which we want to use them
- Dynamic Blocks
- Constraints (Type & Version)

```terraform
# -----------------
# <--- STRINGS --->

"foo" # literal string
"foo {var.bar}" # template string

# -------------------
# <--- OPERATORS --->

# order of operations:
!, - # (multiplication by -1)
*, /, % # (modulo)
+, - # (subtraction)
>, >=, <, <= # (comparison)
==, != # (equality)
&& # (AND)
|| # (OR)


# ----------------------
# <--- CONDITIONALS --->

condition ? true_val : false_val

# for example
var.a != "" ? var.a : "default-a"

```

### 5.1.2 Functions

- Numeric: It's mathematical
- String
- Collection
- Encoding
- Filesystem
- Date & Time: Where if we wanted to have something that is named after the specific timestamp at which it was created.
- Hash & Crypto
- IP Network
- Type Conversion

```terraform
# -----------------
# <--- NUMERIC --->
abs()
ceil()
floor()
log()
max()
parseint() # parse as integer
pow()
signum() # sign of number

# ----------------
# <--- STRING --->
chomp() # remove newlines at end
format() # format number
formatlist()
indent()
join()
lower()
regex()
regexall()
replace()
split()
strrev() # reverse string
substr()
title()
trim()
trimprefix()
trimsuffix()
trimspace()
upper()
```

## 5.2 Meta-Arguments

### 5.2.1 depends_on

If there are things that need to happen in a certain sequence. If you're provisioning a server and then you need the IP address from that to pass to a firewall rule.

When you run the plan command, Terraform will figure out the sequence of events and the dependency graph there, but there are cases though where one resource implicitly depends on another but there's no direct connection within the config.

- Terraform automatically generates dependency graph based on references

- If two resources depend on each other (but not each other's data), `depends_on` specifies that dependency to enforce ordering.

- For example, if software on the instance needs access to S3, trying to create the `aws_instance` would fail if attempting to create it before the `aws_iam_role_policy`.

  If my instance needs to be able to access an S3 bucket we need to have a role policy that can make that happen but there's no direct connection within my config and so we can tell Terraform what this depends on using the `depends_on` key.

  > _You should make sure to provision this role policy before you provision the instance otherwise it's going to fail._

```terraform
resource "aws_iam_role" "example" {
    name = "example"
    assume_role_policy = "..."
    }

resource "aws_iam_instance_profile" "example" {
    role = aws_iam_role.example.name
    }

resource "aws_iam_role_policy" "example" {
    name = "example"
    role = aws_iam_role.example.name
    policy = jsonencode({
        "Statement" = [{
            "Action" = "s3:*",
            "Effect" = "Allow"
            }],
        })
    }

resource "aws_instance" "example" {
    ami           = "ami-a1b2c3d4"
    instance_type = "t2.micro"

    iam_instance_profile = aws_iam_instance_profile.example

    depends_on = [
        aws_iam_role_policy.example,
        ]
    }
```

### 5.2.2 Count

- Allows for creation of multiple resources/modules from a single block
- Useful when the multiple necessary resources are nearly identical

```terraform
resource "aws_instance" "server" {
    count = 4 # create four EC2 instances

    ami           = "ami-a1b2c3d4"
    instance_type = "t2.micro"

    tags = {
        Name = "Server ${count.index}"
        }
    }
```

### 5.2.3 for_each

It is like the `count` argument but it gives us much more control, so whereas `count` we literally just get one, two, three, four, etc.

- Allows for creation of multiple resources/modules from a single block
- Allows more control to customize each resource than count

```terraform
locals {
    subnet_ids = toset ([
        "subnet-abcdef",
        "subnet-012345",
        ])
    }

resource "aws_instance" "server" {
    for_each = local.subnet_ids

    ami           = "ami-a1b2c3d4"
    instance_type = "t2.micro"
    subnet_id     = each.key

    tags = {
        Name = "Server ${each.key}"
        }
    }
```

### 5.2.4 Lifecycle

- A set of meta arguments to control Terraform behavior for specific resources

- `create_before_destroy` can help with zero downtime deployments

  This is important because there are certain things where we need Terraform to take actions in a specific order. We can use `create_before_destroy` argument to say:

  > _If we're replacing this server, we want you to provision the new one before you delete the old one_

  So this can help us to avoid downtime for applications if we do this properly.

- `ignore_changes` prevents Terraform from trying to revert metadata being set elsewhere

  There are also sometimes where behind the scenes after you've provisioned a resource (AWS or whatever) will add some metadata to that resource, and those can be very annoying from a Terraform state perspective because it looks as though you have a change between your state and the deployed infrastructure, and you can tell Terraform:

  > _Oh yes, that tag exists we don't need to worry about it_

  You can put it within the `ignore_changes` lifecycle meta tag. Otherwise you can end up in a state where you're trying to revert those changes back and forth and you're kind of fighting with the system.

- `prevent_destroy` causes Terraform to reject any plan which would destroy this resource

  This is a kind of safeguard if you have some piece of your infrastructure that is critical to not delete, you can add this tag and then anytime if the `terraform plan` or `terraform apply` command would have deleted that resource, it will throw an error and so this can help you really lock down some specific core pieces of the infrastructure that you don't want to be deleted.

```terraform
resource "aws_instance" "server" {
    ami           = "ami-a1b2c3d4"
    instance_type = "t2.micro"

    lifecycle {
        create_before_destroy = true
        ignore_changes = [
            # Some resources have metadata
            # modified automatically outside
            # of Terraform
            tags
            ]
        }
    }
```

# 5.3 Provisioners

## Perform action on local or remote machine

Provisioners allow you to perform some action either locally or on a remote machine. There are a number of different types of provisioners:

- file
- local-exec
- remote-exec
- vendor
  - Chef
  - Puppet
  - Ansible

Example 1: Once you've finished applying your Terraform config and you have your server up, then you can use the Ansible provisioner to then go off and install and modify those servers.

Example 2: In a simpler example, we could have a startup script that we wanted to execute after we have provisioned our servers and that could be a file provisioner with a bash script stored there that Terraform configuration references.
