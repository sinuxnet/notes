[TOC]

# 08: Testing Terraform Code

## 8.1 Code Rot

Code Rot in general refers to this concept that over time things change about your software system and if you don't test and use code it will oftentimes degrade over time, which could be due to external dependencies changing, or due to other changes in your codebase that are impacting your specific function.

### 8.1.1 Types of Rot

- **Out of band changes**: Deploy something with Terraform and then your colleague goes in and changes something via UI, that is now a misconfiguration
- **Unpinned versions**: If you forgot to specify version of our provider and it just used latest one, that could cause conflict if that provider was upgraded in the background,
- **Deprecated dependencies**: We are depending on an external module or a specific resource type within the cloud provider and that was then deprecated.
- **Unapplied changes**: If we have made a change to our infrastructure config (our Terraform config) but never got applied to a specific environment, let's say we rolled it out to staging but we forgot because we didn't automate it and we forgot to apply to production.

> **- How to prevent these types of code rots?**
>
> **+ TESTING YOUR CODE!**

## 8.2 Types of Test

### 8.2.1 Static Checks

#### 8.2.1.1 Built in

- **`terraform fmt`**: Formatting indent and structuring the codebase

  Try `terraform fmt -check` to just see what files have mistakes.

- **`terraform validate`**: Check to see if my configurations are setting all of the required input variables. Am I passing a boolean to a number variable, etc.

- **`terraform plan`**: Tell us what needs to happen to get from our desired config to an eventual state of those resources being deployed. It can be a great way to check if something has changed out of band.

  If run `terraform plan` and it says:

  - **"Zero changes are required"**: Means we're good to go, our config has not been modified

  - **"Need to change these four things"**: Means that something happened!

    1. You have changed config.
    2. If you have not changed config, means something happened out of band.

    :warning: Often a good check is to just run a `terraform plan` once a day or once a week

- **Custom validation rules**:

  ```terraform
  variable "short_variable" {
    type = string

    validation {
      condition     = length(var.short_variable) < 4
      error_message = "The short_variable value must be less than 4 characters!"
    }
  }
  ```

#### 8.2.1.2 External

- `**tflint**`
- **`checkov`, `ttfsec`, `terrascan`, terraform-compliance, `snyk`**
- Terraform Sentinel (enterprise only)

### 8.2.2 Manual Testing

Following the similar lifecycle of Terraform commands.

- `terraform init`
- `terraform plan`
- `terraform apply`
- `terraform destroy`

It's great, but we would much prefer for this type of thing to be automated.

### 8.2.3 Automated

#### 8.2.3.1 with **BASH!**

```bash
#!/bin/bash

set -euo pipefail

# Change directory to project
cd /path/to/project

# Create the resources
terraform init
terraform apply -auto-approve

# Wait while the instance boots up
# (Could also use a provisioner in the TF config to do this)
sleep 60

# query the output, extract the IP and make a request
terraform output -json |\
jq -r '.instance_ip_addr.value' |\
xargs -I {} curl 'http://{}:8080' -m 10

# If request succeeds, destroy the resources
terraform destroy -auto-approve
```

#### 8.2.3.1 with **Terratest**

```go
package test

import (
    "crypto/tls"
    "fmt"
    "testing"
    "time"

    "github.com/gruntwork-io/terratest/modules/http-helper"
    "github.com/gruntwork-io/terratest/modules/terraform"
)

func TestTerraformHelloWorldExample(t *testing.T) {
    // retryable errors in terraform testing.
    terraformOptions := terraform.WithDefaultRetryableErrors(t, &terraform.Options{
    	TerraformDir: "../../examples/hello-world",
     })

    defer terraform.Destroy(t, terraformOptions)

    terraform.InitAndApply(t, terraformOptions)

    instanceURL := terraform.Output(t, terraformOptions, "url")
    tlsConfig := tls.Config{}
    maxRetries := 30
    timeBetweenRetries := 10 * time.Second

    http_helper.HttpGetWithRetryWithCustomValidation(
    	t, instanceURL, &tlsConfig, maxRetries, timeBetweenRetries, validate,
    )
}

func validate(status int, body string) bool {
        fmt.Println(body)
        return status == 200
}
```

directory tree:

> .
> ├── go.mod
> ├── go.sum
> └── hello_world_test.go

then we run this commands:

```bash
go mod download
go test -v --timeout 10m
```
