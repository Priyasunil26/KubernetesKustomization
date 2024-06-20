# Deploy Bold BI on Kubernetes
This section provides instructions on how to deploy Bold BI in different cloud cluster. Please follow the documentation below to successfully deploy the application.

* [Installation using kustomization with kubectl](#deployment-using-kustomization-with-kubectl)
    
* [Installation using helm](#deployment-using-helm)

## Prerequisites

- **[Install helm](https://helm.sh/docs/intro/install/)**: If you prefer to use Helm to install Bold BI, make sure to install Helm.
- **[Install kubectl](https://kubernetes.io/docs/tasks/tools/#kubectl)**: Ensure that you install kubectl on your local machine to facilitate the deployment process.
- **Kubernetes Cluster** ([AKS](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal), [AWS](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal#connect-to-the-cluster), [GKE](https://console.cloud.google.com/kubernetes)): Create a cluster in which you want to deploy the Bold BI Application.
- **[Persistent Volume](PersistentVolumeCreation.md)**: Set up the necessary file storage for your Kubernetes clusters.
- **Load Balancer**: To configure Bold BI with Ingress, you need to install the [Nginx Ingress controller](https://kubernetes.github.io/ingress-nginx/deploy/) in your cluster.
- **Database**: Create a database to store metadata and intermediate data details of the Bold BI site.
- **Web Browsers**: The supported web browsers include Microsoft Edge, Mozilla Firefox, and Google Chrome.

Please ensure that you have fulfilled these prerequisites before proceeding with the deployment.

## Deployment using kustomization with kubectl

1. Create and connect to a Kubernetes cluster to deploy Bold BI. Please refer to the table below for creating and connecting to Kubernetes clusters on different cloud providers and on-premise.

    | Cloud Providers            | Cluster Creation                                                                                    | Cluster Connection                                                                                      |
    |----------------------------|----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
    | Azure Kubernetes Service   | [Azure AKS Walkthrough](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal) | [AKS Cluster Connection](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal#connect-to-the-cluster) |
    | Google Kubernetes Engine   | [Google GKE Console](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)                                | [GKE Cluster Connection](https://cloud.google.com/kubernetes-engine/docs/quickstart)                     |
    | Elastic Kubernetes Service | [AWS EKS Guide](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)             | [EKS Cluster Connection](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/) |

2. Create a File share instance in your storage account and note the File share name to store the shared folders for application usage.

    | **Cloud Provider** | **Link** |
    |--------------------|----------|
    |  AKS File Storage | [Create an NFS file share instance](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-portal?tabs=azure-portal) |
    |  EKS File System   | [Create an Amazon Elastic File System](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) |
    |  GKE File Store   | [Create a Google filestore instance](https://cloud.google.com/filestore/docs/creating-instances) |

    Please ensure that you follow the provided links to set up the necessary file storage for your Kubernetes clusters.

3. To deploy Bold BI in a Kubernetes cluster, you can use the following Kustomization.yaml files based on your specific requirement:

    - **Azure Kubernetes Service (AKS):**
        - [Download AKS Kustomization.yaml](aks/boldbi/kustomization.yaml)

    - **Google Kubernetes Engine (GKE):**
        - [Download GKE Kustomization.yaml](gke/boldbi/kustomization.yaml)

    - **Amazon Elastic Kubernetes Service (EKS):**
        - [Download EKS Kustomization.yaml](eks/boldbi/kustomization.yaml)

    Make sure to choose the correct Kustomization.yaml file based on your deployment target.

5. Open the Kustomization.yaml file. Edit the File store Path value.

    |File Storage                  | Action |
    -------------------------------|-------------------------------|
    | Azure File Share             | Replace the `storage account name and file share name` with `<storage_account_name>` and `<file_share_name>`, respectively, in the file.                                ![After Replacing File Storage name](images/After-replace-fileshare.png)                               |
    | GKE File Store               | Replace the `File share name and IP address` with `<file_share_name>` and `<file_share_ip_address>`, in the file.                                                 ![Replace file store name](images/replace-filestore.png)                                              |
    | Elastic File Storage for EKS | Replace the `File system ID` with `<efs_file_system_id>` in the file.                                                                                                          ![replace-fs-id](images/replace-fs-id.png)                                                                |
    
   
6. After connecting with your cluster, deploy the `latest Nginx ingress controller` to your cluster using the following command.

    | Cloud Provider                  | Installation Command                                                                                       |
    |---------------------------------|--------------------------------------------------------------------------------------------------------|
    | Azure Kubernetes Service (AKS)  | kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml |
    | Google Kubernetes Engine (GKE)  | kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml                                         |
    | Elastic Kubernetes Service (EKS)| kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/aws/deploy.yaml  |
   
 
    For more information about the load balancer installation, please refer to this [link](https://kubernetes.github.io/ingress-nginx/deploy/). Deploy Nginx according to your target cloud provider (AKS, GKE, EKS) by choosing the appropriate configuration.

7. The latest Nginx Ingress controller has a configuration change where the default value for `allow-snippet-annotations` is set to `false`. To fix this, you need to edit the Nginx Ingress ConfigMap file and set the value to `true`.

    ![Set Snippet value true](images/snippet-true.png)

    Use the following command to edit the ConfigMap:
        
        kubectl edit cm ingress-nginx-controller -n ingress-nginx

8. Run the following command to obtain the ingress IP address. Note the ingress EXTERNAL-IP address and map it with your DNS. If you do not have the DNS and want to use the application, then you can use the ingress IP address.

    ```bash 
    kubectl get service/ingress-nginx-controller -n ingress-nginx

9. After obtaining the External IP address, replace the `app-base URL` with your External IP address or Domain name.

    ![App-Base-URL](images/app-base-url.png)

10. Navigate to the folder where the deployment file were downloaded from Step 4.

11. Run the following command to deploy Bold BI application on cluster
    ```bash
    kubectl apply -k .

12. Please wait for some time until the Bold BI  application is deployed to your cluster.

13. Use the following command to get the pods status.
    ```bash 
    kubectl get pods -n bold-services

14. Wait until you see the applications running. Then, use the DNS or ingress IP address you obtained from Step 8 to access the application in the browser.

15. Configure the Bold BI On-Premise application startup to use the application. Please refer the following link for more details on configuring the application startup.

    https://help.boldbi.com/embedded-bi/application-startup


## Deployment using Helm

1. Create and connect to a Kubernetes cluster to deploy Bold BI. Please refer to the table below for creating and connecting to Kubernetes clusters on different cloud providers and on-premise.

    | Cloud Providers            | Cluster Creation                                                                                    | Cluster Connection                                                                                      |
    |----------------------------|----------------------------------------------------------------------------------------------------|--------------------------------------------------------------------------------------------------------|
    | Azure Kubernetes Service   | [Azure AKS Walkthrough](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal) | [AKS Cluster Connection](https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough-portal#connect-to-the-cluster) |
    | Google Kubernetes Engine   | [Google GKE Console](https://cloud.google.com/kubernetes-engine/docs/concepts/kubernetes-engine-overview)                                | [GKE Cluster Connection](https://cloud.google.com/kubernetes-engine/docs/quickstart)                     |
    | Elastic Kubernetes Service | [AWS EKS Guide](https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html)             | [EKS Cluster Connection](https://aws.amazon.com/premiumsupport/knowledge-center/eks-cluster-connection/) |

2. Create a File share instance in your storage account and note the File share name to store the shared folders for application usage.

    | **Cloud Provider** | **Link** |
    |--------------------|----------|
    |  AKS File Storage | [Create an NFS file share instance](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-portal?tabs=azure-portal) |
    |  EKS File System   | [Create an Amazon Elastic File System](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) |
    |  GKE File Store   | [Create a Google filestore instance](https://cloud.google.com/filestore/docs/creating-instances) |

    Please ensure that you follow the provided links to set up the necessary file storage for your Kubernetes clusters.
    
3. Adding the Bold BI Helm Repository

    To add the Bold BI Helm repository to your Helm setup, you need to run the following command:

```shell
helm repo add boldbi https://boldbi.github.io/boldbi-kubernetes
```

```shell
helm repo update
```

4. Viewing Charts in the Bold BI Repository

    To view the available charts in the Bold BI repository, you can use the `helm search repo` command:

```shell
helm search repo boldbi
```

```
NAME            CHART VERSION   APP VERSION     DESCRIPTION
boldbi/boldbi   7.9.50          7.9.50          Embed powerful analytics inside your apps and t...
```

   
5. After connecting with your cluster, deploy the `latest Nginx ingress controller` to your cluster using the following command.

    | Cloud Provider                  | Installation Command                                                                                       |
    |---------------------------------|--------------------------------------------------------------------------------------------------------|
    | Azure Kubernetes Service (AKS)  | kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml |
    | Google Kubernetes Engine (GKE)  | kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/cloud/deploy.yaml                                         |
    | Elastic Kubernetes Service (EKS)| kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.0/deploy/static/provider/aws/deploy.yaml  |
   
 
    For more information about the load balancer installation, please refer to this [link](https://kubernetes.github.io/ingress-nginx/deploy/). Deploy Nginx according to your target cloud provider (AKS, GKE, EKS) by choosing the appropriate configuration.

6. The latest Nginx Ingress controller has a configuration change where the default value for `allow-snippet-annotations` is set to `false`. To fix this, you need to edit the Nginx Ingress ConfigMap file and set the value to `true`.

    ![Set Snippet value true](images/snippet-true.png)

    Use the following command to edit the ConfigMap:
        
        kubectl edit cm ingress-nginx-controller -n ingress-nginx

7. Run the following command to obtain the ingress IP address.

    ```bash 
    kubectl get service/ingress-nginx-controller -n ingress-nginx

8. After obtaining the external IP address of the Nginx Ingress controller, map the IP address to a domain name for accessing the application.

9. To deploy Bold BI on Kubernetes, run the appropriate command for your cloud provider. Ensure to update the `fileshare` name and `appBaseUrl` in the configuration to match your specific resources.

    | Cloud Provider | Command |
    |----------------|---------|
    | **AKS** | `helm upgrade --install boldbi boldbi/boldbi -f https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/aks-values.yaml --set appBaseUrl=https://your-app-base-url --set persistentVolume.aks.nfs.fileShareName=your-fileshare-name/nfs --set persistentVolume.aks.nfs.hostName=your-fileshare-hostname` |
    | **GKE** | `helm upgrade --install boldbi boldbi/boldbi -f https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/gke-values.yaml --set appBaseUrl=https://your-app-base-url --set persistentVolume.gke.nfs.fileShareName=your-fileshare-name --set persistentVolume.gke.nfs.hostName=your-fileshare-hostname` |
    | **EKS** | `helm upgrade --install boldbi boldbi/boldbi -f https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/eks-values.yaml --set appBaseUrl=https://your-app-base-url --set persistentVolume.eks.efs.fileSystemId=your-efs-file-system-id` |

    Replace the placeholders:
    - `https://your-app-base-url` with your actual application base URL.
    - `your-fileshare-name` with the name of your file share.
    - `your-fileshare-hostname` with the hostname of your file share.
    - `your-efs-file-system-id` with your EFS file system ID (for EKS).

    **For example:**
```sh
helm upgrade --install boldbi boldbi/boldbi -f https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/aks-values.yaml --set appBaseUrl=https://optionallib-check.boldbi.demo.com --set persistentVolume.aks.nfs.fileShareName=aksstorage1026/nfs --set persistentVolume.aks.nfs.hostName=aksstorage1026.file.core.windows.net
```

10. If you want to download the YAML files and edit them, please download the YAML from the links below and update the YAML file according to your needs.

    | Cloud Provider | Link |
    |----------------|------|
    | AKS | [click here to download](https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/aks-values.yaml) |
    | GKE | [click here to download](https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/gke-values.yaml) |
    | EKS | [click here to download](https://raw.githubusercontent.com/boldbi/boldbi-kubernetes/main/helm/custom-values/eks-values.yaml) |

    Make the necessary changes and save the updated YAML files for deployment. Run the following command to deploy Bold BI in your cluster:
```bash
helm upgrade --install [RELEASE_NAME] boldbi/boldbi -f [Crafted values.yaml file]
```
**For example:**

```sh 
helm upgrade boldbi boldbi/boldbi -f my-values.yaml
```
11. Please wait for some time until the Bold BI  application is deployed to your cluster.

12. Use the following command to get the pods status.
    ```bash 
    kubectl get pods -n bold-services

13. Wait until you see the applications running. Then, use the `appBaseUrl` to access the application in the browser.

14. Configure the Bold BI On-Premise application startup to use the application. Please refer the following link for more details on configuring the application startup.

    [Bold BI Application Startup Configuration](https://help.boldbi.com/embedded-bi/application-startup)