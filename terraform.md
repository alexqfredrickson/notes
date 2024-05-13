# Terraform

## Syntax

* Terraform `.tf` files can be expressed either in JSON or [HCL](https://github.com/hashicorp/hcl) (or "Hashicorp Configuration Language") format.
* `~>` represents a "maximum version" constraint.
* `main.tf` is just a naming convention, and isn't necessary.

## Concepts

### Backends

* Backends house workspaces, which house state files. 
* Some (but not all) Terraform backends support multiple workspaces.

### Workspaces

Workspaces are like atomic instances of state data.

They are useful in representing individual environments. For example: a "production" workpace versus a "development" workspace.

### Provisioners

Provisioners execute imperative scripts. This breaks Terraform's declarative model, and should be used "as a last resort".

### Providers

Providers are plugins that tell Terraform how to interact with external hosting environments.

## Commands

### `terraform apply`

Applies Terraform *plans* against Terraform *backends*.  This is an asychronous operation that supports up to ten concurrent nodes by default.

### `terraform state`

* `terraform state mv` - `TODO`

### `terraform init`

On Terraform initialization, Terraform will (1) initialize a backend state, (2) download all providers, and (3) download modules referenced from public/private repositories.

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