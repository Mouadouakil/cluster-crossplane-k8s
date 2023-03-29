# ClusterDiali crossplane k8s 

## Production-Ready Amazon EKS cluster using Crossplane

This example shows how to create production-ready EKS CLuster using the Crossplane.

### Prerequisites
Please make sure to install the following tools on your machine before moving on.
- Minikube, Kind, or EKS Cluster
- Kubectl


The figure below provides an overview of the configuration of the demo.

<img src="./Assets/conf.png">

First create a namespaces `crossplane-system` and `team-a` with the following command:

After creating the "crossplane-system" namespace, write your AWS credentials to `aws-creds.conf` file and create a secret with the following command.
```
kubectl -n crossplane-system \
    create secret generic aws-creds \
    --from-file creds=./aws-creds.conf
```

After these steps are completed, Install the necessary tools and configuration files with the following commands.
```
helm upgrade --install \
    crossplane crossplane-stable/crossplane \
    --namespace crossplane-system \
    --create-namespace \
    --wait
```
```
kubectl apply \
    --filename crossplane-config/provider-aws.yaml
```
```
kubectl apply \
    --filename crossplane-config/provider-config-aws.yaml
```

> :warning: If the output is "unable to recognize," wait a couple of seconds and re-run the previous command!
```
kubectl apply \
    --filename crossplane-config/provider-helm.yaml
```
```
kubectl apply \
    --filename crossplane-config/provider-kubernetes.yaml
```
```
kubectl apply \
    --filename crossplane-config/config-k8s.yaml
```

Run the following command and wait until all packages are ready.
```
kubectl get pkgrev
```

Now it is time to provision our production-ready EKS cluster. Use the "aws-eks.yaml" file to provision the EKS cluster with the following command.
```
kubectl -n team-a apply -f aws-eks.yaml
```

The last step is to get the “kubeconfig” file to use our EKS cluster. Run the following commands to get and set the “kubeconfig” file.

```
kubectl --namespace crossplane-system \
    get secret team-a-eks-cluster \
    --output jsonpath="{.data.kubeconfig}" \
    | base64 -d >kubeconfig.yaml

export KUBECONFIG=$PWD/kubeconfig.yaml
```

## Destroy

Run the following command to destroy your resources.

```
unset KUBECONFIG

kubectl --namespace team-a delete \
    --filename examples/aws-eks.yaml
```

# Using Crossplane to provision a GKE cluster


## Quickstart

This code creates installs Crossplane on an existing GKE cluster, and provisions a new GKE cluster using Crossplane.

Feel free to skip running `1-gcp_config.sh` if you're installing Crossplane elsewhere.
### Pre-requisites

* A Google Cloud project
* `glcoud` CLI
* An existing GKE cluster (Crossplane will be installed here)
* The [`envsubst`](https://kb.novaordis.com/index.php/Envsubst) tool installed on your local machine


### 1- Update the env_vars.sh file

Replace the values in `<...>` with your own values, and save.

### 2- Run the scripts

Run the following script files to:

* Install Crossplane on an existing Kubernetes cluster in Google Cloude
* Create configure the Crossplane GCP Provider
* Provision a new GKE Cluster and Node Pool using Crossplane

```bash
./scripts/1-gcp_config.sh
./scripts/2-install_crossplane.sh
./scripts/3-configure_gcp_provider.sh
./scripts/4-create_gke_cluster.sh
```

## References

* [Crossplane Getting Started Docs (for v1.2.0)](https://crossplane.io/docs/v1.2/getting-started/install-configure.html)
* [Crossplane GKECluster API docs for GCP](https://doc.crds.dev/github.com/crossplane/provider-gcp/container.gcp.crossplane.io/GKECluster/v1beta1@v0.16.0)
* [Crossplane NodePool API docs for GCP](https://doc.crds.dev/github.com/crossplane/provider-gcp/container.gcp.crossplane.io/NodePool/v1alpha1@v0.16.0)
* [Crossplane provider-gcp on Github (contains examples)](https://github.com/crossplane/provider-gcp/blob/master/examples/gke/gke.yaml)




