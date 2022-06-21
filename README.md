# agroCD  

What is Argo CD?
Argo CD is a GitOps tool to automatically synchronize the cluster to the desired state defined in a Git repository. Each workload is defined declarative through a resource manifest in a YAML file. Argo CD checks if the state defined in the Git repository matches what is running on the cluster and synchronizes it if changes were detected.

For example, instead of manually running CLI commands to update Kubernetes resources with kubectl apply or helm upgrade, we would update a YAML file in our Git repository that contains an Application manifest. Argo CD periodically checks this manifest for changes and will automatically synchronize resources that are defined in it with the ones that are running on our cluster.

A connection to the cluster, either from the developers laptop or from a CI/CD system, is no longer needed as changes are pulled from the Git repository by a Kubernetes Operator running inside the cluster.

#### charts/argo-cd/values.yaml

- We disable the dex component that is used for integration with external auth providers
- We start the server with the --insecure flag to serve the Web UI over http (This is assuming we’re using a local k8s server without TLS setup)
- We add the Argo CD Helm repository to the repositories list to be used by applications

#### Before we install the chart we need to generate a Chart.lock file:

- helm repo add argo-cd https://argoproj.github.io/argo-helm
- helm dep update charts/argo-cd/


#### This will generate two files:

- Chart.lock
- charts/argo-cd-4.2.2.tgz

The tgz file is the downloaded dependency and not required in our Git repository, we can therefore exclude it. Argo CD will download the dependencies by itself based on the Chart.lock file.

We exclude it by creating a .gitignore file in the chart directory:

- echo "charts/" > charts/argo-cd/.gitignore


## Installing our Argo CD Helm chart

- helm repo add argo https://argoproj.github.io/argo-helm
- helm install argo-cd argo/argo-cd

## Accessing the Web UI
The Helm chart doesn’t install an Ingress by default, to access the Web UI we have to port-forward to the argocd-server service:

- kubectl port-forward service/argo-cd-argocd-server -n default 8080:443

- kubectl -n default get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d

- We can then visit http://localhost:8080 to access it



### Deploy ArgoCD 

- helm template apps/ | kubectl apply -f -

### Once the Argo CD application is synced it can now manage itself and we can, demonstrate how to deploy a Helm chart with Argo CD, we’ll add Echo-Server to our cluster.

- We’re using chart instead of path to install a Helm chart from a different Helm repository
- The targetRevision is the specific chart version that we want to install
- The repoURL is set to the ealenn.github.io Helm chart repository

- To deploy the application all we have to do is push the manifest to our Git repository:


### Note: Argo CD will not use helm install to install charts. It will render the chart with helm template and then apply the output with kubectl. This means we can’t run helm list on a local machine to get all installed releases.