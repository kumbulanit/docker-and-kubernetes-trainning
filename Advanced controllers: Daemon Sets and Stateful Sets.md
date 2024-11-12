### Practical Exercise: Advanced Controllers - DaemonSets and StatefulSets in Kubernetes

In this practical exercise, we will explore two advanced controllers in Kubernetes:

1. **DaemonSet**: Ensures that a copy of a pod is running on every node (or selected nodes) in the cluster.
2. **StatefulSet**: Manages stateful applications, ensuring that each pod has a unique identity and persistent storage.

We will walk through how to deploy both of these controllers and how they are useful for specific use cases, including managing logging agents and stateful applications such as databases.

---

### **1. DaemonSet Example: Deploying Fluentd as a Log Forwarder**

A `DaemonSet` ensures that a pod is running on each node. We will use Fluentd, a popular log forwarder, to collect logs from every node in the cluster and forward them to a logging system.

#### **Step 1: Create a DaemonSet for Fluentd**

1. **Create a DaemonSet YAML file (`fluentd-daemonset.yaml`):**

```yaml
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: fluentd
spec:
  selector:
    matchLabels:
      app: fluentd
  template:
    metadata:
      labels:
        app: fluentd
    spec:
      containers:
        - name: fluentd
          image: fluent/fluentd:v1.12-1
          ports:
            - containerPort: 24224
          volumeMounts:
            - name: fluentd-config
              mountPath: /fluentd/etc
      volumes:
        - name: fluentd-config
          configMap:
            name: fluentd-config
```

2. **Create the ConfigMap for Fluentd configuration:**

Before applying the DaemonSet, we need a ConfigMap for Fluentd configuration. Create a file named `fluentd-configmap.yaml`:

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: fluentd-config
data:
  fluentd.conf: |
    <source>
      @type tail
      tag kubernetes.*
      path /var/log/containers/*.log
      pos_file /var/log/fluentd-containers.log.pos
      format json
    </source>

    <match kubernetes.**>
      @type stdout
    </match>
```

3. **Apply the ConfigMap and DaemonSet:**

```bash
kubectl apply -f fluentd-configmap.yaml
kubectl apply -f fluentd-daemonset.yaml
```

4. **Verify the DaemonSet:**

After applying the DaemonSet, check the status of the pods:

```bash
kubectl get daemonsets
kubectl get pods -l app=fluentd
```

Fluentd will now run on each node in the cluster and collect logs from all the containers.

---

### **2. StatefulSet Example: Deploying Redis with Persistent Storage**

A `StatefulSet` is used to manage stateful applications, ensuring that each pod has a unique identity and stable storage. We will deploy **Redis** using a `StatefulSet` to demonstrate this.

#### **Step 1: Create a StatefulSet for Redis**

1. **Create a StatefulSet YAML file (`redis-statefulset.yaml`):**

```yaml
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: redis
spec:
  serviceName: "redis"
  replicas: 3
  selector:
    matchLabels:
      app: redis
  template:
    metadata:
      labels:
        app: redis
    spec:
      containers:
        - name: redis
          image: redis:latest
          ports:
            - containerPort: 6379
          volumeMounts:
            - name: redis-data
              mountPath: /data
  volumeClaimTemplates:
    - metadata:
        name: redis-data
      spec:
        accessModes: ["ReadWriteOnce"]
        resources:
          requests:
            storage: 1Gi
```

2. **Apply the StatefulSet:**

```bash
kubectl apply -f redis-statefulset.yaml
```

3. **Verify the StatefulSet:**

```bash
kubectl get statefulsets
kubectl get pods -l app=redis
```

4. **Check Persistent Volumes:**

The StatefulSet will create Persistent Volume Claims (PVCs) for each Redis pod (e.g., `redis-data-0`, `redis-data-1`, `redis-data-2`). You can verify the Persistent Volumes (PVs) as follows:

```bash
kubectl get pvc
```

5. **Access Redis Pods:**

You can access individual Redis pods using their unique names, like `redis-0`, `redis-1`, etc.

To test the Redis instance, you can exec into one of the Redis pods:

```bash
kubectl exec -it redis-0 -- redis-cli
```

Now, you have a Redis instance running with persistent storage that can be accessed using its stable DNS name (`redis-0.redis`), ensuring the data persists even if a pod is rescheduled.

---

### **3. Exposing Services for Both DaemonSet and StatefulSet**

You can expose the `DaemonSet` and `StatefulSet` applications using Kubernetes services.

#### **3.1. Expose the Fluentd DaemonSet as a Service**

Since Fluentd runs on every node, you might not need to expose it externally. However, you can expose it internally as a ClusterIP service:

```bash
kubectl expose daemonset fluentd --name=fluentd-service --port=24224 --target-port=24224
```

Verify the service:

```bash
kubectl get svc fluentd-service
```

#### **3.2. Expose the Redis StatefulSet as a Service**

For Redis, you can expose it using a **Headless Service** (no load balancing) so that each Redis pod can be accessed by its stable DNS name:

1. **Create a Headless Service YAML file (`redis-service.yaml`):**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: redis
spec:
  clusterIP: None
  selector:
    app: redis
  ports:
    - port: 6379
      targetPort: 6379
```

2. **Apply the Service:**

```bash
kubectl apply -f redis-service.yaml
```

Now, you can access each Redis pod using the following DNS names: `redis-0.redis`, `redis-1.redis`, etc.

---

### **4. Scaling and Managing Pods**

Both `DaemonSet` and `StatefulSet` support scaling, though the use cases differ:

- **Scaling a DaemonSet**: By default, a DaemonSet runs one pod per node. However, you can adjust the number of nodes in your cluster to scale the number of Fluentd pods.

- **Scaling a StatefulSet**: A `StatefulSet` allows you to scale the number of replicas (Redis instances) up or down. To scale Redis:

```bash
kubectl scale statefulset redis --replicas=5
```

Verify the scaling:

```bash
kubectl get statefulsets
kubectl get pods -l app=redis
```

---

### **Conclusion**

In this exercise, we explored how to deploy advanced controllers like **DaemonSet** and **StatefulSet** in Kubernetes.

- **DaemonSet**: We used Fluentd as an example to deploy a log forwarder on every node in the cluster. This ensures that logs are collected from all nodes, which is a common use case for system monitoring tools.
  
- **StatefulSet**: We deployed Redis as an example of a stateful application that requires persistent storage and stable, unique identities for each pod. StatefulSets are particularly useful for databases and other stateful services.

