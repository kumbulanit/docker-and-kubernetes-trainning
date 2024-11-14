### Practical Exercise: Kubernetes Network Policies

In Kubernetes, **Network Policies** are used to control the communication between pods and/or services based on specific rules. These rules allow you to specify which pods can communicate with each other, effectively controlling inbound and outbound traffic between them.

This exercise walks through creating a **NetworkPolicy** that restricts traffic between pods in a Kubernetes cluster. We will create two pods, apply a network policy to restrict access, and then verify the behavior by testing the network connectivity between the pods.

### **Prerequisites:**

- Kubernetes cluster (Minikube, kubeadm, or any other setup).
- Calico or another CNI (Container Network Interface) plugin that supports network policies (Calico is often used for network policies, and it must be enabled in your cluster).

### **Step 1: Create Two Example Pods**

To test the network policy, we need at least two pods. One pod will be allowed to communicate with the other, and the other will be blocked.

1. **Create a YAML file for the first pod (`pod-1.yaml`)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-1
spec:
  containers:
  - name: nginx
    image: nginx
```

2. **Create a YAML file for the second pod (`pod-2.yaml`)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: pod-2
spec:
  containers:
  - name: nginx
    image: nginx
```

### **Step 2: Deploy the Pods**

Apply both YAML files to deploy the pods:

```bash
kubectl apply -f pod-1.yaml
kubectl apply -f pod-2.yaml
```

Verify that both pods are running:

```bash
kubectl get pods
```

- **Expected Output**: You should see both pods listed with `STATUS` as `Running`.

### **Step 3: Create the Network Policy**

Now, we'll create a **NetworkPolicy** that only allows traffic from `pod-1` to `pod-2`. This means `pod-2` will be blocked from accessing any other pods in the cluster, and `pod-1` will be allowed to access `pod-2`.

1. **Create a NetworkPolicy YAML file (`network-policy.yaml`)**:

```yaml
apiVersion: networking.k8s.io/v1
kind: NetworkPolicy
metadata:
  name: allow-pod-1-to-pod-2
spec:
  podSelector:
    matchLabels: {}  # Select all pods in the namespace
  ingress:
  - from:
    - podSelector:
        matchLabels:
          name: pod-1  # Only allow pod-1 to communicate with pod-2
  policyTypes:
  - Ingress
```

### **Explanation of the NetworkPolicy**:
- **`podSelector`**: Selects all the pods in the namespace. This means the policy applies to all pods in the namespace.
- **`ingress.from.podSelector.matchLabels`**: Specifies that only traffic coming from pods with the label `name: pod-1` is allowed to ingress to the selected pods (`pod-2` in this case).
- **`policyTypes: Ingress`**: Specifies that this policy applies to inbound traffic.

### **Step 4: Apply the Network Policy**

Apply the network policy to the cluster:

```bash
kubectl apply -f network-policy.yaml
```

### **Step 5: Test the Network Policy**

Now we will test the behavior of the network policy to ensure that `pod-1` can communicate with `pod-2`, but `pod-2` cannot initiate a connection back to `pod-1` or any other pods.

#### 1. **Test from `pod-1` to `pod-2`:**

1. Use `kubectl exec` to enter the `pod-1` container and test connectivity:

```bash
kubectl exec -it pod-1 -- curl -s pod-2:80
```

- **Expected Output**: You should receive a successful response from `pod-2`, as the policy allows traffic from `pod-1` to `pod-2`.

#### 2. **Test from `pod-2` to `pod-1`:**

1. Similarly, use `kubectl exec` to enter the `pod-2` container and test connectivity to `pod-1`:

```bash
kubectl exec -it pod-2 -- curl -s pod-1:80
```

- **Expected Output**: This request should fail (connection refused or timeout), as the network policy blocks traffic from `pod-2` to `pod-1`.

#### 3. **Test connectivity from `pod-2` to any other pods:**

1. Try to curl another pod, for example `kubernetes-dashboard` (if installed) or any other running pod in the cluster:

```bash
kubectl exec -it pod-2 -- curl -s <any-other-pod-ip>:<port>
```

- **Expected Output**: The request should fail because `pod-2` is restricted by the network policy and cannot initiate any connections to other pods.

#### 4. **Check the Network Policy Status:**

You can also check the status of the NetworkPolicy by describing it:

```bash
kubectl describe networkpolicy allow-pod-1-to-pod-2
```

- **Expected Output**: You will see details about the policy, including the pods it applies to and the ingress rules.

### **Step 6: Clean Up**

Once you've finished testing, you can clean up the resources:

1. **Delete the pods**:

```bash
kubectl delete -f pod-1.yaml
kubectl delete -f pod-2.yaml
```

2. **Delete the NetworkPolicy**:

```bash
kubectl delete -f network-policy.yaml
```

---

### **Conclusion**

In this exercise, you:
1. Created two pods in the Kubernetes cluster.
2. Applied a **NetworkPolicy** to allow traffic from one pod to another and restricted the opposite direction.
3. Verified the policy's behavior by testing network connectivity between the pods.
4. Cleaned up the resources.

The key takeaway is that **NetworkPolicies** provide a powerful way to control pod-to-pod communication in a Kubernetes cluster. You can use these policies to secure your cluster by restricting which pods can communicate with each other based on specific labels and namespaces.