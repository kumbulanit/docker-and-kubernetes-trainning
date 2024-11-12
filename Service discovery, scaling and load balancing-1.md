

### **Exercise 1: Service Discovery in Kubernetes**

**Objective:** Show how Kubernetes Services enable Service Discovery for Pods.

#### **1. Create a Deployment for an Application**

First, we’ll create a deployment for a simple NGINX application.

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
          image: nginx:latest
          ports:
            - containerPort: 80
```

Apply the deployment to create the Pods:

```bash
kubectl apply -f nginx-deployment.yaml
```

#### **2. Create a Service for the Deployment**

Create a Kubernetes Service to expose the NGINX Pods. This will allow Pods to discover each other via DNS.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  clusterIP: None  # This creates a headless service for direct Pod-to-Pod discovery
```

Apply the service:

```bash
kubectl apply -f nginx-service.yaml
```

#### **3. Verify Service Discovery**

Check the Pods in the deployment and ensure the `nginx-service` resolves the Pods by using DNS.

1. First, get the Pod names:

```bash
kubectl get pods -l app=nginx
```

2. Next, test accessing one Pod from another (Pod-to-Pod communication via DNS):

```bash
kubectl exec -it <frontend-pod-name> -- /bin/bash
curl nginx-service.default.svc.cluster.local
```

You should see the NGINX default page, which indicates that the Service is routing traffic to one of the NGINX Pods.

---

### **Exercise 2: Scaling Applications in Kubernetes**

**Objective:** Demonstrate how to scale a deployment in Kubernetes.

#### **1. Create a Deployment with a Simple Application**

We’ll create a simple deployment for an NGINX application (already done in Exercise 1), but this time, we’ll scale it up and down dynamically.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  replicas: 2
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

Apply the deployment:

```bash
kubectl apply -f nginx-deployment.yaml
```

#### **2. Scale the Deployment**

To scale the deployment, we will change the number of replicas.

To scale up to 5 replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=5
```

Check the Pods:

```bash
kubectl get pods -l app=nginx
```

You should see 5 Pods running.

#### **3. Scale the Deployment Down**

To scale down to 2 replicas:

```bash
kubectl scale deployment nginx-deployment --replicas=2
```

Check again:

```bash
kubectl get pods -l app=nginx
```

You should see only 2 Pods running.

---

### **Exercise 3: Load Balancing with Kubernetes Services**

**Objective:** Demonstrate how Kubernetes Services automatically balance traffic across Pods.

#### **1. Create a Deployment**

We’ll use the same deployment we created in the scaling example.

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
          image: nginx:latest
          ports:
            - containerPort: 80
```

Apply it:

```bash
kubectl apply -f nginx-deployment.yaml
```

#### **2. Create a Service to Load Balance Traffic**

Now, create a Service to load balance traffic to the NGINX Pods.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx-service
spec:
  selector:
    app: nginx
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

Apply the service:

```bash
kubectl apply -f nginx-service.yaml
```

#### **3. Test Load Balancing**

To test load balancing, we will send multiple requests to the service and observe that traffic is distributed among the available Pods.

Use `kubectl port-forward` to forward traffic from your local machine to the service:

```bash
kubectl port-forward service/nginx-service 8080:80
```

Now, you can send requests to the service:

```bash
curl http://localhost:8080
```

You should see the NGINX default page, and if you keep sending requests, Kubernetes will distribute traffic to the available Pods (load balancing).

You can also watch the Pods being hit by the traffic:

```bash
kubectl logs -f <nginx-pod-name>
```

You should see different Pods handling different requests.

---

### **Exercise 4: Advanced Load Balancing with Multiple Services**

**Objective:** Demonstrate how Kubernetes can use multiple Services for advanced load balancing.

#### **1. Create Multiple Deployments**

Create two different deployments: one for the `frontend` and another for `backend`.

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: frontend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: frontend
  template:
    metadata:
      labels:
        app: frontend
    spec:
      containers:
        - name: frontend
          image: nginx
          ports:
            - containerPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: backend-deployment
spec:
  replicas: 2
  selector:
    matchLabels:
      app: backend
  template:
    metadata:
      labels:
        app: backend
    spec:
      containers:
        - name: backend
          image: nginx
          ports:
            - containerPort: 80
```

Apply the deployments:

```bash
kubectl apply -f frontend-deployment.yaml
kubectl apply -f backend-deployment.yaml
```

#### **2. Create Multiple Services**

Now, create two separate services: one for the frontend and one for the backend.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: frontend-service
spec:
  selector:
    app: frontend
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
---
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - port: 80
      targetPort: 80
  type: ClusterIP
```

Apply the services:

```bash
kubectl apply -f frontend-service.yaml
kubectl apply -f backend-service.yaml
```

#### **3. Load Balancing Between Services**

Use `kubectl port-forward` to forward traffic to both services:

```bash
kubectl port-forward service/frontend-service 8081:80
kubectl port-forward service/backend-service 8082:80
```

Test the load balancing by making multiple requests to each service:

```bash
curl http://localhost:8081
curl http://localhost:8082
```

Kubernetes will automatically load balance the traffic among Pods associated with each Service.

