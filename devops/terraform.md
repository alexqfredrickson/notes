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

#### Local backends

* Local backends do not support locking (go figure).

#### Remote backends

[Remote backends](https://developer.hashicorp.com/terraform/language/settings/backends/remote) is a unique type of backend that stores state snapshots, and executes operations for the HCP Terraform application.

### Workspaces

Workspaces are like atomic instances of state data.

They are useful in representing individual environments. For example: a "production" workpace versus a "development" workspace.

* Workspace-specific state is found in `terraform.tfstate.d` files (not `terraform.tfstate`).
* In HCP Terraform, workspaces are often mapped to different code repositories. In local Terraform, one code repository generally contains multiple workspaces (perhaps). 

### Provisioners

Provisioners execute imperative scripts. This breaks Terraform's declarative model, and should be used "as a last resort".

### Providers

[Providers](https://developer.hashicorp.com/terraform/language/providers) are plugins that tell Terraform how to interact with external hosting environments.

* Provider dependencies are defined by way of provider blocks, references to providers in state, and references to providers in resource blocks. They are not defined by providers found in the local working directory - those might just be the result of a cache or something.
* Providers cannot be downloaded and installed directly from source code. They are acquired from  Terraform registries, HashiCorp releases, local plugins directories, and from plugins caches.
* Provider blocks are (surprisingly) optional; Terraform tries to figure out what you're talking about automatically, and downloads providers without them being explicitly declared ... on a side note, this seems a bit sketchy!

### Input Variables

Input variables are [template parameters](https://developer.hashicorp.com/terraform/language/values/variables). They can be declared in root modules and/or child modules. 

At the root level, they are set in either:

* HCP Terraform workspaces
* With the `-var` command line option, like `terraform apply -var="image_id=ami-abc123"`.
* As environment variable - prefixed with `TF_VAR` - like `export TF_VAR_image_id=ami-abc123`.
* In a file called `.tfvars`, like `terraform apply -var-file="testing.tfvars"`.

At the module level, input variables are set in the `module` block.

Variables are referenced by way of `variable` blocks - for instance:

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

Note that from a practical standpoint, you're probably going to hardcode variables in a `terraform.tfvars` file (which is usually `.gitignore`d) and then create a `variables.tf` file to actually instantiate variable blocks. Both are required to make this work.

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

### Modules

[Modules](https://developer.hashicorp.com/terraform/language/modules) are containers for groups of resources. They consist of `.tf` files in a directory.

Terraform always has a "root" module, and child modules, which bring resources into their own configuration.

### Data Sources

[Data sources](https://developer.hashicorp.com/terraform/language/data-sources) let Terraform load data from outside of Terraform, or from other Terraform configurations.

For example:

```
data "aws_ami" "example" {
  most_recent = true
 
  owners = ["self"]
  tags = {
    Name   = "app-server"
    Tested = "true"
  }
}
```

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

### `terraform apply -refresh=False`

This prevents Terraform from refreshing infrastructure state prior to a `terraform apply`. This is used when you want to skip the overhead of Terraform re-checking the remote infrastructure to ensure it matches the current local state.

#### `terraform apply -auto-approve`

This is like a `terraform apply -force`. Normally, `terraform apply` will prompt, requiring manual intervention. Don't use this in production.

### `terraform state`

This is an advanced command used to modify Terraform state directly.

#### `terraform state list`

Shows all resources defined in the current state.

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

* This does not run recursively by-default.

### `terraform validate`

Ensures that configuration is syntactically _valid_. Note that `terraform fmt` is what makes it _pretty_.

### `terraform console`

This provides an interacive console for the evaluation of expressions. Confusingly, this holds a lock on state, even if the backend is `local`.

### `terraform import`

Imports existing infrastructure into the Terraform state.

### `terraform plan`

This creates a manifest of the resources that will be created on `terraform apply`. This is kind of like a dry-run. It can be saved to an output file and changed for later, with `terraform plan out=[path]`.

### `terraform destroy`

Deletes all infrastructure. This is an alias for `terraform apply -destroy`. This prompts for manual confirmation.

#### `terraform destroy -auto-approve`

This is the same as `terraform destroy`, but it does not prompt for manual confirmation.

### `terraform login`

This logs into HCP Terraform.

## Vault

Hashicorp Vault is a third-party centralized secrets management service, which encrypts secrets at-rest and in-transit to clients. It's very similar to AWS Secrets Manager. 

Terraform has providers that talk to Vault.

Terraform secrets are sometimes exposed through Terraform configuration and state files, so Hashicorp claims that it's best practice to use Vault.

## CLI

Terraform CLI stores local state in a `terraform.tfstate` file.

## HCP Terraform

HCP Terraform (formerly Terraform Cloud) is located at https://app.terraform.io/. It's basically a web-based abstraction over Terraform, designed for use with teams. It's like what GitHub is to `git` itself.

* HCP Terraform creates speculative plans and runs Sentinel scans on PR approval. It does _not_ automatically apply Terraform plans by-default.
* When migrating to HCP Terraform (from local), HCP Terraform will inherit from the local version (i.e. the version used to perform the migration).

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


### Practical Examples

These are some practical examples of how to structure a Terraform project from scratch - via [https://www.resourcely.io/post/10-terraform-config-root-setups](https://www.resourcely.io/post/10-terraform-config-root-setups).

#### Single Environment, Single Config

```
📂 repo-root/
├── 📂 terraform/
│   ├── main.tf
│   ├── variables.tf
│   ├── outputs.tf
│   ├── terraform.tfvars
```

Low complexity, but lacks environment isolation, limited scalability, and no rollback-per-environment.

#### Multi-Environment, Separate State Files

```
📂 repo-root/
├── 📂 terraform/
│   ├── 📂 dev/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
│   ├── 📂 prod/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
```

Environments are isolated and managed separately, so this is flexible and safe, but it violates DRY, and it's annoying to manage drift.

#### Multi-Environment, Shared Modules

```
📂 repo-root/
├── 📂 modules/
│   ├── 📂 network/
│   ├── 📂 compute/
├── 📂 envs/
│   ├── 📂 dev/
│   │   ├── main.tf (calls `modules/network`)
│   │   ├── terraform.tfvars
│   ├── 📂 prod/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
```

This is a higher-complexity approach. There is more code-reuse, but there are cross-environment dependencies. You also lose the ability to version per-environment. An update to *prod* will cause a version bump, which also affects *dev*. 

#### Multi-Region

```
📂 repo-root/
├── 📂 terraform/
│   ├── 📂 us-east-1/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
│   ├── 📂 us-west-2/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
```

In theory, this provides good DR - if you can afford multiple AWS environments.

#### Microservices

```
📂 repo-root/
├── 📂 services/
│   ├── 📂 frontend/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
│   ├── 📂 backend/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
```

This segregates microservice infrastructure, but requires careful coordination between microservices, as microservices have implicit dependencies on one another.

#### Multi-Tenant SaaS Platform

```
📂 repo-root/
├── 📂 tenants/
│   ├── 📂 customerA/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
│   ├── 📂 customerB/
│   │   ├── main.tf
│   │   ├── terraform.tfvars
```

This segregates infrastructure per-tenant, but it is expensive to duplicate infrastructure per-tenant.

#### Monorepo

```
📂 repo-root/
├── 📂 modules/              
│   ├── network/
│   ├── compute/
│   └── database/
├── 📂 services/              
│   ├── service-a/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   ├── outputs.tf
│   │   └── backend.tf
│   ├── service-b/
│   └── service-c/
├── 📂 environments/          
│   ├── dev/
│   ├── staging/
│   └── prod/
```

`modules/` re-uses code, `services` themselves have isolated infrastructure configurations, and `environments/` is for overrides. This is DevOps-friendly (easy VCS & CI/CD; shared governance model) - but it is big, requires increased collaboration amongst teams, and may require a complex permissions structure (for instance, if developers are not allowed to update shared production infrastructure, or something).

#### Feature Branch-Based Deployments

```
📂 repo-root/
├── 📂 modules/
│   └── common/            
├── 📂 environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
├── 📂 features/              
│   ├── feature-xyz/
│   │   ├── main.tf
│   │   ├── variables.tf
│   │   └── backend.tf
│   └── feature-abc/
```

Each *feature branch* gets its own configuration. This allows for separate environments per-feature. This is developer-friendly, because developers can safely experiment in isolated sandbox environments, in parallel. *Feature environments* are completely disposable. On the downside, this is expensive, and increases CI/CD time. State file management becomes complex.

#### Multi-Cloud

```
📂 repo-root/
├── 📂 aws/
│   ├── main.tf
│   ├── terraform.tfvars
├── 📂 gcp/
│   ├── main.tf
│   ├── terraform.tfvars
```

Going multi-cloud is generally highly complex, and requires careful inter-cloud communication and security.
    
#### Enterprise Team-Based

```
📂 repo-root/
├── 📂 modules/              
├── 📂 teams/
│   ├── 📂 platform-team/
│   │   ├── networking/
│   │   ├── security/
│   │   └── storage/
│   ├── 📂 devops-team/
│   │   ├── ci-cd/
│   │   └── monitoring/
│   └── 📂 app-team/
│       ├── frontend/
│       └── backend/
├── 📂 environments/
│   ├── dev/
│   ├── staging/
│   └── prod/
```

Infrastructure is segregated by team, and is RBAC-friendly. This can result in lots of potential code duplication, cross-team dependencies can be difficult.


## Errata

* Terraform can handle cross-cloud dependencies.