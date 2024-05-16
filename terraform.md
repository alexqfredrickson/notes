# Terraform

## Syntax

* Terraform `.tf` files can be expressed either in JSON or [HCL](https://github.com/hashicorp/hcl) (or "Hashicorp Configuration Language") format.
* `~>` represents a "maximum version" constraint.
* `main.tf` is just a naming convention, and isn't necessary.

## Components

### State

Terraform [state](https://developer.hashicorp.com/terraform/language/state) describes the infrastructure that is managed by Terraform: mapping Terraform configuration with external resources.

* State is stored (by default) in a `terraform.tfstate` file. For teams, remote state files are used - which have built-in locking mechanisms.
* `terraform refresh` occurs prior to any operation. This updates the local state.
* `terraform state` can be used to modify the state file.
* State is expressed in JSON format.

### Backends
 
[Backends](https://developer.hashicorp.com/terraform/language/settings/backends/configuration) store state files. The default backend is `local`, storing state on-disk.

Backends are expressed within `terraform` blocks - for example:

```hcl
terraform {
  backend "remote" {
    organization = "example_corp"
 
    workspaces {
      name = "my-app-prod"
    }
  }
}
```

* Backends may house workspaces, which house state files. Some (but not all) Terraform backends support multiple workspaces.

#### Remote backends

[Remote backends](https://developer.hashicorp.com/terraform/language/settings/backends/remote) is a unique type of backend that stores state snapshots, and executes operations for the HCP Terraform application.

### Workspaces

Workspaces are like atomic instances of state data.

They are useful in representing individual environments. For example: a "production" workpace versus a "development" workspace.

### Provisioners

Provisioners execute imperative scripts. This breaks Terraform's declarative model, and should be used "as a last resort".

### Providers

[Providers](https://developer.hashicorp.com/terraform/language/providers) are plugins that tell Terraform how to interact with external hosting environments.

### Variables

Variables (or "input variables") are [template parameters](https://developer.hashicorp.com/terraform/language/values/variables). 

Variables can be declared in root modules, and/or child modules. At the root level, they are set in the CLI call to Terraform, or as an environment variable. At the module level, they are set in the `module` block.

They are declared in `variable` blocks:

```hcl
# base case
variable "image_id" {
  type = string
}
 
# with default
variable "availability_zone_names" {
  type    = list(string)
  default = ["us-west-1a"]
}
 
# as a list of objects, with defaults 
variable "docker_ports" {
  type = list(object({
    internal = number
    external = number
    protocol = string
  }))

  default = [
    {
      internal = 8300
      external = 8300
      protocol = "tcp"
    }
  ]
}
```

All variables are required unless `default` is declared.

Variables have optional arguments:

| Argument    | Description |
| -------- | ------- |
| default  | The default value. Makes the variable optional.    |
| type | `string`, `number`, or `bool`. `list()`, `set()`, `map()`, `object()`, and `tuple()` are also valid complex types.     |
| description    | A description.    |
| validation    | Some validation contraints - for example: `length(var.image_id) > 4 && substr(var.image_id, 0, 4) == "ami-"`.    |
| sensitive    | If `true`, then `terraform plan` and `terraform apply` will not send the variable to STDOUT. Sensitive values are still stored in plaintext in the Terraform state.   |
| nullable    | If `true`, a null value is permitted.    |

Variables are referenced like "`var.image_id`". They may also be interpolated into strings - like `"${var.foo}-bar!"`.

### Locals

[Local values](https://developer.hashicorp.com/terraform/language/values/locals) are temp variables, expressed in their own types of resource blocks. They function similar to enums. For example:

```hcl
locals {
  service_name = "forum"
  owner        = "Community Team"
}
```

They are referenced like this:

```hcl
resource "aws_instance" "example" {
  # ...
 
  tags = local.common_tags
}
```

### Expressions

Expressions are like [computed values](https://developer.hashicorp.com/terraform/language/expressions). They are sometimes used in conjunction with locals.

### Outputs

Outputs (or "output values") are like function return statements.

## Concepts

### Resource Dependencies

Terraform automatically detects _implicit_ dependencies, when a resource's property is referenced by another resource:

```hcl
data "aws_ami" "amazon_linux" {  
  most_recent = true  
  owners      = ["amazon"]   
} 

resource "aws_instance" "example_a" {
  ami           = data.aws_ami.amazon_linux.id    instance_type = "t2.micro"
} 

resource "aws_eip" "ip" {  
  vpc      = true  
  instance = aws_instance.example_a.id
}
```

It can't always find this out, so it allows for `depends_on` arguments to declare _explicit_ dependencies - for example: `depends_on = [aws_s3_bucket.example, aws_instance.example_c]`.


## Commands

### `terraform apply`

`terraform apply` applies Terraform *plans* against Terraform *backends* (which store state). By default, it generates a new plan. You can also give it a plan from `terraform plan`.

* This is an asychronous operation that supports up to ten concurrent nodes by default.

#### `terraform apply -replace`

Terraform can't always detect issues with remote objects, so `terraform apply -replace` can replace atomic resources - for example:

```
$ terraform apply -replace="aws_instance.example"
# ...
 
  # aws_instance.example will be replaced, as requested
-/+ resource "aws_instance" "example" {
      # ...
    }
```

### `terraform state`

* `terraform state mv` - `TODO`

### `terraform init`

`terraform init` instantiates state files, downloads providers (i.e. cloud plugins), and downloads modules (i.e. groups of resource configurations) to the root module directory (i.e. where `.tf` files are located).

### `terraform force-unlock`

Unlocks the state file.

### `terraform get`

Downloads modules recursively to the current working directory. Generally, this is not executed - `terraform init` does the same thing, and then some.

### `terraform workspace`

Creates, deletes, lists, selects, and shows (displays names) of workspaces.

* The syntax is `terraform workspace new [workspace-name]` (for example).

### `terraform show`

Displays state information.

### `terraform fmt`

Reformats configuration to conform with style guidelines.

### `terraform validate`

Ensures that configuration is syntactically _valid_. Note that `terraform fmt` is what makes it _pretty_.

## Vault

Hashicorp Vault is a third-party centralized secrets management service, which encrypts secrets at-rest and in-transit to clients. It's very similar to AWS Secrets Manager. 

Terraform has providers that talk to Vault.

Terraform secrets are sometimes exposed through Terraform configuration and state files, so Hashicorp claims that it's best practice to use Vault.

## CLI

Terraform CLI stores local state in a `terraform.tfstate` file.

## HCP Terraform

HCP Terraform (formerly Terraform Cloud) is located at https://app.terraform.io/. It's basically a web-based abstraction over Terraform, designed for use with teams. It's like what GitHub is to `git` itself.

* HCP Terraform creates speculative plans and runs Sentinel scans on PR approval. It does _not_ automatically apply Terraform plans by-default.

## Examples

### Basic Example

```hcl
terraform {
  required_providers {
    aws = {
      source  = "hashicorp/aws"
      version = "~> 4.16"
    }
  }
 
  required_version = ">= 1.2.0"
}
 
provider "aws" {
  region  = "us-west-2"
}
 
resource "aws_instance" "app_server" {
  ami           = "ami-830c94e3"
  instance_type = "t2.micro"
 
  tags = {
    Name = "ExampleAppServerInstance"
  }
}
```

* The `terraform` block contains [basic Terraform configuration](https://developer.hashicorp.com/terraform/language/settings#terraform-block-syntax).
* `terraform -> required_providers` [is where providers are defined](https://developer.hashicorp.com/terraform/language/providers/requirements). Providers are plugins that interact with remote systems.
* `terraform -> required_providers -> {provider} -> source` is a [source address](https://developer.hashicorp.com/terraform/language/providers/requirements#source-addresses). This is a globally unique identifier that expresses where Terraform can download the provider from - in format `NAMESPACE/TYPE` OR `HOSTNAME/NAMESPACE/TYPE`. The default hostname is `registry.terraform.io`.
* `~>` represents a "maximum version" constraint.
* `terraform -> required_version` is the Terraform version; `>= x.x` represents the minimum version, and is best-practice to allow for patching.
* Provider configuration is found at the `provider` block. For `hashicorp/aws`, see https://registry.terraform.io/providers/hashicorp/aws/latest/docs for documentation about this provider's properties. 
* The `resource` block defines an atomic resource, with a specific name. Names are locally-scoped. `resource "aws_instance" "app_server"` is an AWS EC2 instance named "web". Arguments depend on the resource type. Arguments are documented at the Terraform registry - for example: `https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/instance`.


## Errata

* Terraform can handle cross-cloud dependencies.