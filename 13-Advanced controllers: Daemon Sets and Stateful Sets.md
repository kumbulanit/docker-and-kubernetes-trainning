### Practical Exercise: Advanced Controllers - DaemonSets and StatefulSets in Kubernetes

In this practical exercise, we will explore two advanced controllers in Kubernetes:

1. **DaemonSet**: Ensures that a copy of a pod is running on every node (or selected nodes) in the cluster.
2. **StatefulSet**: Manages stateful applications, ensuring that each pod has a unique identity and persistent storage.

We will walk through how to deploy both of these controllers and how they are useful for specific use cases, including managing logging agents and stateful applications such as databases.

---

### **1. DaemonSet Example: Deploying filebeat**

A `DaemonSet` ensures that a pod is running on each node. We will use filebeat, a popular log forwarder, to collect logs from every node in the cluster and forward them to a logging system.

#### **Step 1: Create a DaemonSet for Filebeat, it also includes namespace,configmap in one file **

1. **Create a DaemonSet YAML file (`filebeat-daemonset.yaml`):**

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: logging
```

create config map 

```yaml

apiVersion: v1
kind: ConfigMap
metadata:
  name: filebeat-config
  namespace: logging
  labels:
    app: filebeat
data:
  filebeat.yml: |-
    filebeat.inputs:
    - type: container
      paths:
        - /var/log/containers/*.log
      processors:
        - add_kubernetes_metadata:
            in_cluster: true

    output.elasticsearch:
      hosts: ["http://elasticsearch:9200"]
      username: "elastic"
      password: "changeme"

    setup.template.enabled: false
    setup.ilm.enabled: false
```
create a deamonset deamonset-filebeat.yaml
```yaml

apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: filebeat
  namespace: logging
  labels:
    app: filebeat
spec:
  selector:
    matchLabels:
      app: filebeat
  template:
    metadata:
      labels:
        app: filebeat
    spec:
      serviceAccountName: filebeat
      containers:
      - name: filebeat
        image: docker.elastic.co/beats/filebeat:8.10.0
        args: [
          "-c", "/etc/filebeat.yml",
          "-e",
        ]
        env:
          - name: ELASTICSEARCH_HOST
            value: "elasticsearch"
          - name: ELASTICSEARCH_PORT
            value: "9200"
        securityContext:
          runAsUser: 0
        resources:
          limits:
            memory: 200Mi
            cpu: 100m
          requests:
            memory: 100Mi
            cpu: 100m
        volumeMounts:
        - name: config-volume
          mountPath: /etc/filebeat.yml
          subPath: filebeat.yml
        - name: varlog
          mountPath: /var/log
        - name: varlibdockercontainers
          mountPath: /var/lib/docker/containers
          readOnly: true
      volumes:
      - name: config-volume
        configMap:
          name: filebeat-config
      - name: varlog
        hostPath:
          path: /var/log
      - name: varlibdockercontainers
        hostPath:
          path: /var/lib/docker/containers
```
create a service account service-account.yaml

```yaml

apiVersion: v1
kind: ServiceAccount
metadata:
  name: filebeat
  namespace: logging
  labels:
    app: filebeat

```

3. **Apply the ConfigMap and DaemonSet:**

```bash
kubectl apply -f filebeat-daemonset.yaml
```

4. **Verify the DaemonSet:**

After applying the DaemonSet, check the status of the pods:

```bash
kubectl get daemonsets -n logging
kubectl get pods -l app=filebeat
```

Filebeat will now run on each node in the cluster and collect logs from all the containers.

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

- **Scaling a DaemonSet**: By default, a DaemonSet runs one pod per node. However, you can adjust the number of nodes in your cluster to scale the number of filebeat pods.

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

- **DaemonSet**: We used filebeat as an example to deploy a log forwarder on every node in the cluster. This ensures that logs are collected from all nodes, which is a common use case for system monitoring tools.
  
- **StatefulSet**: We deployed Redis as an example of a stateful application that requires persistent storage and stable, unique identities for each pod. StatefulSets are particularly useful for databases and other stateful services.

