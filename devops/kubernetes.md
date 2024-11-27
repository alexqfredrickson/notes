# Kubernetes

Kubernetes attempts to solve the following problems:

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

Kubernetes supports `containerd`, `CRI-O`, `Docker Engine`, and `Mirantis Container Runtime` container runtimes.

## Control Plane

The Kubernetes *control plane* is really just a shorthand way of referring to five distinct services: `kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`, and `cloud-controller-manager`.

They usually run on the same box - generally a box that does not house other containers - but each service is distinct and can be scaled out independently of the others, creating a distributed control plane.

The CLI used to communicate with the K8S control plane is calle `kubectl`. It derives configuration from `kubeconfig` files, which organize information about K8S clusters, users, namespaces, and authentication mechanisms.

### `kube-apiserver`

`kube-apiserver` is a REST API used to communicate with a given Kubernetes cluster.

### `etcd`

`etcd` is a [key-value store](https://etcd.io/) that Kubernetes uses to store cluster data. 

### `kube-scheduler`

`kube-scheduler` places K8S pods in K8S nodes, based on resource requirements, hardware/software constraints, and policy constraints. Also based on "affinity specifications", "data locality", and "inter-workfload interference", but [the Kubernetes documentation](https://kubernetes.io/docs/concepts/architecture/#kube-scheduler) is vague on what that actually means.

### `kube-controller-manager`

`kube-controller-manager` runs controller processes. A controller watches the shared cluster state in a loop, via the `kube-apiserver`, and makes changes to the cluster, attempting to move towards desired state.

Controllers are separate processes *logically*, but they run in a single binary/process to reduce complexity.

Controllers include things like "node controllers" which respond when nodes go down, "job controllers" which create pods for one-off tasks, "endpoint slice controllers" which provide links between K8S services and K8S pods, "service account controllers" which create ServiceAccounts for new namespaces, etc. 

### `cloud-controller-manager`

`cloud-controller-manager` provides capabilities to interact with cloud services - delegating some responsibilities to them. For example: "node controllers" in this context will interact with nodes located in the cloud.

## Nodes

Nodes are worker machines that are managed by the K8S control plane. They can be virtual or physical. They run pods.

Nodes contain `kubelet`s (which communicate with the control plane), and a container runtime (which pulls containers from registries, unpacks them, and runs them).

## Pods

Pods (like a pod of whales) is a group of one or more containers running on a node. They share storage and network resources. They share the same specification on how to run the containers. They are co-located. A pod may have init containers. You can inject ephemeral containers into a pod for debugging purposes.

Pods are isolated by way of Linux namespaces, `cgroup`s, etc. - the same ways in which *containers* are generally isolated under-the-hood. They are designed to be ephemeral and disposable, like containers are.

Pods represent either (a) an atomic container or (b) a set of tightly coupled containers.

Pods are generally not created directly. They are created automatically when you are working with workloads.

### Pod Lifecycles

Pods move from `Pending` states, to `Running` states, to `Succeeded` or `Failed` states.

During `Pending`, K8S recognizes the request to set up containers, but they aren't ready yet. Scheduled pods sit in this state for a while.

During `Running`, a pod is bound to a node, and its containers have been created.

During `Succeeded`, the pod's containers have terminated successfully.

During `Failed`, at least one of the pod's containers has terminated with a non-zero exit code.

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

## Production Recommendations

[The Kubernetes documentation](https://kubernetes.io/docs/setup/production-environment/) has some recommendations for how to set up a production K8S environment. It's pretty good general advice.

1. Create a highly-available cluster by:
  * Ensuring that control plane components are not running on worker nodes 
  * Replicating control plane components to multiple nodes
  * Ensuring the API server ingress traffic is load-balanced
  * Ensuring you have enough worker nodes
2. You really don't need to worry about scaling if your traffic patterns are predictable.
3. Use RBAC authorization and set limits on resources that users/workloads can access, via policies and container resources.
4. Consider turnkey solution providers (e.g. AWS EKS) for (a) managed, serverless infrastructure (b) a managed control plane (c) managed worker nodes, or (d) cloud providers to integrate K8S with storage, container registries, third-party auth, etc.
5. Follow K8S guidance for production worker nodes/control planes.
6. Enable authentication against `apiserver` calls via client certificates, or bearer tokens, or authenticating proxies, or HTTP basic auth, or LDAP/Kerberos via plugins.
7. Determine if you want to use RBAC (role-based) or ABAC (attribute-based) access control models for users. In RBAC, you can assign permissions for namespaces ("roles") or clusters ("cluster roles") - then "role bindings" or "cluster role bindings" are attached to users. In ABAC, policies are based on resource attributes.
8. Set up per-namespace quotas on CPU and memory (this is called a "namespace limit").
9. DNS services need to be scaled up if workloads scale up.

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

### Container Failures

If a container fails within a pod:

1. K8S attempts an immediate restart, according to the `restartPolicy` in the pod's specification.
2. K8S continues to restart the continer, using an exponential backoff delay, which is also in the `restartPolicy`.
3. The container enters a `CrashLoopBackOff` state, indicating that the container is in a crash loop.
4. Once a container comes up successfully, the backoff delay is reset.