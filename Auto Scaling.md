### Practical Exercise: Auto Scaling in Kubernetes

In Kubernetes, **Auto Scaling** allows you to automatically scale your applications based on resource usage or other metrics. In this exercise, we will demonstrate two common types of auto-scaling:

1. **Horizontal Pod Autoscaler (HPA)**: Automatically scales the number of pod replicas based on CPU or memory usage.
2. **Cluster Autoscaler**: Automatically adjusts the number of nodes in your cluster based on the resource usage and demand.

### **Prerequisites:**

- Minikube or a working Kubernetes cluster.
- `kubectl` command-line tool installed and configured.
- A basic understanding of Kubernetes Pods, Deployments, and Metrics Server.

### **Steps to Implement Auto Scaling in Kubernetes:**

#### **Step 1: Set up Metrics Server**

To enable autoscaling in Kubernetes, you need the **Metrics Server** installed, as it collects resource metrics from the nodes and pods, which are required for Horizontal Pod Autoscaler (HPA).

1. **Install Metrics Server**:

   Run the following command to install the Metrics Server:

   ```bash
   kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
   ```

2. **Verify the Metrics Server is running**:

   To ensure the Metrics Server is working, check its pods:

   ```bash
   kubectl get pods -n kube-system
   ```

   - **Expected Output**: You should see a pod with a name starting with `metrics-server`, and it should be in the `Running` state.

#### **Step 2: Create a Deployment**

Next, we'll create a simple deployment with a web server (like NGINX) to demonstrate how auto-scaling works.

1. **Create a simple NGINX deployment**:

   Create a file called `nginx-deployment.yaml`:

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
             image: nginx
             ports:
               - containerPort: 80
   ```

2. **Deploy it**:

   Apply the deployment configuration to your Kubernetes cluster:

   ```bash
   kubectl apply -f nginx-deployment.yaml
   ```

3. **Verify the deployment**:

   Check that the NGINX deployment is created and running:

   ```bash
   kubectl get deployments
   kubectl get pods
   ```

   - **Expected Output**: You should see the `nginx-deployment` with 2 replicas running.

#### **Step 3: Create a Horizontal Pod Autoscaler (HPA)**

The Horizontal Pod Autoscaler (HPA) automatically adjusts the number of replicas based on CPU or memory usage.

1. **Create an HPA for the NGINX deployment**:

   Run the following command to create an HPA that will scale the NGINX pods based on CPU utilization.

   ```bash
   kubectl autoscale deployment nginx-deployment --cpu-percent=50 --min=2 --max=10
   ```

   This command creates an HPA that will:
   - Monitor the CPU usage of the pods in the `nginx-deployment`.
   - Scale the deployment between 2 and 10 replicas based on the CPU utilization (targeting 50% CPU usage per pod).

2. **Verify the Horizontal Pod Autoscaler**:

   Check the status of the HPA:

   ```bash
   kubectl get hpa
   ```

   - **Expected Output**: You should see the `nginx-deployment` HPA with current CPU utilization and replica count.

#### **Step 4: Simulate Load and Observe Autoscaling**

To test the auto-scaling functionality, we'll simulate load on the NGINX deployment by running an HTTP stress test tool like `ab` (Apache Benchmark).

1. **Install Apache Benchmark** (if not already installed):

   On Ubuntu, you can install it with the following command:

   ```bash
   sudo apt install apache2-utils
   ```

2. **Run a stress test**:

   Use Apache Benchmark to generate load on the NGINX deployment:

   ```bash
   ab -n 1000 -c 100 http://<minikube-ip>:<nginx-service-port>/
   ```

   Replace `<minikube-ip>` with the IP of your Minikube cluster and `<nginx-service-port>` with the port where NGINX is exposed.

3. **Check the autoscaler behavior**:

   After a few moments, you should see the number of replicas increase as the CPU utilization goes up. You can check the status of the pods and the HPA again:

   ```bash
   kubectl get pods
   kubectl get hpa
   ```

   - **Expected Output**: The number of pods in the `nginx-deployment` should increase if CPU utilization exceeds the 50% threshold, and the HPA should reflect the new scaling behavior.

#### **Step 5: Clean up Resources**

Once you’ve tested the auto-scaling behavior, you can delete the resources created.

1. **Delete the HPA**:

   ```bash
   kubectl delete hpa nginx-deployment
   ```

2. **Delete the deployment**:

   ```bash
   kubectl delete -f nginx-deployment.yaml
   ```

---

### **Optional: Cluster Autoscaler (On Cloud Providers)**

The **Cluster Autoscaler** automatically adjusts the number of nodes in your Kubernetes cluster when resource requests exceed the capacity of the current nodes.

- To enable **Cluster Autoscaler**, you'll need a cloud provider setup (e.g., AWS, GCP, Azure) with the autoscaler enabled. The installation and configuration for the **Cluster Autoscaler** depend on the provider and can be complex, so it is not covered in this Minikube exercise.

---

### **Conclusion**

In this exercise, you:
1. Installed the **Metrics Server** to enable resource-based auto-scaling.
2. Created a simple **NGINX Deployment** to test scaling.
3. Set up a **Horizontal Pod Autoscaler** to scale the number of replicas based on CPU usage.
4. Simulated load using **Apache Benchmark** to trigger auto-scaling.
5. Cleaned up the resources.

This demonstrates how **Horizontal Pod Autoscaling** works in Kubernetes. For **Cluster Autoscaling**, you can explore it in cloud environments with multiple nodes and more advanced configurations.