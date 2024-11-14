### 1. **Deploying using Kubernetes CLI (`kubectl`)**

#### Example: Deploying a simple Nginx pod
```bash
# Create a simple Nginx Pod using kubectl
kubectl run nginx --image=nginx --port=80
```

#### Example: Exposing the Nginx pod as a service
```bash
# Expose the Pod as a Service
kubectl expose pod nginx --port=80 --target-port=80 --type=LoadBalancer
```

#### Example: Scale the deployment
```bash
# Scale the deployment to 3 replicas
kubectl scale deployment nginx --replicas=3
```

### 2. **Deploying using YAML Manifest Files**

#### Example: Create a deployment using a YAML file

1. Create a file named `nginx-deployment.yaml`:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: nginx-deployment
   spec:
     replicas: 3
     selector:
       matchLabels:
         app: nginx
     template:
       metadata:
         labels:
           app: nginx
       spec:
         containers:
         - name: nginx
           image: nginx:1.21
           ports:
           - containerPort: 80
   ```
2. Deploy using `kubectl`:
   ```bash
   # Deploy the YAML file
   kubectl apply -f nginx-deployment.yaml
   ```

#### Example: Create a service using a YAML file

1. Create a file named `nginx-service.yaml`:
   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: nginx-service
   spec:
     selector:
       app: nginx
     ports:
       - protocol: TCP
         port: 80
         targetPort: 80
     type: LoadBalancer
   ```
2. Deploy using `kubectl`:
   ```bash
   # Deploy the service
   kubectl apply -f nginx-service.yaml
   ```

### 3. **Deploying using Helm**

#### Example: Deploying an application using a Helm chart

1. **Add a Helm Repository**:
   ```bash
   # Add the official Helm stable repository
   helm repo add bitnami https://charts.bitnami.com/bitnami
   helm repo update
   ```

2. **Install an Nginx Helm Chart**:
   ```bash
   # Install Nginx using Helm
   helm install my-nginx bitnami/nginx
   ```

3. **Uninstall the Helm Release**:
   ```bash
   # Uninstall the deployed release
   helm uninstall my-nginx
   ```

### 4. **Deploying using Kustomize**

Kustomize is a tool that allows you to customize Kubernetes YAML configurations without modifying the original files.

#### Example: Deploying with Kustomize

1. Create the following folder structure:

   ```
   ./kustomize-example/
   ├── deployment.yaml
   └── kustomization.yaml
   ```

2. Create `deployment.yaml` file:
   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: my-app
   spec:
     replicas: 2
     selector:
       matchLabels:
         app: my-app
     template:
       metadata:
         labels:
           app: my-app
       spec:
         containers:
         - name: my-app
           image: nginx
           ports:
           - containerPort: 80
   ```

3. Create `kustomization.yaml` file:
   ```yaml
   apiVersion: kustomize.config.k8s.io/v1beta1
   kind: Kustomization
   resources:
     - deployment.yaml
   ```

4. Deploy using Kustomize:
   ```bash
   # Deploy using kubectl with Kustomize
   kubectl apply -k ./kustomize-example/
   ```

### 5. **Deploying using `kubectl` with Kustomize directly**

If your manifest files include a `kustomization.yaml`, you can deploy using `kubectl` directly:
```bash
# Deploy with kubectl using Kustomize support
kubectl apply -k ./kustomize-example/
```

### 6. **Deploying using `kubectl` with Remote YAML**

You can directly deploy YAML from a remote URL using `kubectl`.
```bash
# Deploy using a remote YAML file
kubectl apply -f https://raw.githubusercontent.com/kubernetes/website/main/content/en/examples/application/nginx-app.yaml
```

### 7. **Deploying using `kubeadm` (Cluster Bootstrap)**

`kubeadm` is a tool to create Kubernetes clusters. Once you have a running cluster:

```bash
# Use kubeadm to deploy a simple Kubernetes cluster
kubeadm init
```

After setting up the cluster, use any deployment method (YAML, Helm, etc.) to deploy applications.

### 8. **Deploying using `skaffold`**

Skaffold is a command-line tool that facilitates continuous development for Kubernetes applications.

1. **Initialize a Skaffold Project**:
   ```bash
   skaffold init
   ```

2. **Deploy an Application with Skaffold**:
   ```bash
   skaffold dev
   ```

### 9. **Deploying using `kubectl apply -f` with Multiple Files**

You can apply multiple YAML files at once:
```bash
# Apply multiple files
kubectl apply -f ./deployment.yaml -f ./service.yaml
```

### 10. **Deploying with `kubectl` using JSON Format**

You can create resources using JSON files:
1. Create `nginx-deployment.json`:
   ```json
   {
     "apiVersion": "apps/v1",
     "kind": "Deployment",
     "metadata": {
       "name": "nginx-deployment"
     },
     "spec": {
       "replicas": 2,
       "selector": {
         "matchLabels": {
           "app": "nginx"
         }
       },
       "template": {
         "metadata": {
           "labels": {
             "app": "nginx"
           }
         },
         "spec": {
           "containers": [
             {
               "name": "nginx",
               "image": "nginx:latest",
               "ports": [
                 {
                   "containerPort": 80
                 }
               ]
             }
           ]
         }
       }
     }
   }
   ```
2. Deploy with `kubectl`:
   ```bash
   kubectl apply -f nginx-deployment.json
   ```

### Summary

| Deployment Method         | Command or Tool                 | Use Case                                     |
|---------------------------|--------------------------------|----------------------------------------------|
| **kubectl CLI**            | `kubectl run`, `kubectl expose` | Simple or quick deployments                  |
| **YAML Files**             | `kubectl apply -f`              | Configuring deployments in detail            |
| **Helm**                   | `helm install`, `helm uninstall`| Complex applications and package management  |
| **Kustomize**              | `kubectl apply -k`              | Customizing existing configurations          |
| **kubectl with Remote URL**| `kubectl apply -f <URL>`        | Deploying pre-defined resources from URL     |
| **kubeadm**                | `kubeadm init`                  | Bootstrapping a Kubernetes cluster           |
| **Skaffold**               | `skaffold dev`                  | Continuous development and deployment        |

These methods cover a wide range of deployment scenarios in Kubernetes.