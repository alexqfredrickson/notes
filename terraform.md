# Terraform

## Syntax

* Terraform `.tf` files can be expressed either in JSON or [HCL](https://github.com/hashicorp/hcl) (or "Hashicorp Configuration Language") format.
* `~>` represents a "maximum version" constraint.
* `main.tf` is just a naming convention, and isn't necessary.

## Concepts

### Backends

* State files live in Terraform workspaces, which (in-turn) live in Terraform backends.
* Some (but not all) Terraform backends support multiple workspaces - for instance, if you need a different workspace for "production" versus "development".

### Workspaces

A Terraform workspace is like an atomic instance of state data.

### Provisioners

Terraform provisioners invoke scripts imperatively, which generally breaks Terraform's declarative model.

### Providers

Terraform providers 

## Commands

### `terraform apply`

Applies Terraform *plans* against Terraform *backends*.  This is an asychronous operation that supports up to ten concurrent nodes by default.

### `terraform state`

* `terraform state mv` - `TODO`

### `terraform init`

On Terraform initialization, Terraform will (1) initialize a backend state, (2) download all providers, and (3) download modules referenced from public/private repositories.

