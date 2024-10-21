# Harness

Harness is a SaaS (app.harness.io) and on-prem CI/CD framework. 

## Delegates

Harness uses *delegates* to communicate with infrastructure. They are like worker nodes, or task orchestrators. They define a delegate as a "service that can run in a local network, or VPC, to connect artifact servers, infrastructure, collaboration, verification, and other providers, with the Harness Manager".

Delegates are containerized, and can be Kubernetes or docker-based.

## Accounts, Organizations, and Projects

Accounts contain multiple organizations, and organizations contail multiple projects. Each project has modules, connectors, pipelines, secrets, templates, delegates, etc.

## Secrets

Harness supports AWS KMS, AWS Secrets Manager, Hashicorp Vault, Azure Key Vault, and GCP Secrets Manager. It also performs its own secrets management.

## Connectors

Connectors integrate the Harness Platform with external systems, such as Kubernetes clusters, secrets managers, SCM repositories, artifact repositories, cloud providers, monitoring systems, logging systems, and ticketing systems.

Connectors need to be supplied with credentials. Authentication is performed by Delegates.

Connectors can be scoped to accounts, organizations, or projects.

## Pipelines

Pipelines have stages, and stages have steps - similar to GitHub Actions.

YAML templates are supported. There is also a WSYIWYG editor.

Pipelines can be stored in the Git repository, or in Harness itself (i.e. like Jenkins' "freestyle jobs").

## Approvals

Approvals can be criteria or metric-based. They can be manual or automated.
