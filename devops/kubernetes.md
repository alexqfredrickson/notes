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

Kubernetes supports `containerd`/`runc`, `CRI-O`, `Docker Engine`, and `Mirantis Container Runtime` container runtimes.

## Nodes

Nodes are worker machines that are managed by the Kubernetes Control Plane. They can be virtual or physical. They run pods.

Nodes contain `kubelet`s (which communicate with the control plane), and a container runtime (which pulls containers from registries, unpacks them, and runs them). Kubelets also run on the control plane.

## Pods

Pods (like a *pod* of whales) is a group of one or more containers running on a node (i.e. any Kubernetes worker machine).

Each container within a pod shares network resources provided by the pod. Every pod is assigned a unique IP address in the cluster.

Containers within a pod can communicate with one another via IPC (inter-process communication).

Pods share storage resources. They share the same specification on how to run the containers. They are co-located. A pod may have init containers. You can inject ephemeral containers into a pod for debugging purposes.

Pods are isolated by way of Linux namespaces, `cgroup`s, etc. - the same ways in which *containers* are generally isolated under-the-hood. They are designed to be ephemeral and disposable, like containers are.

Pods are generally not created directly. They are created automatically when you are working with workloads.

### Static Pods

If a `kubelet` daemon creates a pod on some node, and doesn't want `kube-apiserver` (i.e. the Kubernetes control plane) to observe it or manage it, then it is called a *static pod*. A better term might be a *protected pod*. 

### Pod Lifecycles

Pods move from `Pending` states, to `Running` states, to `Succeeded` or `Failed` states.

During `Pending`, K8S recognizes the request to set up containers, but they aren't ready yet. Scheduled pods sit in this state for a while.

During `Running`, a pod is bound to a node, and its containers have been created.

During `Succeeded`, the pod's containers have terminated successfully.

During `Failed`, at least one of the pod's containers has terminated with a non-zero exit code.

### Container Failures

If a container fails within a pod:

1. K8S attempts an immediate restart, according to the `restartPolicy` in the pod's specification.
2. K8S continues to restart the continer, using an exponential backoff delay, which is also in the `restartPolicy`.
3. The container enters a `CrashLoopBackOff` state, indicating that the container is in a crash loop.
4. Once a container comes up successfully, the backoff delay is reset.

## Kubelets

Kubelets run on control planes and nodes. They maintain pods. They use *pod specs*, which are descriptions of pods in YAML/JSON format. They receive pod specs via API or by monitoring directories of YAML files (i.e. `/etc/kubernetes/manifests`).

In a control plane context, `kubelet`s receive pod specs, and pass them to high-level container runtimes (such as `containerd`), which pass those off to low-level container runtimes (such as `runc`), which create *static pods*.

## Control Plane

The Kubernetes *control plane* is a shorthand way of referring to five distinct services: `kube-apiserver`, `etcd`, `kube-scheduler`, `kube-controller-manager`, and `cloud-controller-manager`.

They usually run on the same box - generally a box that does not house other containers - but each service is distinct and can be scaled out independently of the others, creating a distributed control plane.

The CLI used to communicate with the K8S control plane is calle `kubectl`. It derives configuration from `kubeconfig` files, which organize information about K8S clusters, users, namespaces, and authentication mechanisms.

The control plane contains a `kubelet` which communicates with other `kubelet`s on Nodes.

### `kube-apiserver`

`kube-apiserver` is a REST API used to communicate with a given Kubernetes cluster. Its data is stored in `etcd`. It runs in a static pod in the control plane. `kube-apiserver` communicates with `kubelet`s on nodes. `kubectl` frontloads requests to `kube-apiserver`. The API server is generally replicated across multiple static pods, and load balanced.

`kube-apiserver` is interactable via `kubectl`, `kubeadm`, and REST API.

`kube-apiserver` uses an *admission controller* to enforce policies, modify resources, enforce quotas, set defaults, validate configurations, manage namespaces, accept/reject requests, etc.

Unremarkably, requests generally come into the server via HTTPS port 6443, routes the request to some part of the API via some HTTP verb, checks authentication, and checks authorization. *Then* the admission controller determines whether or not to serve the request. After that, the request is validated and the request is handled.

CRDs (custom resource definitions) extend the Kubernetes API by defining new resource types - such as an InnoDB MySQL cluster. 

There's a decent OpenAPI/Swagger document out there for this.

### `etcd`

