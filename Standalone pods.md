### Practical Exercise: Working with Standalone Pods in Kubernetes

In Kubernetes, **Pods** are the smallest and simplest unit of deployment. A **Standalone Pod** is a pod that runs independently without being part of a deployment or replica set. It can be useful for simple tasks or to test an application in a single pod, though it does not provide features like scaling or self-healing that come with Deployments.

In this exercise, we'll create a simple **Standalone Pod** running a basic web server using `nginx`.

---

### **Step 1: Define the Standalone Pod**

1. **Create the Pod definition file (`standalone-pod.yaml`)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-standalone
  labels:
    app: nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

- `apiVersion: v1`: Specifies the Kubernetes API version.
- `kind: Pod`: The type of resource, in this case, a Pod.
- `metadata`: Metadata about the pod, including its name and labels.
- `spec`: Specifies the pod's specification, including containers and ports. In this case, a single container running the `nginx` image on port 80.

---

### **Step 2: Apply the Pod Definition**

To create the pod from the YAML file, run the following command:

```bash
kubectl apply -f standalone-pod.yaml
```

This will create a pod named `nginx-standalone` in the default namespace.

---

### **Step 3: Verify Pod Creation**

Check the status of the pod to make sure it was created successfully:

```bash
kubectl get pods
```

You should see the `nginx-standalone` pod listed. The `STATUS` should show as `Running` once the pod is up and running.

---

### **Step 4: Access the Pod's Application**

To interact with the pod, you need to expose the web server inside the pod. Since the pod is standalone, we will use `kubectl port-forward` to forward the pod’s port to your local machine.

Run the following command to forward port 80 of the pod to port 8080 on your local machine:

```bash
kubectl port-forward pod/nginx-standalone 8080:80
```

This command forwards the pod's port `80` to your local machine's port `8080`.

Now, open a browser and navigate to `http://localhost:8080`. You should see the default `nginx` welcome page, indicating that the pod is serving content.

---

### **Step 5: Interact with the Pod's Logs**

To view the logs of the `nginx` container running inside the pod, use:

```bash
kubectl logs nginx-standalone
```

You should see logs related to the `nginx` web server.

---

### **Step 6: Clean Up the Pod**

Once you are done testing the standalone pod, you can delete it using:

```bash
kubectl delete pod nginx-standalone
```

This will remove the pod from your cluster.

---

### **Conclusion**

In this exercise, we have:

- Created a **Standalone Pod** running a simple `nginx` web server.
- Exposed the pod to access it via `kubectl port-forward`.
- Verified the pod's functionality by checking its logs and accessing the web server through a browser.
- Cleaned up the resources by deleting the pod.

