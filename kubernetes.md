# Deploy Bold BI on Kubernetes

This section provides instructions on how to deploy Bold BI on different cloud clusters. Follow the methods below to successfully deploy the application.

## Installation Methods

You can deploy Bold BI on your Kubernetes cluster using either Kustomize or Helm. Follow the steps based on your preference:

* [Installation using Kustomize with kubectl](#installation-using-kustomize-with-kubectl)
* [Installation using Helm](#installation-using-helm)

## Prerequisites

- **[Install Helm](https://helm.sh/docs/intro/install/)**: If you prefer to use Helm to install Bold BI, make sure to install Helm.
- **[Install kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)**: Ensure that you install kubectl on your local machine to facilitate the deployment process.
- **Kubernetes Cluster**: Create a cluster in which you want to deploy the Bold BI Application. You can use services like [AKS](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), [AWS EKS](https://docs.aws.amazon.com/eks/latest/userguide/getting-started.html), or [GKE](https://cloud.google.com/kubernetes-engine/docs/quickstart).
- **[Persistent Volume](PersistentVolumeCreation.md)**: Set up the necessary file storage for your Kubernetes clusters.
- **Load Balancer**: To configure Bold BI with Ingress, you need to install the [Nginx Ingress controller](https://kubernetes.github.io/ingress-nginx/deploy/) in your cluster.
- **Database**: Create a database to store metadata and intermediate data details of the Bold BI site.
- **Web Browsers**: The supported web browsers include Microsoft Edge, Mozilla Firefox, and Google Chrome.

Please ensure that you have fulfilled these prerequisites before proceeding with the deployment.

## Installation using Kustomize with kubectl

### 1. Create and Connect to a Kubernetes Cluster

Before deploying Bold BI, you need to create and connect to a Kubernetes cluster. Refer to the table below for instructions based on your chosen cloud provider or on-premise setup:

| Cloud Providers            | Cluster Creation                                                                                      | Cluster Connection                                                                                           |
|----------------------------|-------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| Azure Kubernetes Service   | [Azure AKS Walkthrough](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal)      | [AKS Cluster Connection](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal#connect-to-the-cluster) |
| Google Kubernetes Engine   | [Google GKE Console](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview) | [GKE Cluster Connection](https://cloud.google.com/kubernetes-engine/docs/quickstart)                         |
| Elastic Kubernetes Service | [AWS EKS Guide](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)                  | [EKS Cluster Connection](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/)     |

Ensure you have a Kubernetes cluster ready and connected before proceeding with the installation steps.


### 2. Setup Persistent Volume

Since Bold BI is a stateful application, a persistent volume is required to store the application data. We recommend using external file systems such as Azure File Share, Amazon EFS, or Google Filestore to ensure that the application data is not tied to the cluster. This setup also ensures seamless backup of application data and supports the ReadWriteMany access mode, which is essential for Bold BI's multi-node scaling capabilities. Please follow the documentation links below to set up persistent volume storage:

| **Cloud Provider** | **Link** |
|--------------------|----------|
| AKS File Storage   | [Create an NFS file share instance](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-portal?tabs=azure-portal) |
| EKS File System    | [Create an Amazon Elastic File System](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) |
| GKE File Store     | [Create a Google Filestore instance](https://cloud.google.com/filestore/docs/creating-instances) |

Please ensure that you follow the provided links to set up the necessary file storage for your Kubernetes clusters.

After creating the respective cloud provider storage, note the following details according to the provider:

| **Cloud Provider** | **Required Details** |
|--------------------|----------------------|
| AKS File Storage   | Storage account name <br> File share name |
| EKS File System    | EFS filesystem ID    |
| GKE File Store     | Filestore name <br> Filestore IP address |

### 3. Setup Load Balancer

* Bold BI supports Nginx and Istio as load balancers. Please install the Nginx Ingress Controller or Istio Gateway in your cluster by following the links below. If you already have either of them installed, you can skip this step.

    | **Cloud Provider**                | **Nginx Ingress Controller**                                                               | **Istio Gateway**                                                                                          |
    |-----------------------------------|-------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
    | Azure Kubernetes Service (AKS)    | [Nginx for AKS](https://kubernetes.github.io/ingress-nginx/deploy/#azure)                  | [Istio for AKS](https://docs.microsoft.com/en-us/azure/aks/servicemesh-istio-install)                      |
    | Google Kubernetes Engine (GKE)    | [Nginx for GKE](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke)                | [Istio for GKE](https://cloud.google.com/istio/docs/istio-on-gke/installing)                               |
    | Elastic Kubernetes Service (EKS)  | [Nginx for EKS](https://kubernetes.github.io/ingress-nginx/deploy/#aws)                    | [Istio for EKS](https://aws.amazon.com/blogs/opensource/getting-started-istio-eks/)                        |

* After installing the load balancer, note the external IP address. If you need to host the application using a domain, map the IP address with the domain. Alternatively, you can use the IP address directly for hosting the application. Use the commands below to obtain the external IP of the Nginx Load Balancer and Istio Gateway.

    | **Load Balancer** | **Command to Obtain External IP** |
    |-------------------|-----------------------------------|
    | Nginx             | `kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` |
    | Istio             | `kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'`     |

* Apply the Nginx Ingress or Istio Gateway configuration in your cluster.

    | **Load Balancer** | **Command to Apply Configuration** |
    |-------------------|------------------------------------|
    | Nginx             | `kubectl apply -f ingress.yaml`    |
    | Istio             | `kubectl apply -f gateway.yaml`    |

### 4. Download Kustomization and Configure Customization for Bold BI Installation

* To deploy Bold BI in a Kubernetes cluster, use the appropriate `kustomization.yaml` file based on your specific requirement:

    - **Azure Kubernetes Service (AKS):**
        - [Download AKS Kustomization.yaml](aks/boldbi/kustomization.yaml)

    - **Google Kubernetes Engine (GKE):**
        - [Download GKE Kustomization.yaml](gke/boldbi/kustomization.yaml)

    - **Amazon Elastic Kubernetes Service (EKS):**
        - [Download EKS Kustomization.yaml](eks/boldbi/kustomization.yaml)

    Make sure to choose the correct `kustomization.yaml` file based on your deployment target.

* Update the external IP address or domain name obtained in step 3 in the `kustomization.yaml` file.

    ![App-Base-URL](images/app-base-url.png) 

* Update the persistent volume details you obtained from step 2 based on the cloud provider.

    | File Storage                  | Action                                                                                                      |
    |-------------------------------|-------------------------------------------------------------------------------------------------------------|
    | Azure File Share              | Replace the `storage account name` and `file share name` with `<storage_account_name>` and `<file_share_name>`, respectively, in the file. ![After Replacing File Storage name](images/After-replace-fileshare.png) |
    | GKE File Store                | Replace the `file share name` and `IP address` with `<file_share_name>` and `<file_share_ip_address>` in the file. ![Replace file store name](images/replace-filestore.png)                |
    | Elastic File Storage for EKS  | Replace the `file system ID` with `<efs_file_system_id>` in the file. ![replace-fs-id](images/replace-fs-id.png) |

Ensure that all the required details are accurately updated in the `kustomization.yaml` file before proceeding with the deployment.

    
### 5. Bold BI Installation and Application Startup

* Run the following command to deploy the Bold BI application on your cluster from the `kustomization.yaml` file location:

    ```bash
    kubectl apply -k .
    ```

* Use the following command to get the status of the pods and wait until the pods are in the running state:

    ```bash 
    kubectl get pods -n bold-services
    ```

* After the pods successfully transition to the running state, you can access the application using the IP address. If you configured the application using a DNS, you can access it using the DNS.

* Configure the Bold BI On-Premise application startup to use the application. Please refer to the following link for more details on configuring the application startup:

    [Bold BI Application Startup](https://help.boldbi.com/embedded-bi/application-startup)


## Deployment using Helm

### 1. Create and Connect to a Kubernetes Cluster

Before deploying Bold BI, you need to create and connect to a Kubernetes cluster. Refer to the table below for instructions based on your chosen cloud provider or on-premise setup:

| Cloud Providers            | Cluster Creation                                                                                      | Cluster Connection                                                                                           |
|----------------------------|-------------------------------------------------------------------------------------------------------|-------------------------------------------------------------------------------------------------------------|
| Azure Kubernetes Service   | [Azure AKS Walkthrough](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal)      | [AKS Cluster Connection](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal#connect-to-the-cluster) |
| Google Kubernetes Engine   | [Google GKE Console](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview) | [GKE Cluster Connection](https://cloud.google.com/kubernetes-engine/docs/quickstart)                         |
| Elastic Kubernetes Service | [AWS EKS Guide](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)                  | [EKS Cluster Connection](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/)     |

Ensure you have a Kubernetes cluster ready and connected before proceeding with the installation steps.


### 2. Setup Persistent Volume

Since Bold BI is a stateful application, a persistent volume is required to store the application data. We recommend using external file systems such as Azure File Share, Amazon EFS, or Google Filestore to ensure that the application data is not tied to the cluster. This setup also ensures seamless backup of application data and supports the ReadWriteMany access mode, which is essential for Bold BI's multi-node scaling capabilities. Please follow the documentation links below to set up persistent volume storage:

| **Cloud Provider** | **Link** |
|--------------------|----------|
| AKS File Storage   | [Create an NFS file share instance](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-portal?tabs=azure-portal) |
| EKS File System    | [Create an Amazon Elastic File System](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) |
| GKE File Store     | [Create a Google Filestore instance](https://cloud.google.com/filestore/docs/creating-instances) |

Please ensure that you follow the provided links to set up the necessary file storage for your Kubernetes clusters.

After creating the respective cloud provider storage, note the following details according to the provider:

| **Cloud Provider** | **Required Details** |
|--------------------|----------------------|
| AKS File Storage   | Storage account name <br> File share name |
| EKS File System    | EFS filesystem ID    |
| GKE File Store     | Filestore name <br> Filestore IP address |

### 3. Setup Load Balancer

* Bold BI supports Nginx and Istio as load balancers. Please install the Nginx Ingress Controller or Istio Gateway in your cluster by following the links below. If you already have either of them installed, you can skip this step.

    | **Cloud Provider**                | **Nginx Ingress Controller**                                                               | **Istio Gateway**                                                                                          |
    |-----------------------------------|-------------------------------------------------------------------------------------------|------------------------------------------------------------------------------------------------------------|
    | Azure Kubernetes Service (AKS)    | [Nginx for AKS](https://kubernetes.github.io/ingress-nginx/deploy/#azure)                  | [Istio for AKS](https://docs.microsoft.com/en-us/azure/aks/servicemesh-istio-install)                      |
    | Google Kubernetes Engine (GKE)    | [Nginx for GKE](https://kubernetes.github.io/ingress-nginx/deploy/#gce-gke)                | [Istio for GKE](https://cloud.google.com/istio/docs/istio-on-gke/installing)                               |
    | Elastic Kubernetes Service (EKS)  | [Nginx for EKS](https://kubernetes.github.io/ingress-nginx/deploy/#aws)                    | [Istio for EKS](https://aws.amazon.com/blogs/opensource/getting-started-istio-eks/)                        |

* After installing the load balancer, note the external IP address and map the IP address with the domain. Use the commands below to obtain the external IP of the Nginx Load Balancer and Istio Gateway.

    | **Load Balancer** | **Command to Obtain External IP** |
    |-------------------|-----------------------------------|
    | Nginx             | `kubectl get svc -n ingress-nginx ingress-nginx-controller -o jsonpath='{.status.loadBalancer.ingress[0].ip}'` |
    | Istio             | `kubectl get svc -n istio-system istio-ingressgateway -o jsonpath='{.status.loadBalancer.ingress[0].ip}'`     |

### 4. Adding the Bold BI Helm Repository

To integrate the Bold BI Helm repository into your Helm setup, follow these steps:

1. Run the command below to add the Bold BI Helm repository:

    ```shell
    helm repo add boldbi https://boldbi.github.io/boldbi-kubernetes
    ```

    This command adds `boldbi` as the name for the repository with the URL `https://boldbi.github.io/boldbi-kubernetes`.

2. Once added, you can proceed to update your Helm repositories:

    ```shell
    helm repo update
    ```

    Updating ensures you have the latest information about available charts from the newly added repository.

### 4. Install Bold BI using helm

* To deploy Bold BI on Kubernetes, execute the appropriate command for your cloud provider. Ensure to replace placeholders such as `appBaseUrl` and `persistentVolume` details specific to your cloud provider resources.

 **AKS** : 
 
 ```shell
  helm upgrade --install boldbi boldbi/boldbi \
    -f https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/aks-values.yaml \
    --set appBaseUrl=https://your-app-base-url \
    --set persistentVolume.aks.nfs.fileShareName=your-fileshare-name/nfs \
    --set persistentVolume.aks.nfs.hostName=your-fileshare-hostname
  ``` 

Replace the placeholders:
- `https://your-app-base-url` with your actual application base URL.
- `your-fileshare-name` with the name of your file share.
- `your-fileshare-hostname` with the hostname of your file share.
- `your-efs-file-system-id` with your EFS file system ID (for EKS).

**For example:**
```shell
helm upgrade --install boldbi boldbi/boldbi \
  -f https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/aks-values.yaml \
  --set appBaseUrl=https://optionallib-check.boldbi.demo.com \
  --set persistentVolume.aks.nfs.fileShareName=aksstorage1026/nfs \
  --set persistentVolume.aks.nfs.hostName=aksstorage1026.file.core.windows.net
```
* If you are configuring the site with HTTPS protocol, please create a secret with SSL cert and keys using the below command.

    ```sh
    kubectl create secret tls bold-tls -n bold-services --key <key-path> --cert <certificate-path>
    ```

### 5. Access the application and Application Startup

* Use the following command to get the status of the pods and wait until the pods are in the running state:

    ```bash 
    kubectl get pods -n bold-services
    ```

* After the pods successfully transition to the running state, you can access using a DNS, you can access it using the DNS.

* Configure the Bold BI On-Premise application startup to use the application. Please refer to the following link for more details on configuring the application startup:

    [Bold BI Application Startup](https://help.boldbi.com/embedded-bi/application-startup)
