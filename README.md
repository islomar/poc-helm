# poc-helm
- Playground for learning about **Helm**: the package manager for k8s
- https://helm.sh/
- https://github.com/helm/helm
- Helm = timón
- Related resources:
  - https://github.com/islomar/my-notes/tree/master/kubernetes

## Courses and training
- https://www.linkedin.com/learning/kubernetes-package-management-with-helm
- https://acloudguru.com/
- https://training.linuxfoundation.org/

## General
- Without Helm: `kubectl apply -f <xxx>` all over.

## Helm architecture
- https://helm.sh/docs/topics/architecture/
- Helm is a tool for managing Kubernetes packages called **charts**.
- For Helm, there are three important concepts:
  1. The **Chart** is a bundle of information necessary to create an instance of a Kubernetes application.
  2. The **config** contains configuration information that can be merged into a packaged chart to create a releasable object.
  3. A **release** is a running instance of a **chart**, combined with a specific **config**.
- Three big concepts:
  1. A **Chart** is a Helm package. It contains all of the resource definitions necessary to run an application, tool, or service inside of a Kubernetes cluster. Think of it like the Kubernetes equivalent of a Homebrew formula, an Apt dpkg, or a Yum RPM file.
  2. A **Repository** is the place where charts can be collected and shared. It's like Perl's CPAN archive or the Fedora Package Database, but for Kubernetes packages.
  3. A **Release** is an instance of a chart running in a Kubernetes cluster. One chart can often be installed many times into the same cluster. And each time it is installed, a new release is created. Consider a MySQL chart. If you want two databases running in your cluster, you can install that chart twice. Each one will have its own release, which will in turn have its own release name.
- Helm is an executable which is implemented into two distinct parts:
  - The Helm **client**
  - The Helm **library**
    - It interfaces with the Kubernetes API server
    - The library uses the Kubernetes client library to communicate with Kubernetes (using REST+JSON).
    - It stores information in Secrets located inside of Kubernetes. 
    - It does not need its own database.
- The Helm client and library is written in the Go programming language.
- Configuration files are, when possible, written in YAML.

## Quickstart
- `helm env`: to see all your Helm environment variables
- Whenever you install a chart, a new release is created. So one chart can be installed multiple times into the same cluster.
- `helm list`: to see what has been released using Helm (list of all deployed releases).
- `helm search hub <chart_name_to_search>` searches the [Artifact Hub](https://artifacthub.io/packages/search?kind=0)
- `helm search repo <repo_name_to_search>` searches the repositories that you have added to your local helm client (with `helm repo add <repo_name> <repo_url>`).
  - `helm repo list`
  - `helm repo remove`
  - `helm repo udate`: Because chart repositories change frequently, at any point you can make sure your Helm client is up to date
- `helm install <chart_name_to_install> --generate-name` or `helm install <release_name> <repo/chart_name_to_install>`
  - Helm does not wait until all of the resources are running before it exits. Many charts require Docker images that are over 600M in size, and may take a long time to install into the cluster.
- `helm status <release_name>`: to keep track of a release's state
- To see what options are configurable on a chart, use `helm show values`
  - You can then override any of these settings in a YAML formatted file (e.g. `values.yaml`), and then pass that file during installation.
  - helm install -f values.yaml bitnami/wordpress --generate-name`
- `helm show chart <chart_name>`
- `helm show helm`
- `helm upgrade -f values.yaml <release_name> <chart_name_to_upgrade>`: An upgrade takes an existing release and upgrades it according to the information you provide
- `helm get values <release_name>`: The helm get command is a useful tool for looking at a release in the cluster.
- `helm rollback <release_version> <revision>`: if something does not go as planned during a release, it is easy to roll back to a previous release
  - A release version is an incremental revision. Every time an install, upgrade, or rollback happens, the revision number is incremented by 1. The first revision number is always 1. And 
  - we can use `helm history [RELEASE]` to see revision numbers for a certain release.
- Options for [install/upgrade/rollback](https://helm.sh/docs/intro/using_helm/#helpful-options-for-installupgraderollback)
  - `--timeout`
  - `--wait`
- `helm uninstall <release_name>`: uninstall a release from the cluster
- `helm delete <release_name> -n <namespace>`
- You can create your own Charts: https://helm.sh/docs/topics/charts/

## [Best practices](https://helm.sh/docs/chart_best_practices)
- Chart names must be lower case letters and numbers.
- YAML files should be indented using two spaces (and never tabs).
- Values:
  - Variable names should begin with a lowercase letter, and words should be separated with camelcase
  - YAML: In most cases, flat should be favored over nested.
  - it's often better to structure your values file using maps.
- Templates
  - Template file names should use dashed notation.
  - Each resource definition should be in its own template file.
  - Template file names should reflect the resource kind in the name.

## Related tools
- https://hub.docker.com/r/alpine/helm
- Helm completion in zsh: https://helm.sh/docs/helm/helm_completion_zsh/#helm-completion-zsh
- Helm Chart repositories: https://artifacthub.io/packages/search?kind=0
- [Terraform Helm provider](https://registry.terraform.io/providers/hashicorp/helm/latest/docs)
  - Tutorial: https://developer.hashicorp.com/terraform/tutorials/kubernetes/helm-provider?in=terraform%2Fkubernetes
- [Helm plugins](https://helm.sh/docs/topics/plugins/): 
  - A Helm plugin is a tool that can be accessed through the helm CLI, but which is not part of the built-in Helm codebase.
  - They provide a way to extend the core feature set of Helm, but without requiring every new feature to be written in Go and added to the core tool.
  - Helm plugins live in $HELM_PLUGINS
  - [helm-diff](https://github.com/databus23/helm-diff):This is a Helm plugin giving you a preview of what a `helm upgrade would change. It basically generates a diff between the latest deployed version of a release and a helm upgrade --debug --dry-run`.
  - [kube-state-metrics (KSM)](https://github.com/kubernetes/kube-state-metrics) is a simple service that listens to the Kubernetes API server and generates metrics about the state of the objects.
    - https://artifacthub.io/packages/helm/bitnami/kube-state-metrics
    - `helm repo add bitnami https://charts.bitnami.com/bitnami`
    - `helm repo update`
    - `helm repo list`
    - `kubectl create ns metrics`
    - `kubectl get ns`
    - `helm install kube-state-metrics bitnami/kube-state-metrics -n metrics`
    `helm ls -n metrics`
    - `kubectl get all -n metrics`: to see all the k8s resources created by the Helm Chart
    - `kubectl logs <pod_name> -n metrics`
    - `kubectl port-forward svc/kube-state-metrics 8080:8080 -n metrics`: forward service to a local port that I can access in order to check the metrics
      - http://localhost:8080
    - `helm upgrade kube-state-metrics bitnamic/kube-state-metrics --version 0.4.0 -n metrics`: to use a specific Chart version
    - `kubectl delete ns metrics`
    - `helm history first-chart .`