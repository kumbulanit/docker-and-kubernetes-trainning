### Practical Working Example: Cluster Monitoring in Kubernetes

Monitoring is an essential part of managing a Kubernetes cluster. It helps in tracking the health of the cluster, identifying issues, and ensuring smooth operations. For this exercise, we will set up a **Kubernetes cluster monitoring solution** using **Prometheus** and **Grafana**, two of the most widely used tools for Kubernetes cluster monitoring.

In this example, we will deploy **Prometheus** for gathering metrics and **Grafana** for visualizing these metrics in a dashboard.

### **Prerequisites:**

- A running Kubernetes cluster (Minikube or any other).
- `kubectl` command-line tool configured.
- Basic understanding of Kubernetes resources and monitoring concepts.

### **Steps to Implement Cluster Monitoring with Prometheus and Grafana:**

#### **Step 1: Install Prometheus and Grafana using Kubernetes Manifests**

1. **Create a Namespace for Monitoring**:

   It’s a good practice to isolate monitoring resources in a separate namespace.

   ```bash
   kubectl create namespace monitoring
   ```

2. **Deploy Prometheus**:

   We will deploy **Prometheus** to scrape metrics from the Kubernetes nodes and pods.

   First, create a file called `prometheus-deployment.yaml` with the following content:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: prometheus
     namespace: monitoring
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: prometheus
     template:
       metadata:
         labels:
           app: prometheus
       spec:
         containers:
           - name: prometheus
             image: prom/prometheus:v2.42.0
             ports:
               - containerPort: 9090
             volumeMounts:
               - name: prometheus-storage
                 mountPath: /prometheus
         volumes:
           - name: prometheus-storage
             emptyDir: {}
   ```

   Apply the deployment to Kubernetes:

   ```bash
   kubectl apply -f prometheus-deployment.yaml
   ```

3. **Create a Service to Access Prometheus**:

   Create a service to expose Prometheus to the outside world. Create a file called `prometheus-service.yaml`:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: prometheus
     namespace: monitoring
   spec:
     ports:
       - port: 9090
     selector:
       app: prometheus
   ```

   Apply the service:

   ```bash
   kubectl apply -f prometheus-service.yaml
   ```

4. **Verify Prometheus Deployment**:

   Check that Prometheus is running:

   ```bash
   kubectl get pods -n monitoring
   ```

   You should see a pod running with the name `prometheus`.

   Get the Prometheus service:

   ```bash
   kubectl get svc -n monitoring
   ```

   - **Expected Output**: You should see the `prometheus` service running on port 9090.

#### **Step 2: Deploy Grafana**

Next, we will deploy **Grafana** to visualize the metrics collected by Prometheus.

1. **Create a Deployment for Grafana**:

   Create a file called `grafana-deployment.yaml` with the following content:

   ```yaml
   apiVersion: apps/v1
   kind: Deployment
   metadata:
     name: grafana
     namespace: monitoring
   spec:
     replicas: 1
     selector:
       matchLabels:
         app: grafana
     template:
       metadata:
         labels:
           app: grafana
       spec:
         containers:
           - name: grafana
             image: grafana/grafana:8.3.0
             ports:
               - containerPort: 3000
             env:
               - name: GF_SECURITY_ADMIN_PASSWORD
                 value: "admin"  # Set a password for Grafana
   ```

   Apply the Grafana deployment:

   ```bash
   kubectl apply -f grafana-deployment.yaml
   ```

2. **Create a Service for Grafana**:

   To expose Grafana, create a file called `grafana-service.yaml`:

   ```yaml
   apiVersion: v1
   kind: Service
   metadata:
     name: grafana
     namespace: monitoring
   spec:
     ports:
       - port: 3000
     selector:
       app: grafana
   ```

   Apply the service:

   ```bash
   kubectl apply -f grafana-service.yaml
   ```

3. **Verify Grafana Deployment**:

   Verify that Grafana is running:

   ```bash
   kubectl get pods -n monitoring
   ```

   Get the Grafana service:

   ```bash
   kubectl get svc -n monitoring
   ```

   - **Expected Output**: You should see the `grafana` service running on port 3000.

#### **Step 3: Connect Grafana to Prometheus**

1. **Access Grafana Dashboard**:

   Get the IP address of your Minikube instance or Kubernetes cluster:

   ```bash
   minikube service grafana -n monitoring --url
   ```

   This command will return the URL to access the Grafana dashboard.

2. **Login to Grafana**:

   Open the URL in your browser. The default username is `admin`, and the password is also `admin` (as we set earlier).

3. **Configure Prometheus as the Data Source**:

   - Go to **Configuration** (the gear icon) in the left sidebar.
   - Click on **Data Sources**.
   - Click on **Add data source**.
   - Select **Prometheus**.
   - In the **URL** field, enter `http://prometheus:9090`.
   - Click **Save & Test** to verify the connection.

#### **Step 4: Create a Grafana Dashboard**

1. **Create a new Dashboard**:

   - From the left sidebar, click on the **+** icon and select **Dashboard**.
   - Click on **Add new panel**.
   - In the query editor, select **Prometheus** as the data source.
   - Use the following query to display the CPU usage of your nodes:

   ```text
   rate(container_cpu_usage_seconds_total{job="kubelet", cluster="", container!="POD", container!="prometheus"}[5m])
   ```

2. **Visualize Cluster Metrics**:

   - You can create multiple panels for different metrics like CPU, memory, network usage, etc.
   - Use queries to fetch node metrics, pod metrics, and cluster health information.

3. **Save the Dashboard**:

   Once you are satisfied with the panels, click **Save dashboard** and provide a name for your dashboard.

#### **Step 5: Verify Cluster Monitoring**

1. **Check Prometheus Metrics**:

   To verify that Prometheus is scraping metrics, you can visit the Prometheus UI:

   ```bash
   kubectl port-forward svc/prometheus -n monitoring 9090:9090
   ```

   Visit `http://localhost:9090` in your browser. You can query for metrics like:

   ```text
   container_cpu_usage_seconds_total
   ```

2. **Check Grafana Dashboards**:

   You can also check the Grafana dashboard you created for real-time monitoring. The graphs should update with the metrics collected from the Prometheus server.

---

### **Step 6: Clean up Resources**

Once you’re done testing, you can clean up the resources created for monitoring.

1. **Delete the deployments**:

   ```bash
   kubectl delete -f prometheus-deployment.yaml
   kubectl delete -f grafana-deployment.yaml
   ```

2. **Delete the services**:

   ```bash
   kubectl delete -f prometheus-service.yaml
   kubectl delete -f grafana-service.yaml
   ```

3. **Delete the monitoring namespace**:

   ```bash
   kubectl delete namespace monitoring
   ```

---

### **Conclusion**

In this exercise, you:
1. Set up **Prometheus** for monitoring Kubernetes cluster metrics.
2. Deployed **Grafana** for visualizing Prometheus data in real-time dashboards.
3. Verified the deployment by accessing the Prometheus and Grafana dashboards.
4. Cleaned up the resources to free up your Kubernetes cluster.
