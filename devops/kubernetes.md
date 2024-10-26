# Kubernetes

Kubernetes atempts to solve the following problems:

 * Service discovery: exposing containers via DNS or IP
 * Load balancing
 * Storage orchestration: allowing local storage, cloud providers, etc.
 * Automated deployments and rollbacks: using desired state configurations
 * Automatic bin packing: you give Kuberenetes nodes, tell it how much CPU/RAM each container needs, and Kubernetes figures it out
 * Self-healing: restarting failing containers
 * Secrets/configuration management: for sidecar OAuth tokens, SSH keys, password, etc. 
 * Batch execution-based management: if desired
 * Horizontal autoscaling
 * IPv4/IPv6 dual-stack address allocation for pods/services

## Pods

Pods (like a pod of whales) is a group of one or more containers. They share storage and network resources. They share the same specification on how to run the containers. They are co-located. A pod may have init containers. You can inject ephemeral containers into a pod for debugging purposes.

Pods are isolated by way of Linux namespaces, `cgroup`s, etc. - the same ways in which *containers* are generally isolated under-the-hood. They are designed to be ephemeral and disposable, like containers are.

Pods represent either (a) an atomic container or (b) a set of tightly coupled containers.

Pods are generally not created directly. They are created automatically when you are working with workloads.

## Workloads

Workloads are applications that run on Kubernetes. They run in sets of pods. Workload resource manage those pods.

Workload resources include *deployments*, *replica sets*, *stateful sets*, *daemon sets*, *jobs*, and *cron jobs*.

*Replication controllers* are deprecated.

### Deployments

Deployments manage non-stateful application workloads. They provide declarative updates for Pods and ReplicaSets. You describe a desired state, and a Deployment Controller changes the pods to the desired state. 

You would use a Deployment to roll out ReplicaSets, declare new pod state, roll back to earlier deployment revision, scale up a deploymnet to facilitate more load, pause deployments, or clean up old ReplicaSets.

### ReplicaSets

ReplicaSets try to maintain sets of replica Pods. It tries to guarantee availability of those identical Pods. They are managed by Deployments.

ReplicaSets define which Pods they can aquire, a replica count, and a template specifying how Pods should be provisioned.

### StatefulSets

StatefulSets are used when Pods need to be "stateful", in the sense that ordering and uniqueness matters. The pods that StatefulSets create have "sticky identities" with persistent identifiers.

You would use a StatefulSet if you need stable network identifiers, stable storage, ordered deployments/scaling, or ordered rolling updates.

### DaemonSets

DaemonSets ensure that all Nodes run some kind of Pod. DaemonSets guarantee that new Nodes will have a given Pod.

You would use a DaemonSet if you need to run a cluster storage, log collection, or node monitoring daemon on every node.

### Jobs

Jobs are one-off Pods that run to completion, and then terminate.

### CronJobs

CronJobs are Jobs that run on a schedule.

## Associated Frameworks

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