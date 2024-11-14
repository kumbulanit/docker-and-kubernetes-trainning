Here are two different scenarios that demonstrate **Authentication, Authorization, and Access Control** in a Kubernetes cluster using Minikube, highlighting different access restrictions:

### Scenario 1: Read-Only Access to Pods in a Specific Namespace
In this scenario, we'll create a service account that has **read-only access** to Pods in a specific namespace (`demo`), restricting its permissions to only **view** Pod details but **not modify them**.

### Scenario 2: Admin Access to Deployments but Restricted Access to Pods
In this scenario, we'll set up a service account that has **admin access to manage Deployments** but only has **read-only access to Pods** within the same namespace. This demonstrates a scenario where a user can manage applications via Deployments without having full control over the underlying Pods.

### **Scenario 1: Read-Only Access to Pods in `demo` Namespace**

#### 1. Create a Namespace
Create a `demo` namespace:
```bash
kubectl create namespace demo
```

#### 2. Create a Service Account
Create a service account named `readonly-sa` in the `demo` namespace:
```bash
kubectl create serviceaccount readonly-sa -n demo
```

#### 3. Create a Role with Read-Only Access to Pods
Create a `pod-reader` role to give read-only access to Pods in the `demo` namespace:
```yaml
# pod-reader-role.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: Role
metadata:
  namespace: demo
  name: pod-reader
rules:
- apiGroups: [""]
  resources: ["pods"]
  verbs: ["get", "list"]
```

Apply the role:
```bash
kubectl apply -f pod-reader-role.yaml
```

#### 4. Bind the Role to the Service Account
Bind the `pod-reader` role to the `readonly-sa` service account:
```yaml
# pod-reader-rolebinding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: demo
subjects:
- kind: ServiceAccount
  name: readonly-sa
  namespace: demo
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the role binding:
```bash
kubectl apply -f pod-reader-rolebinding.yaml
```

#### 5. Deploy a Test Pod
Deploy a sample Pod in the `demo` namespace:
```yaml
# nginx-pod.yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
```

Apply the Pod configuration:
```bash
kubectl apply -f nginx-pod.yaml
```

#### 6. Test Access Control
Authenticate using the `readonly-sa` service account:
```bash
SECRET_NAME=$(kubectl get sa readonly-sa -n demo -o jsonpath="{.secrets[0].name}")
TOKEN=$(kubectl get secret $SECRET_NAME -n demo -o jsonpath="{.data.token}" | base64 --decode)
kubectl config set-credentials readonly-user --token=$TOKEN
kubectl config set-context readonly-context --cluster=minikube --user=readonly-user --namespace=demo
kubectl config use-context readonly-context
```

- List the Pods: `kubectl get pods` (Should succeed)
- Try deleting a Pod: `kubectl delete pod nginx` (Should fail due to insufficient permissions)

### **Scenario 2: Admin Access to Deployments, Limited Access to Pods**

#### 1. Create a Namespace
Use the `demo` namespace from the previous example:
```bash
kubectl create namespace demo
```

#### 2. Create a Service Account
Create another service account called `deployment-admin-sa` in the `demo` namespace:
```bash
kubectl create serviceaccount deployment-admin-sa -n demo
```

#### 3. Create Roles for Deployment Admin and Pod Reader
1. **Deployment Admin Role**:
   ```yaml
   # deployment-admin-role.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: demo
     name: deployment-admin
   rules:
   - apiGroups: ["apps"]
     resources: ["deployments"]
     verbs: ["create", "delete", "get", "list", "update", "patch"]
   ```

   Apply the role:
   ```bash
   kubectl apply -f deployment-admin-role.yaml
   ```

2. **Pod Reader Role** (if not created already in the previous scenario):
   ```yaml
   # pod-reader-role.yaml
   apiVersion: rbac.authorization.k8s.io/v1
   kind: Role
   metadata:
     namespace: demo
     name: pod-reader
   rules:
   - apiGroups: [""]
     resources: ["pods"]
     verbs: ["get", "list"]
   ```

   Apply the role:
   ```bash
   kubectl apply -f pod-reader-role.yaml
   ```

#### 4. Bind the Roles to the Service Account
Create two RoleBindings: one for Deployment admin and another for Pod Reader:
```yaml
# deployment-admin-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: deployment-admin-binding
  namespace: demo
subjects:
- kind: ServiceAccount
  name: deployment-admin-sa
  namespace: demo
roleRef:
  kind: Role
  name: deployment-admin
  apiGroup: rbac.authorization.k8s.io
```

```yaml
# pod-reader-binding.yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: RoleBinding
metadata:
  name: pod-reader-binding
  namespace: demo
subjects:
- kind: ServiceAccount
  name: deployment-admin-sa
  namespace: demo
roleRef:
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
```

Apply the bindings:
```bash
kubectl apply -f deployment-admin-binding.yaml
kubectl apply -f pod-reader-binding.yaml
```

#### 5. Deploy a Test Deployment
Deploy an `nginx` Deployment:
```yaml
# nginx-deployment.yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: nginx-deployment
  namespace: demo
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
        image: nginx
        ports:
        - containerPort: 80
```

Apply the deployment:
```bash
kubectl apply -f nginx-deployment.yaml
```

#### 6. Test Access Control
Authenticate using the `deployment-admin-sa` service account:
```bash
SECRET_NAME=$(kubectl get sa deployment-admin-sa -n demo -o jsonpath="{.secrets[0].name}")
TOKEN=$(kubectl get secret $SECRET_NAME -n demo -o jsonpath="{.data.token}" | base64 --decode)
kubectl config set-credentials deployment-admin-user --token=$TOKEN
kubectl config set-context deployment-admin-context --cluster=minikube --user=deployment-admin-user --namespace=demo
kubectl config use-context deployment-admin-context
```

- **List Deployments**: `kubectl get deployments` (Should succeed)
- **Update Deployment**: `kubectl scale deployment nginx-deployment --replicas=3` (Should succeed)
- **List Pods**: `kubectl get pods` (Should succeed, read-only)
- **Delete Pod**: `kubectl delete pod nginx-xxxxx` (Should fail)

### Conclusion
These two scenarios showcase different ways to implement fine-grained access control using RBAC in Kubernetes:
1. Scenario 1 restricts access to read-only actions on Pods.
2. Scenario 2 provides admin rights for managing Deployments while limiting direct Pod control.