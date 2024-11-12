### Practical Example: Persistence of Application State and the Data Volume Model in Kubernetes

In this exercise, we'll demonstrate how to manage persistent application state in Kubernetes using **Persistent Volumes (PVs)** and **Persistent Volume Claims (PVCs)**. We'll deploy a simple application that requires persistent storage to store data, such as a **PostgreSQL database**, and ensure that data persists even if the pod is deleted or restarted.

---

### **Step 1: Set Up the Persistent Volume (PV)**

A Persistent Volume (PV) is a piece of storage in the cluster that has been provisioned by an administrator or dynamically provisioned using a storage class.

For this example, we'll use a **hostPath** volume, which stores data on the node's filesystem. While this is typically not suitable for production, it's good for testing.

#### **1.1 Create the Persistent Volume**

Create a file named `postgres-pv.yaml` to define the Persistent Volume:

```yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: postgres-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data/postgres
```

Explanation:
- `hostPath` specifies where the data will be stored on the node.
- `storage` is the size of the volume (1Gi in this case).
- `accessModes: ReadWriteOnce` means only one pod can mount the volume at a time.
- `persistentVolumeReclaimPolicy: Retain` means the volume will not be deleted when the PVC is deleted.

Apply the Persistent Volume:

```bash
kubectl apply -f postgres-pv.yaml
```

Verify the Persistent Volume:

```bash
kubectl get pv
```

You should see the `postgres-pv` listed.

---

### **Step 2: Create the Persistent Volume Claim (PVC)**

A Persistent Volume Claim (PVC) is a request for storage by a user. It allows a pod to request specific storage resources, and Kubernetes will bind the PVC to an available PV.

#### **2.1 Create the Persistent Volume Claim**

Create a file named `postgres-pvc.yaml` to define the Persistent Volume Claim:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: postgres-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

Explanation:
- The `accessModes` and `resources.requests.storage` match the PV specifications.
- The `storageClassName` should be the same as the PV’s to bind the PVC to the PV.

Apply the Persistent Volume Claim:

```bash
kubectl apply -f postgres-pvc.yaml
```

Verify the Persistent Volume Claim:

```bash
kubectl get pvc
```

You should see the `postgres-pvc` with the status `Bound`.

---

### **Step 3: Deploy a PostgreSQL Application**

Now that we have the persistent volume claim (PVC) created, we will deploy a PostgreSQL database that uses this PVC to store data.

#### **3.1 Create the PostgreSQL Deployment**

Create a file named `postgres-deployment.yaml` to define the PostgreSQL deployment:

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: postgres-deployment
spec:
  replicas: 1
  selector:
    matchLabels:
      app: postgres
  template:
    metadata:
      labels:
        app: postgres
    spec:
      containers:
        - name: postgres
          image: postgres:13
          env:
            - name: POSTGRES_PASSWORD
              value: "mysecretpassword"
          ports:
            - containerPort: 5432
          volumeMounts:
            - mountPath: /var/lib/postgresql/data
              name: postgres-storage
      volumes:
        - name: postgres-storage
          persistentVolumeClaim:
            claimName: postgres-pvc
```

Explanation:
- The PostgreSQL container is configured to use the `postgres-pvc` Persistent Volume Claim.
- The volume is mounted to `/var/lib/postgresql/data`, which is the default data directory for PostgreSQL.

Apply the PostgreSQL deployment:

```bash
kubectl apply -f postgres-deployment.yaml
```

Verify the PostgreSQL deployment:

```bash
kubectl get deployments
```

You should see the `postgres-deployment` running.

---

### **Step 4: Expose the PostgreSQL Service**

Now that PostgreSQL is running, we need to expose it so that other services or applications can connect to it.

#### **4.1 Create the PostgreSQL Service**

Create a file named `postgres-service.yaml` to expose the PostgreSQL service:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: postgres-service
spec:
  selector:
    app: postgres
  ports:
    - protocol: TCP
      port: 5432
      targetPort: 5432
  type: ClusterIP
```

Apply the PostgreSQL service:

```bash
kubectl apply -f postgres-service.yaml
```

Verify the service:

```bash
kubectl get svc
```

You should see the `postgres-service` listed, and it should be assigned an internal IP.

---

### **Step 5: Testing Persistence of Data**

To test the persistence of data, we will interact with the PostgreSQL database and then delete the pod to see if the data persists.

#### **5.1 Connect to PostgreSQL**

Get the name of the PostgreSQL pod:

```bash
kubectl get pods
```

Then, exec into the PostgreSQL pod:

```bash
kubectl exec -it <postgres-pod-name> -- bash
```

Inside the pod, access the PostgreSQL database:

```bash
psql -U postgres
```

Create a test database and insert some data:

```sql
CREATE DATABASE testdb;
\c testdb;
CREATE TABLE test_table (id SERIAL PRIMARY KEY, name VARCHAR(50));
INSERT INTO test_table (name) VALUES ('Test Name');
```

Verify that the data is inserted:

```sql
SELECT * FROM test_table;
```

#### **5.2 Delete the PostgreSQL Pod**

Now, delete the PostgreSQL pod:

```bash
kubectl delete pod <postgres-pod-name>
```

The pod will be recreated because it's managed by a deployment.

#### **5.3 Verify Data Persistence**

Once the pod is back up, exec into the new pod:

```bash
kubectl get pods
kubectl exec -it <new-postgres-pod-name> -- bash
```

Access the PostgreSQL database again:

```bash
psql -U postgres
```

Switch to the `testdb` database and check if the data persists:

```sql
\c testdb;
SELECT * FROM test_table;
```

You should see that the data (`Test Name`) persists even after the pod was deleted and recreated.

---

### **Step 6: Clean Up Resources**

Once you're done with the exercise, clean up the resources created.

```bash
kubectl delete -f postgres-deployment.yaml
kubectl delete -f postgres-service.yaml
kubectl delete -f postgres-pvc.yaml
kubectl delete -f postgres-pv.yaml
```

---

### **Summary**

In this exercise:
- We created a **Persistent Volume (PV)** and **Persistent Volume Claim (PVC)** in Kubernetes to persist data.
- We deployed a **PostgreSQL database** and mounted the PVC as a volume to store the database data.
- We tested the persistence of the data by deleting and recreating the PostgreSQL pod.
- The data persisted even after the pod was deleted, demonstrating the power of Kubernetes’ persistent storage model.

T