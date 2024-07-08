# Configuring SSL in Kubernetes Deployment Using `kubectl`

To configure SSL in your Kubernetes deployment using `kubectl`, follow the steps below. This guide will help you set up SSL by modifying your Ingress YAML file.

## Steps to Configure SSL

1. **Locate Your Ingress YAML File**: Find the Ingress YAML file that you use for your Kubernetes deployment. This file typically contains the rules and configurations for routing traffic to your services.

2. **Uncomment the Host Line**: In the Ingress YAML file, locate the section under `rules` where the host is specified. Uncomment the line and add your host.

    ```yaml
    spec:
      #tls:
      #- hosts:
        #- example.com
        #secretName: bold-tls
      rules:
      - #host: example.com
    ```

3. **Add Your Host and Secret Name**: Replace `example.com` with your actual host and specify the `secretName` that contains your SSL certificate.

    ```yaml
    spec:
      tls:
      - hosts:
        - yourdomain.com
        secretName: your-tls-secret
      rules:
      - host: yourdomain.com
    ```

4. **Save the Ingress YAML File**: After making the necessary changes, save the Ingress YAML file.

5. **Apply the Configuration**: Use `kubectl` to apply the updated Ingress configuration.

    ```sh
    kubectl apply -f path/to/your-ingress.yaml
    ```

6. **Verify the Configuration**: Ensure that the Ingress resource is correctly configured and that the SSL certificate is properly applied.

    ```sh
    kubectl describe ingress your-ingress-name
    ```
![image](https://github.com/Priyasunil26/KubernetesKustomization/assets/158257824/727bc8e7-19ba-4fc5-9887-dc0f361907c8)