---

To effectively simulate load on your Kubernetes load balancer and test its capabilities, we can add a simple load-generating application or script that continuously sends HTTP requests to the service. This will help ensure the load balancer is distributing traffic to the available Pods.

Here’s how you can set this up using a **load generator** application (a simple HTTP load generator like `hey` or `wrk`) and deploy it in your Kubernetes cluster. We will also use a simple `curl` loop to generate traffic for load testing.

### **Step 1: Create a Load Testing Pod (Using `hey` or `wrk`)**

You can deploy a simple HTTP load testing application like `hey` or `wrk` inside a Kubernetes Pod. These tools will generate load on your `nginx-service` and simulate traffic to check how the load balancing works.

#### **1. Create a Load Generator Deployment**

Here, we use `hey`, which is a simple HTTP load generator, for testing. We'll create a Deployment for `hey` that will continuously send HTTP requests to the `nginx-service` deployed earlier.

Create a file named `load-generator.yaml`:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: load-generator
spec:
  replicas: 1
  selector:
    matchLabels:
      app: load-generator
  template:
    metadata:
      labels:
        app: load-generator
    spec:
      containers:
        - name: hey
          image: rakyll/hey
          args:
            - "-z"
            - "60s"  # Run for 60 seconds
            - "-c"
            - "10"   # Send 10 concurrent requests
            - "http://nginx-service.default.svc.cluster.local"  # The service URL
```

#### **2. Apply the Load Generator Deployment**

```bash
kubectl apply -f load-generator.yaml
```

This will create a `load-generator` Pod that continuously sends HTTP requests to the `nginx-service` for 60 seconds with 10 concurrent requests. It will simulate a constant load on the service, triggering load balancing between the Pods in the `nginx-service`.

#### **3. Monitor Load Generation**

You can check the logs of the `load-generator` to see the load test results:

```bash
kubectl logs -f deployment/load-generator
```

You will see output like this, showing the HTTP request statistics:

```
Requests/sec: 100
Concurrency: 10
Total Requests: 6000
```

#### **4. Observe Load Balancing**

While the load generator is running, you can monitor the Pods' logs and check how the traffic is being distributed between the Pods.

To check which Pods are receiving traffic, run:

```bash
kubectl logs -f <nginx-pod-name>
```

You should observe that the traffic is distributed among different Pods in the NGINX Deployment. The Kubernetes service is automatically balancing the load between these Pods.

---

### **Step 2: Create an External Load Generator (Alternative)**

Alternatively, if you prefer generating load externally (outside the Kubernetes cluster), you can use tools like `wrk` or `hey` from your local machine or a VM.

#### **1. Install `hey` on Your Local Machine**

You can install `hey` from the GitHub releases page:

- [Hey on GitHub](https://github.com/rakyll/hey/releases)

For example, on Linux:

```bash
wget https://github.com/rakyll/hey/releases/download/v0.1.0/hey-linux-amd64 -O hey
chmod +x hey
sudo mv hey /usr/local/bin/
```

#### **2. Run Load Testing**

Once installed, you can generate load by running the following command (replace `NGINX-SERVICE-IP` with the external IP address of your service):

```bash
hey -z 60s -c 10 http://NGINX-SERVICE-IP/
```

This will send 10 concurrent requests to the service for 60 seconds, simulating load from an external client.

You can find the external IP of the service using:

```bash
kubectl get svc nginx-service
```

---

### **Step 3: Use Kubernetes Horizontal Pod Autoscaling (Optional)**

To make the application automatically scale based on load, you can enable **Horizontal Pod Autoscaling (HPA)** in Kubernetes.

#### **1. Create an HPA Resource**

If your application is under heavy load, you can configure the autoscaler to scale your Pods automatically.

Create a file named `hpa.yaml`:

```yaml
apiVersion: autoscaling/v2
kind: HorizontalPodAutoscaler
metadata:
  name: nginx-hpa
  namespace: default
spec:
  scaleTargetRef:
    apiVersion: apps/v1
    kind: Deployment
    name: nginx-deployment
  minReplicas: 2
  maxReplicas: 10
  metrics:
    - type: Resource
      resource:
        name: cpu
        target:
          type: Utilization
          averageUtilization: 50  # Scale when average CPU usage exceeds 50%
```

#### **2. Apply HPA**

```bash
kubectl apply -f hpa.yaml
```

Now, the `nginx-deployment` will scale automatically based on CPU usage. If the load generator increases the CPU utilization, Kubernetes will increase the number of Pods in the deployment to handle the traffic.

To monitor the scaling activity, you can use:

```bash
kubectl get hpa
```

---

### **Step 4: Cleanup**

After the load testing is complete, don't forget to clean up the resources:

```bash
kubectl delete -f load-generator.yaml
kubectl delete hpa nginx-hpa
kubectl delete -f nginx-deployment.yaml
kubectl delete -f nginx-service.yaml
```

---

### **Conclusion**

With this setup, you are able to:

- **Generate load** on your Kubernetes service using an HTTP load generator (`hey`).
- **Monitor the load** distribution across the Pods.
- **Scale the application** dynamically using Horizontal Pod Autoscaling (HPA).
