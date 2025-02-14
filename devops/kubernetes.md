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

Workloads are applications that run on Kubernetes. They run in sets of pods. Workload resources manage those pods.

Workload resources include *deployments*, *replica sets*, *stateful sets*, *daemon sets*, *jobs*, and *cron jobs*.

*Replication controllers* are deprecated.

### Deployments

A Deployment specifies how an application should run, and the amount of replicas that should run. Deployments manage ReplicaSets.

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

## Services

Services expose network applications as single outward-facing endpoints. 

Pods have IP addresses, but those change if those pods are destroyed - so if pods are dependent on one another, services are needed to ensure a loose coupling between those components. 

This is an example of a service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  selector:
    app.kubernetes.io/name: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 9376
```

`port` represents the outward-facing TCP port. Requests to port 80 are routed to the container's `targetPort` on port 9376.

## Cluster Administration

### Node Autoscaling

Nodes can be auto-scaled with Autoscalers. Two options are endorsed by the autoscaling special interest group (SIG) for K8S: Cluster Autoscaler and Karpenter. [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/README.md) automatically adjusts the size of a Kubernetes cluster if (a) pods failed to run in a cluster due to insufficient resources or (b) nodes are underutilized and their pods can be placed in other nodes.


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

### Container Failures

If a container fails within a pod:

1. K8S attempts an immediate restart, according to the `restartPolicy` in the pod's specification.
2. K8S continues to restart the continer, using an exponential backoff delay, which is also in the `restartPolicy`.
3. The container enters a `CrashLoopBackOff` state, indicating that the container is in a crash loop.
4. Once a container comes up successfully, the backoff delay is reset.

## Cloud Native

Cloud Native is a buzzword used to describe continerized ecosystems deployed to the cloud, with the intention of making applications that are resilient, agile, operational, and observable. 

Resilency means fault-tolerence, with redundancy, failover strategies, the ability to gracefully degrade, self-healing and automated recovery. Agility means the ability to implement agile for application development, and to quickly build and deploy applications. It also implies that microservices and CD pipelines are somehow an agile thing. Operability implies the ease of deployment and running applications, coupled with IaC. Observability is the ability to log, monitor, and trace applications. 

Self-healing is implemented through Deployments with ReplicaSets. Automation can be implemented with configuration management and IaC tools. CI implies code commits kicking off build/test pipelines, followed by a release pipeline. CD implies CI with automated deployment to production. Security-by-default implies zero-trust ("never trust, always verify") using mutual authentication, with secure communication channels between services, and least-privilege for components. Services need simple ways to detect other services on a network with minimal configuration.

Severless applications implement "scale-to-zero", meaning that you don't always need servers to be running for a given application - for example, a Lambda application needs to "warm up" sometimes if it hasn't been invoked in a while, and you don't get charged for idle server time.

## Containers

### History

In the 1960s-70s, CP/CMS was a mainframe operating system that allowed for "time sharing", which was a method that allowed multiple users to access the mainframe's CPU/storage concurrently. Each user got their own virtual operating system within the machine.

In 1979, `chroot` was developed for Unix, which allowed you to change the root directory for a running process and its children - so the process is sandboxed insofar as it doesn't know the rest of the file system exists. However, `chroot` did not sandbox hostnames and IP addresses, it can't run as `su`, and the directories must be owned by `root:root`.

In 2000, FreeBSD introduced Jails, which allowed for a Unix system to be partitioned with separate users, processes, file systems, and networking stacks - but it was difficult to use.

Sun Solaris and HP-UX were early forerunners of multiple isolated virtual environments. Later, VMWare developed hypervisors like ESXi, which allow multiple virtual machines to run on a single physical machine. 

In 2002, the Linux kernel added support for `namespaces`, which partition kernel resources across:

  1. `user`, which provide privilege isolation and user ID segregation across sets of processes
  2. `pid`, which allowed processes in a namespace to have their own unique process IDs
  3. `network`, which allowed processes in a namepace to have their own networking stacks, and which allowed processes to communicate with one another
  4. `mount`, which allowed processes to mount filesystems
  5. `uts`, or Unix Time Sharing, which allowed for processes to have isolated hostnames/domains
  6. `ipc`, which set up message queues for processes
  
In 2006, Google developed `cgroups`, also known as *process containers*. These isolate resource limits (CPU / memory / network allocation / I/O), priority allocations, monitoring/reporting of resource limits, and process control (start/stop/frozen/restarted).

In 2010, Docker was founded, leveraging `namespaces` and `cgroups`, to define container runtimes, which share the same kernel as the host operating system. Containers receive their own users / hostnames / IPs / mounts / resource allocations.
