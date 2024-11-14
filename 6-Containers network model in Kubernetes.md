### **Exercise 1: Pod-to-Pod Communication**

**Objective:** Demonstrate how Pods can communicate with each other using their IP addresses within a Kubernetes cluster.

1. **Create Two Pods in the Same Namespace:**
   - Use `kubectl` to create two Pods: `pod-frontend` and `pod-backend`.
   - The frontend Pod will attempt to access the backend Pod via its Pod IP.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-frontend
spec:
  containers:
  - name: frontend
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-backend
spec:
  containers:
  - name: backend
    image: nginx
```

2. **Access the Backend Pod from the Frontend Pod:**
   - Once the Pods are running, `kubectl exec` into the frontend Pod and try to ping the backend Pod using its IP address.

```bash
kubectl exec -it pod-frontend -- /bin/bash
```

3. **Verify Communication:**
   - Inside the `frontend` Pod, find the IP address of the `backend` Pod by running:

```bash
kubectl get pod pod-backend -o wide
```

   - Ping the `backend` Pod from the `frontend` Pod using the backend’s IP address:

```bash
ping <backend-pod-ip>
```

---

### **Exercise 2: Service Discovery with DNS**

**Objective:** Show how Kubernetes DNS resolves Services to Pods using DNS names.

1. **Create a Simple Service:**
   - Create a Service for the `pod-backend`:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-service
spec:
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

2. **Pod to Service Communication:**
   - Use `kubectl exec` to access the `frontend` Pod again and try to access the backend Service using the Service’s DNS name (`backend-service.default.svc.cluster.local`):

```bash
kubectl exec -it pod-frontend -- /bin/bash
```

3. **Test Service Discovery:**
   - Inside the `frontend` Pod, run `curl` to access the backend service via its DNS name:

```bash
curl backend-service.default.svc.cluster.local:8080
```

---

### **Exercise 3: Exposing Services with NodePort**

**Objective:** Expose a Service to the outside world using NodePort.

1. **Create a NodePort Service:**
   - Modify the `backend-service` to expose it externally using a `NodePort`.

```yaml
apiVersion: v1
kind: Service
metadata:
  name: backend-nodeport
spec:
  type: NodePort
  selector:
    app: backend
  ports:
    - protocol: TCP
      port: 8080
      targetPort: 8080
      nodePort: 30080
```

2. **Access Service from Outside the Cluster:**
   - Get the IP address of one of the worker nodes in your cluster using:

```bash
kubectl get nodes -o wide
```

   - Open a browser or use `curl` to access the service via `http://<node-ip>:30080`.

---

### **Exercise 4: Pod-to-Pod Communication Across Nodes**

**Objective:** Show how Pods running on different nodes can still communicate.

1. **Deploy Pods on Different Nodes:**
   - Use **affinity** rules to place the `frontend` and `backend` Pods on different nodes:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-frontend
spec:
  containers:
  - name: frontend
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: NotIn
            values:
            - <node-1-name>
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-backend
spec:
  containers:
  - name: backend
    image: nginx
  affinity:
    nodeAffinity:
      requiredDuringSchedulingIgnoredDuringExecution:
        nodeSelectorTerms:
        - matchExpressions:
          - key: kubernetes.io/hostname
            operator: NotIn
            values:
            - <node-2-name>
```

2. **Verify Communication Across Nodes:**
   - Verify that both Pods are running on different nodes.
   - Use `kubectl exec` to check if the `frontend` Pod can access the `backend` Pod via the Service or its Pod IP, even though they are on different nodes.

---

### **Exercise 5: Implementing Network Policies**

**Objective:** Control traffic flow between Pods using Network Policies.

1. **Create Network Policies to Restrict Access:**
   - Create a policy that allows traffic from the `frontend` Pods to the `backend` Pods but denies other Pods from accessing the backend.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-frontend-to-backend
spec:
  podSelector:
    matchLabels:
      app: backend
  ingress:
  - from:
    - podSelector:
        matchLabels:
          app: frontend
```

2. **Apply the Policy:**
   - Apply the NetworkPolicy to the cluster.

```bash
kubectl apply -f network-policy.yaml
```

3. **Test the Policy:**
   - Use `kubectl exec` to try to access the `backend` Pod from a Pod other than the `frontend`. It should fail due to the network policy.

```bash
kubectl exec -it pod-other -- /bin/bash
```

---

### **Exercise 6: Ingress Resource for External Access**

**Objective:** Use an Ingress resource to expose a Service to external HTTP traffic.

1. **Install an Ingress Controller:**
   - Install the NGINX Ingress controller if it is not already set up.

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/main/deploy/static/provider/cloud/deploy.yaml
```

2. **Create an Ingress Resource:**
   - Create an Ingress resource to expose the `frontend` service externally:

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: frontend-ingress
spec:
  rules:
  - host: my-frontend-app.example.com
    http:
      paths:
      - path: /
        pathType: Prefix
        backend:
          service:
            name: frontend-service
            port:
              number: 80
```

3. **Access the Application via Ingress:**
   - Update your DNS settings to point `my-frontend-app.example.com` to the Ingress controller’s external IP.
   - Try to access the frontend application via the browser at `http://my-frontend-app.example.com`.

---

### **Exercise 7: Troubleshooting Pod Networking**

**Objective:** Troubleshoot Pod communication issues.

1. **Create Two Pods with Incorrect Network Policies:**
   - Create two Pods and apply a NetworkPolicy that restricts traffic between them.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
spec:
  containers:
  - name: app
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
spec:
  containers:
  - name: app
    image: nginx
```

2. **Check Network Communication:**
   - Try pinging `pod-b` from `pod-a`. It should fail due to the network policy.

```bash
kubectl exec -it pod-a -- /bin/bash
ping pod-b
```

3. **Check and Modify Network Policies:**
   - Use `kubectl describe` to check the policy and update it to allow traffic between the Pods.

---

#