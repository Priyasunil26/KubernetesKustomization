# Load Balancer
We currently support `Nginx` and `Istio` as load balancers in Bold BI. Nginx is used as the default reverse proxy for Bold BI.

## Nginx

1. On-premises cluster (e.g., kind, K0s Cluster, rancher, kubeadm) - Refer this [link](https://kubernetes.github.io/ingress-nginx/deploy/#rancher-desktop)

2. To configure Bold BI with Ingress, you need to install the [Nginx Ingress controller](https://kubernetes.github.io/ingress-nginx/deploy/) in your cluster. Please refer to the following instructions and execute the command accordingly.

| Cloud Provider                  | Installation Command                                                                                       |
|---------------------------------|--------------------------------------------------------------------------------------------------------|
| Azure Kubernetes Service (AKS)  | kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml |
| Google Kubernetes Engine (GKE)  | kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml                                         |
| Elastic Kubernetes Service (EKS)| kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/aws/deploy.yaml  |


The latest Nginx Ingress controller has a configuration change where the default value for `allow-snippet-annotations` is set to `false`. To fix this, you need to edit the Nginx Ingress ConfigMap file and set the value to `true`.

![Set Snippet value true](images/snippet-true.png)

Use the following command to edit the ConfigMap:
    
    kubectl edit cm ingress-nginx-controller -n ingress-nginx

Run the following command to get the ingress IP address for Nginx.

    kubectl get service/ingress-nginx-controller -n ingress-nginx

## Istio

To configure Bold BI with Istio, install the [Istio Ingress Gateway](https://istio.io/latest/docs/setup/install/) in your cluster. Please refer to the corresponding reference links for installation instructions.

| Cloud Provider                  | Reference Link                                                                                  |
|-------------|-----------------------------------------------------------------------------------------------|
| AKS Cluster | https://docs.microsoft.com/en-us/azure/aks/servicemesh-istio-install  |
| GKE Cluster | https://cloud.google.com/istio/docs/istio-on-gke/installing            |
| EKS Cluster | https://aws.amazon.com/blogs/opensource/getting-started-istio-eks/     |
| OnPremise   | https://istio.io/latest/docs/setup/platform-setup/docker/              |

Run the following command to get the ingress IP address for Istio.

    kubectl get service/istio-ingressgateway -n istio-system


Note the ingress EXTERNAL-IP address and map it with your DNS. If you do not have the DNS and want to use the application, then you can use the ingress IP address.