`etcd` is a [key-value store](https://etcd.io/) that Kubernetes uses to store all cluster data. It runs in a static pod in the control plane. 

In a production context, it is recommended to run multiple instances of `etc`, at any odd number. 

### `kube-scheduler`

`kube-scheduler` places Kubernetes pods in nodes, based on: resource requirements, hardware/software constraints, and policy constraints. Also based on "affinity specifications", "data locality", and "inter-workfload interference", but [the Kubernetes documentation](https://kubernetes.io/docs/concepts/architecture/#kube-scheduler) is vague on what that actually means. It runs in a static pod in the control plane. 

### `kube-controller-manager`

`kube-controller-manager` runs controller processes. A controller watches the shared cluster state in a loop, via the `kube-apiserver`, and makes changes to the cluster, attempting to move towards desired state.

Controllers are separate processes *logically*, but they run in a single binary/process to reduce complexity.

Controllers include things like "node controllers" which respond when nodes go down, "job controllers" which create pods for one-off tasks, "endpoint slice controllers" which provide links between K8S services and K8S pods, "service account controllers" which create ServiceAccounts for new namespaces, etc. 

### `cloud-controller-manager`

`cloud-controller-manager` provides capabilities to interact with cloud services - delegating some responsibilities to them. For example: "node controllers" in this context will interact with nodes located in the cloud.

## Workloads

Workloads are applications that run on Kubernetes. They run in sets of pods. Workload resources manage those pods.

Workload resources include *deployments*, *replica sets*, *stateful sets*, *daemon sets*, *jobs*, and *cron jobs*.

*Replication controllers* are deprecated.

### Deployments

A Deployment specifies how an application should run, and the amount of replicas that should run. Deployments can be annotated just like `git` commits. 

Deployments manage non-stateful application workloads. They provide declarative updates for Pods and ReplicaSets. You describe a desired state, and a Deployment Controller changes the pods to the desired state. 

You would use a Deployment to roll out ReplicaSets, declare new pod state, roll back to earlier deployment revision, scale up a deploymnet to facilitate more load, pause deployments, or clean up old ReplicaSets.

Deployments manage ReplicaSets. They automatically create ReplicaSets. Deleting a Deployment removes its ReplicaSets as well.

Deployments have strategies. They perform rolling updates to replicas as a default setting. By default, a rolling deployment will increase the number of pods by 25% above the desired amount, in order to maintain application availability. 

If a deployment is rolled back, the original ReplicaSet for the deployment is re-used and becomes the latest revision.

#### CoreDNS

CoreDNS is a type of Kubernetes deployment which depends on `kube-controller-manager`.

### ReplicaSets

ReplicaSets try to maintain sets of replica Pods. It tries to guarantee availability of those identical Pods. They are managed by Deployments. They are automatically created by deployments.

ReplicaSets define which Pods they can aquire, a replica count, and a template specifying how Pods should be provisioned.

### StatefulSets

StatefulSets are used when Pods need to be "stateful", in the sense that ordering and uniqueness matters. The pods that StatefulSets create have "sticky identities" with persistent identifiers.

You would use a StatefulSet if you need stable network identifiers, stable storage, ordered deployments/scaling, or ordered rolling updates.

### DaemonSets

A DaemonSet is a configuration that requires that all nodes run a specific pod. It ensures that new nodes will contain some pod or another (i.e. a *daemon*, sort of).

You would use a DaemonSet if you need to run a cluster storage, log collection, or node monitoring daemon on every node.

#### `kube-proxy`

`kube-proxy` is a K8S network proxy, running on each node. It performs TCP, UDP, and SCTP (stream control transmission protocol) stream forwarding. It is allegedly a type of DaemonSet, although [the documentation makes no obvious mention of this](https://kubernetes.io/docs/reference/command-line-tools-reference/kube-proxy/). It is a normal pod, not a static pod. It communicates with `kube-apiserver`, and provides DNS resolution.

### Jobs

Jobs are like "supervisors" for one-off Pods that run to completion, and then terminate. They track task progress and retry, if-needed.

Jobs have a `completions` attribute that sets the number of successfully finished jobs required in order to signal the success of all pods. If `1`, parallelism is liimted to one pod. If `null`, any pod will signal the success of all pods.

Jobs also have a `parallelism` attribute for the amount of pods that should be spawned to work on a job.

### CronJobs

CronJobs create Jobs on a schedule - for reporting, system maintenance tasks, etc.

## Services

Services expose network applications as single outward-facing endpoints. In other words, they assign static IP addresses and DNS names to Pods. 

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

There are four type of services:

  1. `ClusterIP`, which exposes a *private* service on a cluster-internal IP address. The service is only reachable from within the cluster. An Ingress or Gateway is required to expose the service to the public internet.
  2. `NodePort`, which exposes the service on the Node's IP, on a static port. Kubernetes will still set up a cluster IP address, in order to make the node port available. If the node's IP addresses are externally accessible, they can be accessed publically.
  3. `LoadBalancer`, which exposes the service externally, using some load balancer. You have to set up your own load balancer.
  4. `ExternalName`, which maps a service to an `externalName` field, which may be a hostname (or whatever). The cluster's DNS server will return a `CNAME` record with this value. This allows a service to be accessed through an alternate name, for redirecting traffic, or for better user experience.

A *headless service* can be configured as well, if `.spec.clusterIP: None` is specified. They provide DNS implementations with no proxies, so a Pod will handle its own traffic. Pods are still given IP addresses, but no cluster IPs are allocated. Headless services interface with service discovery mechanisms.  `kube-proxy` doesn't handle them. No load balancing is performed on them. This allows direct access to Pods without using Service-based IP addresses.

Service Endpoints are IP addresses and ports assigned to the Pods that a Service points to.

## Namespaces

Namespaces isolate groups of resources (such as pods) in a cluster. They must be unique.

The default namespaces are `default`, `kube-node-lease` (for Lease objects, and for `kubelet`s to send heartbeats to detect node failure via the control plane), `kube-public` (if resources need to be globally public), and `kube-system` (for objects created by Kubernetes).

ResourcesQuotas set budgets on usage of CPU/memory resources within a namespace.

LimitRanges set resource limits at the pod/container level.

Namespaces can have RBAC access policies associated with them.


## Cluster Administration

### Node Autoscaling

Nodes can be auto-scaled with Autoscalers. Two options are endorsed by the autoscaling special interest group (SIG) for K8S: Cluster Autoscaler and Karpenter. [Cluster Autoscaler](https://github.com/kubernetes/autoscaler/blob/master/cluster-autoscaler/README.md) automatically adjusts the size of a Kubernetes cluster if (a) pods failed to run in a cluster due to insufficient resources or (b) nodes are underutilized and their pods can be placed in other nodes.

## Configuration

### ConfigMap

ConfigMaps store non-secret data in key-value pairs. They can be referenced as environment variables via Pods. They can also be sent via CLI arguments, or stored as configuration files in a volume.

They can referenced via the `envFrom` property in a Pod specification, under `spec -> containers -> envFrom -> configMapRef`, to send K8S env vars to a container.

ConfigMaps can be made immutable with `immutable: true`.

### Secrets

Secrets contain sensitive data that should not be stored in container images or pod specifications. They are similar to ConfigMaps. They are referenced via `envFrom -> secretRef`.

Secrets are stored unencrypted by default in the API server's `etcd` store. You need to enable encryption-at-rest. 

### Labels

Labels are used to tag Kubernetes objects.

They allow for "label selection" to filter on related objects - for instance, services may use label selectors to identify relevant pods.

## `kubectl`

`kubectl` is a command-line utility that accesses the Kubernetes AI.

`kubectl run [pod]` runs a pod. This does not require a YAML configuration.

`kubectl get pods` lists all pods in a cluster.

`kubectl get namespaces` or `kubectl get ns` lists all namespaces in a cluster.

`kubectl logs [pod]` queries logs for a given pod. It is possible to see logs from crashed containers with the `-p` (previous) flag

`kubectl exec -it [pod] bash` will execute a command in a pod (like `bash`).

`kubectl delete [podn-1] [podn]` will delete pod(s).

`kubectl explain pod` shows a man page for a given pod specification.

`kubectl describe [pod]` returns general details about a pod (i.e. one or more containers that are running together), or some other K8S object.

`kubectl create` is imperative (and may result in configuration drift), whereas `kubectl apply` is declarative.

`kubectl api-resources` lists short names for API commands (like how `kubectl get namespaces` can be shortened to `kubectl get ns`).

`kubectl rollout status` shows you the progress of a deployment.

`kubectl scale` will adjust the number of replicas for a given deployment.

`kubectl expose [deployment]` will expose a deployment as a service.

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
