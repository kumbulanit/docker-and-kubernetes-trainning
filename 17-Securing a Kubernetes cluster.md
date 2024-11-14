### Practical Exercise: Securing a Kubernetes Cluster

In this exercise, we will walk through several common security best practices to secure a Kubernetes cluster. We will cover various aspects of securing the cluster, including:

- Enabling Role-Based Access Control (RBAC)
- Enabling Network Policies
- Restricting access to the Kubernetes API Server
- Securing sensitive data with Secrets management
- Ensuring pod security using security contexts and pod policies

At the end of this exercise, we will verify the applied security measures to ensure they are working as intended.

### **Prerequisites:**

- A working Kubernetes cluster (can use **Minikube** for this exercise).
- `kubectl` command-line tool.
- Basic understanding of Kubernetes concepts such as RBAC, Network Policies, Secrets, and Pods.

### **Step 1: Enable Role-Based Access Control (RBAC)**

Role-Based Access Control (RBAC) is an important mechanism to restrict who can access and modify resources in a Kubernetes cluster. By default, RBAC should be enabled in a Kubernetes cluster, but let’s verify it and configure a simple RBAC policy.

#### 1. **Create a ClusterRole and ClusterRoleBinding**

Create a YAML file for a ClusterRole (`rbac-role.yaml`) that defines permissions for a user. We’ll grant the user permissions to list pods in the cluster.

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  # Name the role
  name: pod-reader
rules:
  - apiGroups: [""]
    resources: ["pods"]
    verbs: ["list", "get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: pod-reader-binding
subjects:
  - kind: User
    name: "test-user"
    apiGroup: rbac.authorization.k8s.io
roleRef:
  kind: ClusterRole
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

This YAML file defines a `ClusterRole` with permissions to list and get pods, and a `ClusterRoleBinding` that binds the role to a user named `test-user`.

#### 2. **Apply the RBAC configuration**

Apply the YAML file to your cluster:

```bash
kubectl apply -f rbac-role.yaml
```

#### 3. **Test the RBAC permissions**

To verify the RBAC configuration, try accessing the Kubernetes API with the `test-user` credentials (assuming you’ve configured your authentication method to support this user, such as through `kubectl` configurations or an external identity provider).

```bash
kubectl auth can-i list pods --as=test-user
```

- **Expected Output**: The output should be `yes` if the user `test-user` has the required permissions to list pods.

### **Step 2: Enabling Network Policies**

Network Policies control the communication between pods. By default, all pod communication is allowed in Kubernetes. Let’s create a network policy to restrict traffic between pods.

#### 1. **Create a NetworkPolicy**

Create a YAML file for a **NetworkPolicy** (`network-policy.yaml`) that denies all ingress traffic to pods in a specific namespace except from a designated source.

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: deny-all-traffic
spec:
  podSelector: {}
  policyTypes:
  - Ingress
  ingress: []
```

This network policy will block all inbound traffic to any pod in the cluster. You can modify the policy to selectively allow traffic from specific sources by adding `from` clauses under the `ingress` field.

#### 2. **Apply the NetworkPolicy**

Apply the network policy to your cluster:

```bash
kubectl apply -f network-policy.yaml
```

#### 3. **Test the NetworkPolicy**

To test the network policy, create two pods (`pod-a` and `pod-b`) and check connectivity between them.

1. **Create the Pods (`pod-a.yaml` and `pod-b.yaml`)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-a
spec:
  containers:
  - name: nginx
    image: nginx
---
apiVersion: v1
kind: Pod
metadata:
  name: pod-b
spec:
  containers:
  - name: nginx
    image: nginx
```

2. **Deploy the pods**:

```bash
kubectl apply -f pod-a.yaml
kubectl apply -f pod-b.yaml
```

3. **Test connectivity between the pods**:

Test connectivity from `pod-a` to `pod-b`:

```bash
kubectl exec -it pod-a -- curl pod-b
```

- **Expected Output**: The connection should be refused because the `deny-all-traffic` network policy blocks traffic between the pods.

### **Step 3: Securing Kubernetes Secrets**

Kubernetes Secrets store sensitive data like passwords, OAuth tokens, and SSH keys. We can create a secret to store sensitive data and restrict access to it.

#### 1. **Create a Secret**

Create a YAML file for a Kubernetes Secret (`secret.yaml`):

```yaml
apiVersion: v1
kind: Secret
metadata:
  name: db-password
type: Opaque
data:
  password: cGFzc3dvcmQxMjM=  # base64 encoded value for 'password123'
```

#### 2. **Apply the Secret**

Apply the secret to the cluster:

```bash
kubectl apply -f secret.yaml
```

#### 3. **Test accessing the Secret**

Test accessing the secret by retrieving it:

```bash
kubectl get secret db-password -o yaml
```

- **Expected Output**: The secret's data will be base64-encoded, and you will need to decode it to view the plain text:

```bash
echo "cGFzc3dvcmQxMjM=" | base64 --decode
```

- **Expected Output**: `password123`

### **Step 4: Securing Pod Access with Security Contexts**

Security Contexts allow you to define security settings for your pods. For example, you can specify that the pod should run as a non-root user.

#### 1. **Create a Pod with a Security Context**

Create a YAML file (`pod-with-security-context.yaml`) to define a pod that runs with non-root privileges:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: secure-pod
spec:
  securityContext:
    runAsUser: 1000
    fsGroup: 2000
  containers:
  - name: nginx
    image: nginx
```

This configuration ensures that the pod runs as a non-root user with a user ID of `1000` and a file system group ID of `2000`.

#### 2. **Apply the Pod with the Security Context**

Apply the pod configuration:

```bash
kubectl apply -f pod-with-security-context.yaml
```

#### 3. **Verify the Pod's Security Context**

Check the pod’s security context:

```bash
kubectl get pod secure-pod -o jsonpath='{.spec.securityContext}'
```

- **Expected Output**: You should see that the `runAsUser` is `1000` and `fsGroup` is `2000`.

### **Step 5: Securing the API Server**

The API Server is the entry point for interacting with Kubernetes. To secure it, you should:

- Enable HTTPS and use certificates.
- Restrict access using RBAC and authentication mechanisms.
- Ensure your API Server is behind a firewall or only accessible to authorized users.

#### 1. **Test Access Control for the API Server**

Attempt to list resources as an unauthorized user:

```bash
kubectl auth can-i get pods --as=unauthorized-user
```

- **Expected Output**: The request should be denied (`no`), as the `unauthorized-user` does not have the required permissions.

### **Step 6: Clean Up**

Once you are finished, delete all the resources to clean up:

```bash
kubectl delete -f rbac-role.yaml
kubectl delete -f network-policy.yaml
kubectl delete -f secret.yaml
kubectl delete -f pod-with-security-context.yaml
```

---

### **Conclusion**

In this exercise, you have:
1. Secured your Kubernetes cluster by implementing RBAC policies.
2. Configured a NetworkPolicy to control communication between pods.
3. Managed sensitive data securely with Kubernetes Secrets.
4. Applied a security context to ensure pods run with non-root privileges.
5. Verified and tested the effectiveness of these security measures.

