### Practical Exercise: Kubernetes Cluster Management Using Minikube

In this exercise, we will manage a Kubernetes cluster using **Minikube**, a local Kubernetes environment that can be run on a single machine. This guide will show how to create a cluster, scale applications, monitor resources, and upgrade Kubernetes components. 



### **Step 1: Set Up the Kubernetes Cluster with Minikube**

#### **1.1 Install Minikube**

If you haven't installed Minikube yet, you can do so with the following commands:

1. **Update your package list:**
   ```bash
   sudo apt update
   sudo apt install -y curl apt-transport-https
   ```

2. **Download the latest Minikube binary:**
   ```bash
   curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
   ```

3. **Install Minikube:**
   ```bash
   sudo install minikube-linux-amd64 /usr/local/bin/minikube
   ```

4. **Verify the installation:**
   ```bash
   minikube version
   ```

#### **1.2 Start Minikube Cluster**

To start a single-node Kubernetes cluster with Minikube, run:

```bash
minikube start
```

This command will download the required Kubernetes components and start a local cluster on your system.

#### **1.3 Check Cluster Status**

Once Minikube starts, check the status of your cluster:

```bash
kubectl cluster-info
```

This will show the cluster's API server and dashboard URL.

---

### **Step 2: Manage Cluster Nodes**

Minikube is designed for a single-node setup, so you won’t need to manage multiple worker nodes directly. However, you can still manage the cluster’s status, add/remove components, and simulate a multi-node environment using different Minikube profiles.

#### **2.1 View the Cluster Nodes**

Minikube always creates a master node and runs all components on that node. To see the status of the cluster nodes:

```bash
kubectl get nodes
```

You should see something like this:

```bash
NAME       STATUS   ROLES    AGE     VERSION
minikube   Ready    master   2m      v1.21.0
```

---

### **Step 3: Scaling Applications**

Scaling applications is a critical part of Kubernetes management. In Minikube, you can scale applications just like you would in a full-fledged cluster.

#### **3.1 Create a Simple Deployment**

Let’s create a simple nginx deployment:

```bash
kubectl create deployment nginx --image=nginx
```

Check if the deployment is running:

```bash
kubectl get deployments
```

You should see an `nginx` deployment with one replica.

#### **3.2 Scale the Deployment**

To scale the deployment up or down, use the `kubectl scale` command. For example, scale the `nginx` deployment to 5 replicas:

```bash
kubectl scale deployment nginx --replicas=5
```

Verify the scaling:

```bash
kubectl get pods
```

You should see five nginx pods running.

#### **3.3 Decrease the Number of Replicas**

To scale down the deployment to 2 replicas, use the same command:

```bash
kubectl scale deployment nginx --replicas=2
```

Check the status:

```bash
kubectl get pods
```

You should now see only two `nginx` pods running.

---

### **Step 4: Cluster Upgrades**

Minikube allows you to upgrade Kubernetes within your local cluster.

#### **4.1 Check the Kubernetes Version**

Check the current version of Kubernetes running in Minikube:

```bash
kubectl version --short
```

You should see both the client and server versions.

#### **4.2 Upgrade Minikube**

To upgrade Minikube itself to the latest version, use:

```bash
sudo apt update
sudo apt upgrade minikube
```

Alternatively, if you installed Minikube via a binary (as shown earlier), you can download the latest version again and replace the binary.

#### **4.3 Upgrade Kubernetes Version**

To upgrade Kubernetes on Minikube:

1. **Stop Minikube**:
   ```bash
   minikube stop
   ```

2. **Upgrade the Kubernetes version**:
   ```bash
   minikube start --kubernetes-version=v1.21.2
   ```

3. **Verify the upgrade**:
   ```bash
   kubectl version --short
   ```

This should show the upgraded Kubernetes version.

---

### **Step 5: Cluster Monitoring**

To monitor the health and status of your Kubernetes cluster and its resources:

#### **5.1 View Cluster Component Status**

Check the status of core components (such as the controller manager and scheduler) with:

```bash
kubectl get componentstatuses
```

You should see:

```bash
NAME                 STATUS    MESSAGE              ERROR
controller-manager   Healthy   ok
scheduler            Healthy   ok
etcd-0               Healthy   ok
```

#### **5.2 Install Metrics Server**

Metrics Server is necessary for viewing resource usage (CPU, memory) in the cluster. Install it with:

```bash
kubectl apply -f https://github.com/kubernetes-sigs/metrics-server/releases/latest/download/components.yaml
```

#### **5.3 View Resource Usage**

Once the Metrics Server is installed, you can check the CPU and memory usage of the cluster nodes and pods:

```bash
kubectl top nodes
kubectl top pods
```

Example output:

```bash
NAME       CPU(cores)   MEMORY(bytes)
minikube   100m          500Mi
```

---

### **Step 6: Cluster Maintenance**

#### **6.1 Draining a Node (Simulating Node Maintenance)**

While Minikube typically runs on a single node, you can simulate draining the node for maintenance:

```bash
kubectl drain minikube --ignore-daemonsets
```

This will evict the pods but leave the DaemonSets running.

#### **6.2 Uncordon a Node**

Once maintenance is complete, uncordon the node to allow scheduling new pods:

```bash
kubectl uncordon minikube
```

---

### **Step 7: Cluster Cleanup**

To clean up unused resources (e.g., deleted deployments, pods, PVCs):

#### **7.1 Delete the Nginx Deployment**

```bash
kubectl delete deployment nginx
```

#### **7.2 Delete All Resources**

If you want to delete all resources in the current namespace:

```bash
kubectl delete all --all
```

---

### **Step 8: Testing Cluster with Load Generation**

To generate load on your cluster and test scalability and load balancing, we can use a simple **load generator**. This can be done with a **wrk** (a modern HTTP benchmarking tool).

1. **Install wrk** on your Minikube VM:
   ```bash
   sudo apt-get install wrk
   ```

2. **Run wrk to generate load** on the nginx service:
   ```bash
   wrk -t12 -c400 -d30s http://<nginx-service-ip>:80
   ```

   This command runs for 30 seconds, using 12 threads and 400 connections.

3. Monitor the performance of the cluster using the `kubectl top` commands and check how the system responds to the load.

---

### **Conclusion**

In this exercise, you have learned how to:

1. Set up and start a Kubernetes cluster using **Minikube**.
2. Scale applications within the cluster.
3. Perform a Kubernetes upgrade in a local Minikube environment.
4. Monitor the health and resource usage of the cluster.
5. Simulate node maintenance with the `drain` and `uncordon` commands.
6. Clean up resources in the cluster.

