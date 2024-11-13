### Practical Working Example: Basic Kubernetes Commands on Minikube

#### **Prerequisites:**
- Minikube installed on your local machine.
- `kubectl` installed and configured to interact with your Minikube cluster.

### **Step 1: Start Minikube**

1. **Start the Minikube Cluster**:

   Open your terminal and start Minikube with:

   ```bash
   minikube start
   ```

   This will create a single-node Kubernetes cluster running locally.

2. **Check Cluster Status**:

   Ensure that Minikube is up and running:

   ```bash
   minikube status
   ```

### **Step 2: Viewing Kubernetes Nodes**

1. **View Available Nodes**:

   Display the nodes in the cluster:

   ```bash
   kubectl get nodes
   ```

   You should see a single node (`minikube`) listed, which is your local machine running as a node.

2. **Detailed Node Information**:

   To get detailed information about the node:

   ```bash
   kubectl describe node minikube
   ```

### **Step 3: Adding and Removing Nodes in Minikube**

1. **Adding Another Node**:

   By default, Minikube starts with a single node. To simulate adding nodes, use Minikube profiles to start a new instance:

   ```bash
   minikube start -p new-node
   ```

   Here, `-p` stands for profile. This creates a new Minikube instance (`new-node`). Now, you can switch between the nodes:

   ```bash
   minikube profile list
   ```

   To interact with the new node, switch context:

   ```bash
   kubectl config use-context new-node
   ```

2. **Removing a Node**:

   To remove the second instance:

   ```bash
   minikube delete -p new-node
   ```

   This removes the `new-node` instance.

### **Step 4: Working with Pods**

1. **Deploy a Simple Pod**:

   Create a `nginx` Pod using the `kubectl` run command:

   ```bash
   kubectl run my-nginx --image=nginx --port=80
   ```

   This command creates a pod named `my-nginx` running the Nginx web server.

2. **View Running Pods**:

   To see the pods running in your cluster:

   ```bash
   kubectl get pods
   ```

   You should see `my-nginx` in the list of running pods.

3. **View Detailed Pod Information**:

   Get more details about the pod:

   ```bash
   kubectl describe pod my-nginx
   ```

4. **Exposing the Pod as a Service**:

   Create a service to expose the `my-nginx` pod:

   ```bash
   kubectl expose pod my-nginx --type=NodePort --port=80
   ```

   This creates a service of type `NodePort` to access the `nginx` pod.

5. **Access the Nginx Service**:

   Use the following command to get the URL for accessing the service:

   ```bash
   minikube service my-nginx --url
   ```

   Open the URL in your browser to see the Nginx welcome page.

### **Step 5: Creating, Scaling, and Deleting Deployments**

1. **Create a Deployment**:

   Create a deployment using the `nginx` image:

   ```bash
   kubectl create deployment my-deployment --image=nginx
   ```

2. **Scaling a Deployment**:

   Scale the deployment to 3 replicas:

   ```bash
   kubectl scale deployment my-deployment --replicas=3
   ```

   Verify the scaling:

   ```bash
   kubectl get pods
   ```

   You should now see 3 instances of the `nginx` pods.

3. **Deleting the Deployment**:

   To delete the deployment:

   ```bash
   kubectl delete deployment my-deployment
   ```

   Confirm that the pods are deleted:

   ```bash
   kubectl get pods
   ```

### **Step 6: Viewing Logs and Executing Commands in Pods**

1. **Viewing Pod Logs**:

   Check the logs of a running pod:

   ```bash
   kubectl logs my-nginx
   ```

2. **Executing Commands Inside a Pod**:

   You can get a shell inside the `nginx` pod:

   ```bash
   kubectl exec -it my-nginx -- /bin/bash
   ```

   Exit the shell using the `exit` command.

### **Step 7: Creating, Editing, and Deleting Configurations**

1. **Create a ConfigMap**:

   Create a ConfigMap to store configuration data:

   ```bash
   kubectl create configmap my-config --from-literal=app.name=nginx-app
   ```

2. **View ConfigMap Details**:

   ```bash
   kubectl get configmap my-config -o yaml
   ```

3. **Editing a ConfigMap**:

   Edit the ConfigMap directly:

   ```bash
   kubectl edit configmap my-config
   ```

   Change `app.name` to another value and save the file.

4. **Deleting the ConfigMap**:

   ```bash
   kubectl delete configmap my-config
   ```

### **Step 8: Handling YAML Manifests**

1. **Create a Pod using YAML**:

   Create a file named `my-nginx-pod.yaml`:

   ```yaml
   apiVersion: v1
   kind: Pod
   metadata:
     name: my-nginx-yaml
   spec:
     containers:
       - name: nginx
         image: nginx
         ports:
           - containerPort: 80
   ```

2. **Apply the YAML File**:

   ```bash
   kubectl apply -f my-nginx-pod.yaml
   ```

3. **Verify Pod Creation**:

   ```bash
   kubectl get pods
   ```

4. **Delete the Pod Using YAML**:

   ```bash
   kubectl delete -f my-nginx-pod.yaml
   ```

### **Step 9: Using Namespaces**

1. **Create a Namespace**:

   ```bash
   kubectl create namespace my-namespace
   ```

2. **Deploy a Pod in the New Namespace**:

   ```bash
   kubectl run nginx-in-namespace --image=nginx --namespace=my-namespace
   ```

3. **View Pods in the New Namespace**:

   ```bash
   kubectl get pods -n my-namespace
   ```

4. **Delete the Namespace**:

   ```bash
   kubectl delete namespace my-namespace
   ```

### **Step 10: Shutting Down Minikube**

When you're finished experimenting, you can stop the Minikube cluster:

```bash
minikube stop
```

And delete it if you no longer need it:

```bash
minikube delete
```

### **Summary**

This practical exercise covered essential Kubernetes commands for managing nodes and pods in Minikube. You learned how to:
- Start a cluster and view nodes.
- Create, scale, and delete pods and deployments.
- Expose services and access them.
- Use ConfigMaps, YAML manifests, and namespaces.

These commands provide a solid foundation for working with Kubernetes clusters in a real-world environment.