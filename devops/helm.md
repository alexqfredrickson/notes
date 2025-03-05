# Helm

Helm is a package manager (like `yum` or `apt`) for Kubernetes.  It defines *charts* (Kubernetes resources), packages them into `.tgz` files, and sends them to chart repositories.  Helm installs packages to Kubernetes clusters, and manages the release cycles of those charts.


## Charts

Charts are files that describe resources that will be deployed to Kubernetes - like `memcached` pods, full web application stacks, databases, etc.  Charts are semver-versioned and packaged.

Charts have two `type`s: `application` and `library`. `application`-type charts are the default. A `library`-type chart is non-installable. 

Charts may have dependencies listed in `Chart.yaml`. Charts which have *dependent* charts will also install their dependencies on `helm install`.

Charts may be parameterized using `values.yaml` files. Values can also be supplied to a chart's dependencies.

Charts can define *custom resource definitions* (CRDs), which are a Kubernetes construct used to declare custom resource types. They go into the `crds/` directory where a Chart is found. There are some limitations here: CRDs are never reinstalled, nor installed on upgrade or rollback, and never deleted. If a CRD in K8S is deleted, all of the CRD's contents across *all cluster-wide namespaces* are also deleted.

Chart repositories are HTTP servers that house packaged charts. GCP, S3, Cloudsmith, GitHub Pages, JFrog Artifactory, ChartMuseum, and GitLab can be used as chart repositories. OCI repositories such as AWS ECR, DockerHub, Google Artifact Repository, etc. can also be used. A repository must have an `index.yaml` which lists packages served by the repository, and metadata that can retrieve and verify packages.

The `helm` CLI can interact with charts:

  * `helm create [chartname]` creates the scaffolding for a chart directory.
  * `helm package [chartname]` packages a chart into an archive.
  * `helm lint [chartname]` lints charts.


## Chart Hooks

Chart hooks can do things like run pre/post-install jobs, load secrets before install, perform database backups, etc.

Chart hooks are normal charts with special annotations: `"helm.sh/hook"`, `"helm.sh/hook-weight"`, and `"helm.sh/hook-delete-policy"`.

Hooks include: `pre/post-install`, `pre/post-delete`, `pre/post-upgrade`, `pre/post-rollback`, and `test` (for when `helm test` is invoked).


## Chart Tests

Charts support tests for things like making sure user credentials work, checking that services are up and running, etc.  Tests are Helm chart hooks (i.e. charts with `"helm.sh/hook"` attributes).

The Helm documentation gives an example of a Pod which performs a `wget` from a `wget` container.

`helm test [chartname]` is used to execute one such "test".


## Provenance

When packages are created, they can be signed with private keys. This creates `.tgz.prov` files.  When accompanied by a keychain, you can validate that charts are signed by trusted parties, verify package integrity, and match charts with provenance data.


## Plugins

Helm supports plugins.

