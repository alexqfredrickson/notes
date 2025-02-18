# Cloud Native Computing Foundation

The Cloud Native Computing Foundation (CNCF) was established in 2015 to advance container technology.

## Cloud Native

Cloud Native is a buzzword used to describe continerized ecosystems deployed to the cloud, with the intention of making applications that are resilient, agile, operational, and observable. 

Resilency means fault-tolerence, with redundancy, failover strategies, the ability to gracefully degrade, self-healing and automated recovery. Agility means the ability to implement agile for application development, and to quickly build and deploy applications. It also implies that microservices and CD pipelines are somehow an agile thing. Operability implies the ease of deployment and running applications, coupled with IaC. Observability is the ability to log, monitor, and trace applications. 

Self-healing is implemented through Deployments with ReplicaSets. Automation can be implemented with configuration management and IaC tools. CI implies code commits kicking off build/test pipelines, followed by a release pipeline. CD implies CI with automated deployment to production. Security-by-default implies zero-trust ("never trust, always verify") using mutual authentication, with secure communication channels between services, and least-privilege for components. Services need simple ways to detect other services on a network with minimal configuration.

Severless applications implement "scale-to-zero", meaning that you don't always need servers to be running for a given application - for example, a Lambda application needs to "warm up" sometimes if it hasn't been invoked in a while, and you don't get charged for idle server time.

## Associated Frameworks

### Open Container Initiative

The Open Container Intiative was a project established by Docker which contains three specifications: the Runtime Specification, the Distribution Specification, and the Image Specification. 

  * The *Image Specification* proposes how a filesystem is bundled into an image (e.g. Docker, `podman`, etc.).
  * The *Runtime Specification* proposes how a "filesystem bundle" is unpacked onto a disk and executed (i.e. *ran*).
  * The *Distribution Specification* proposes how container images are distributed to clients (e.g. the Docker Registry).

### OpenFaaS

OpenFaaS allows "serverless" functions to be more easily deployed to Kubernetes clusters.

### Cilium

Cilium is a container network interface for Kubernetes that provides enhanced eBPF-based networking, observability, and security features. It is intended as a replacement for `iptables`. eBPF runs sandboxed programs in Linux kernels.

### Knative

Knative is a platform-agnostic solution for running serverless deployments.

### KubeVirt

KubeVert is a Kubernetes-based solution for orchestrating virtual machines using Kubernetes.

### OpenTelemetry

Kubernetes leverages OpenTelemetry (or "OTel") to generate, aggregate, export, and orchestrate the collection of telemetry data. OpenTelemetry is an open-source framework.

## CloudEvents

[CloudEvents](https://cloudevents.io/) is a specification for events, with the hopes of standardizing the way that events are consumed. It's a pretty ... lightweight spec. For example:

```json
{
    "specversion" : "1.0",
    "type" : "com.github.pull_request.opened",
    "source" : "https://github.com/cloudevents/spec/pull",
    "subject" : "123",
    "id" : "A234-1234-1234",
    "time" : "2018-04-05T17:31:00Z",
    "comexampleextension1" : "value",
    "comexampleothervalue" : 5,
    "datacontenttype" : "text/xml",
    "data" : "<much wow=\"xml\"/>"
}
```

## Open Policy Agent

[Open Policy Agent (OPA)](https://www.openpolicyagent.org/docs/latest/) is a policy engine that unifies policy enforcement, providing policy-as-code, and APIs to enforce K8S policies. It is a CNCF-graduated project.

## Rook

[Rook](https://rook.io/) is a storage orchestration tool supporting Ceph, which is a distributed storage system. Rook provides self-managing, self-scaling, and self-healing storage services for Kubernetes. Is is a CNCF project.