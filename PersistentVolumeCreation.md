# Persistent Volumes
1. AKS File Storage
2. Google Kubernetes Engine (GKE)
3. Amazon Elastic Kubernetes Service (EKS)

## AKS File Storage

### NFS

- Create an [NFS file share instance](https://learn.microsoft.com/en-us/azure/storage/files/storage-how-to-use-files-portal?tabs=azure-portal) within your premium storage account. 
- Take note of the storage account and file share name, as these will be used to store shared folders for application usage.

    ![NFS Host Name](images/nfs-hostname.png)

    **NOTE:** The `premium storage account` of the NFS fileshare must be within the same subscription as the AKS cluster.

### SMB

1. Create a File share instance in your storage account and note the File share name to store the shared folders for application usage.

2. Encode the storage account name and storage key in `base64` format.

    For encoding the values to base64 please run the following command in powershell

    ```console
    [System.Convert]::ToBase64String([System.Text.Encoding]::UTF8.GetBytes("<plain-text>"))
    ```

    ![File Share details](images/aks-file-storage.png)

## GKE File Store

- Create a [Google filestore instance](https://console.cloud.google.com/filestore)to store the shared folders for application usage.
- Note the File share name and IP address after creating filestore instance.

    ![File Store Details](images/gke_file_share_details.png)

## EKS File System
- create an [Amazon Elastic File System](https://docs.aws.amazon.com/efs/latest/ug/gs-step-two-create-efs-resources.html) (EFS) volume to store the shared folders for application usage.
- Note the File system ID after creating EFS file system.

    ![File-System-ID](images/file-system-id.png)



