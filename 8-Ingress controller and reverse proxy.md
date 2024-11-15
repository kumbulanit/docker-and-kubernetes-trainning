### Practical Exercise: Setting Up and Testing Ingress Controller with a Reverse Proxy in Kubernetes 

In this exercise, we will set up and test an **Ingress Controller** with **NGINX** and use it as a reverse proxy to route traffic to a sample application deployed in Kubernetes.

---

### **Step 1: Install NGINX Ingress Controller Without Helm**

We will manually deploy the NGINX Ingress Controller using Kubernetes manifests.

#### **1.1 Create a Namespace for the Ingress Controller**

First, create a namespace for the Ingress Controller:

```bash
kubectl create namespace ingress-nginx
```

#### **1.2 Deploy NGINX Ingress Controller**

We will deploy the NGINX Ingress Controller using the official NGINX Ingress Controller manifests. Run the following command to apply the necessary resources.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

This will create all the necessary resources in the `ingress-nginx` namespace, including the deployment, services, and configuration for the NGINX Ingress Controller.

#### **1.3 Verify the Installation**

After the NGINX Ingress Controller is deployed, verify that the components are running by checking the pods in the `ingress-nginx` namespace:

```bash
kubectl get pods -n ingress-nginx
```

You should see a pod named `ingress-nginx-controller-xxxx` running.

---

### **Step 2: Deploy a Sample Application (NGINX)**

Now, let's deploy a simple NGINX application that we will use as a backend service.

#### **2.1 Create the NGINX Deployment**

Create a file named `nginx-deployment.yaml` with the following content:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2  # Deploying 2 replicas of NGINX
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
          image: nginx:latest
          ports:
            - containerPort: 80
```

Deploy the NGINX application:

```bash
kubectl apply -f nginx-deployment.yaml
```

#### **2.2 Expose the NGINX Deployment**

Now, expose the NGINX deployment via a Kubernetes Service. Create a file named `nginx-service.yaml`:

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
  type: ClusterIP
```

Apply the service:

```bash
kubectl apply -f nginx-service.yaml
```

---

### **Step 3: Create the Ingress Resource**

Next, create an Ingress resource that will define how traffic should be routed to the NGINX service.

#### **3.1 Create an Ingress Resource**

Create a file named `nginx-ingress.yaml` with the following content:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: nginx.example.com
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

In this configuration:
- We define the domain `nginx.example.com` (you can replace it with your domain).
- The path `/` routes traffic to the NGINX service on port 80.

Apply the Ingress resource:

```bash
kubectl apply -f nginx-ingress.yaml
```

#### **3.2 Verify the Ingress Resource**

Check if the Ingress resource has been created successfully:

```bash
kubectl get ingress
```

This will show the external address (IP or domain) where the service is accessible.

---

### **Step 4: Expose the Ingress Controller’s IP Address**

If you are using a cloud provider, the external IP should be automatically assigned to the Ingress Controller’s service. To get the external IP address, run the following:

```bash
kubectl get svc -n ingress-nginx
```

Look for the `EXTERNAL-IP` under the `ingress-nginx-controller` service. If it says `<pending>`, wait a minute for the external IP to be assigned.

if it does not show the EXTERNAL IP please do the below 

```bash
minikube tunnel
```
and then open a new tab and rerun 

```bash
kubectl get svc -n ingress-nginx
```


---

### **Step 5: Update Your Local DNS or `/etc/hosts`**

For testing purposes, modify your local machine’s `/etc/hosts` file (Linux/macOS) or `C:\Windows\System32\drivers\etc\hosts` file (Windows) to map the domain `nginx.example.com` to the Ingress controller's external IP address.

Add the following line to the `/etc/hosts` file:

```
<EXTERNAL-IP> nginx.example.com
```

Replace `<EXTERNAL-IP>` with the actual IP address of the Ingress Controller.

---

### **Step 6: Test the Setup**

Now, you can test the reverse proxy setup by accessing the application:

1. **Access the NGINX Application:**

   Open your browser and navigate to `http://nginx.example.com/`. You should see the default NGINX welcome page.

   Alternatively, you can use `curl`:

   ```bash
   curl http://nginx.example.com/
   ```

   This should return the default HTML page of the NGINX web server.

---

### **Step 7: Test Reverse Proxy and Path-Based Routing**

To test path-based routing, modify the Ingress resource to add path-based routing rules.

For example, you can route traffic based on `/app1` and `/app2` paths to the NGINX service. Edit the `nginx-ingress.yaml` file:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: nginx-ingress
  namespace: default
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
spec:
  rules:
    - host: nginx.example.com
      http:
        paths:
          - path: /app1
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
          - path: /app2
            pathType: Prefix
            backend:
              service:
                name: nginx-service
                port:
                  number: 80
```

Apply the updated Ingress resource:

```bash
kubectl apply -f nginx-ingress.yaml
```

Now, test the routing:

1. `http://nginx.example.com/app1` should route to the NGINX application.
2. `http://nginx.example.com/app2` should also route to the same NGINX application (you can modify paths for testing different services).

---

### **Step 8: Clean Up Resources**

Once you're done testing, you can clean up all the resources created during the exercise:

```bash
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
kubectl delete -f nginx-ingress.yaml
kubectl delete namespace ingress-nginx
```

---

### **Summary**

In this exercise:
- We manually deployed the **NGINX Ingress Controller** in Kubernetes without using Helm.
- We deployed a simple **NGINX application** and exposed it using a Kubernetes Service.
- We created an **Ingress resource** to define how traffic should be routed to the NGINX service.
- We tested the setup by accessing the service through a domain and verified path-based routing.
