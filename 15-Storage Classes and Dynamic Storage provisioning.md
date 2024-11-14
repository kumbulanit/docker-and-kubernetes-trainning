### Practical Exercise: Storage Classes and Dynamic Storage Provisioning in Kubernetes

In Kubernetes, **StorageClasses** define different types of storage that are available in the cluster, such as SSD, HDD, or network-attached storage. When you create a **PersistentVolumeClaim (PVC)**, Kubernetes can use a **StorageClass** to dynamically provision the appropriate PersistentVolume (PV) for you.

In this exercise, we will:

1. Create a **StorageClass** that defines a dynamic provisioning strategy.
2. Create a **PersistentVolumeClaim (PVC)** to request dynamic storage.
3. Verify that a **PersistentVolume (PV)** is automatically provisioned based on the StorageClass.

### **Prerequisites:**

Ensure that you have a Kubernetes cluster running with a provisioner that supports dynamic storage provisioning. If you're using a cloud provider like AWS, GCP, or Azure, they provide built-in storage classes. For this example, we'll assume the cluster has access to a dynamic storage provisioner like `standard` (used in many cloud providers).

---

### **Step 1: Create a StorageClass**

A **StorageClass** allows you to define the storage characteristics that Kubernetes should use to dynamically provision PersistentVolumes. For this example, we will create a basic `StorageClass` named `fast-storage` that uses a generic storage provisioner.

1. **Create a StorageClass definition file (`storage-class.yaml`)**:

```yaml
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  name: fast-storage
provisioner: kubernetes.io/aws-ebs
parameters:
  type: gp2
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
```

- `provisioner`: Defines the provisioner to use for dynamic provisioning (in this case, AWS EBS).
- `parameters`: Specifies the type of volume. `gp2` is a type of SSD-backed volume in AWS.
- `reclaimPolicy`: Determines what happens to the PV when the PVC is deleted. `Retain` means the volume will not be deleted when the PVC is deleted.
- `volumeBindingMode`: Ensures that the volume is not provisioned until it is actually needed by a pod (WaitForFirstConsumer).

Apply the `StorageClass` definition:

```bash
kubectl apply -f storage-class.yaml
```

---

### **Step 2: Create a PersistentVolumeClaim (PVC)**

Next, we will create a **PersistentVolumeClaim (PVC)** that requests storage from the `fast-storage` StorageClass.

1. **Create a PVC definition file (`pvc-example.yaml`)**:

```yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: fast-storage-claim
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 5Gi
  storageClassName: fast-storage
```

- `accessModes`: Defines the access modes for the volume. `ReadWriteOnce` means the volume can be mounted by a single node.
- `resources.requests.storage`: Specifies the amount of storage requested (5Gi).
- `storageClassName`: Specifies the `StorageClass` to use, in this case, `fast-storage`.

Apply the PVC definition:

```bash
kubectl apply -f pvc-example.yaml
```

---

### **Step 3: Verify Dynamic Provisioning**

After applying the PVC, Kubernetes will use the `fast-storage` StorageClass to dynamically provision a **PersistentVolume (PV)**.

1. **Check the status of the PVC**:

```bash
kubectl get pvc fast-storage-claim
```

You should see that the PVC is in the `Bound` state, indicating that a PersistentVolume has been created and bound to the PVC.

2. **Verify the dynamically provisioned PersistentVolume**:

```bash
kubectl get pv
```

Look for a `PersistentVolume` that matches the PVC `fast-storage-claim`. The `STATUS` should be `Bound`, and the `STORAGECLASS` should be `fast-storage`.

---

### **Step 4: Use the PersistentVolume in a Pod**

Now that we have a dynamically provisioned volume, let's create a pod that uses the PersistentVolumeClaim to mount the storage.

1. **Create a pod definition file (`pod-with-pvc.yaml`)**:

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-with-pvc
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: nginx-storage
  volumes:
    - name: nginx-storage
      persistentVolumeClaim:
        claimName: fast-storage-claim
```

- `volumeMounts`: Specifies where the volume should be mounted in the container (in this case, `/usr/share/nginx/html` for the Nginx container).
- `volumes`: Refers to the PersistentVolumeClaim (`fast-storage-claim`) to mount the volume.

Apply the pod definition:

```bash
kubectl apply -f pod-with-pvc.yaml
```

---

### **Step 5: Verify the Pod and Volume Mount**

1. **Check the pod status**:

```bash
kubectl get pods nginx-with-pvc
```

Ensure the pod is running and the volume has been mounted.

2. **Verify the mount inside the container**:

Once the pod is running, you can exec into the container to check if the volume is mounted correctly:

```bash
kubectl exec -it nginx-with-pvc -- /bin/bash
```

Inside the pod, you can check if the volume is mounted at `/usr/share/nginx/html`:

```bash
ls /usr/share/nginx/html
```

You should see an empty directory. Now, you can copy files to this directory and check if the changes persist across pod restarts.

---

### **Step 6: Clean Up Resources**

Once you are done testing, clean up the resources:

1. Delete the pod:

```bash
kubectl delete pod nginx-with-pvc
```

2. Delete the PVC:

```bash
kubectl delete pvc fast-storage-claim
```

3. Delete the StorageClass (optional):

```bash
kubectl delete storageclass fast-storage
```

---

### **Conclusion**

In this exercise, we:

1. Created a **StorageClass** that defines the dynamic provisioning strategy.
2. Created a **PersistentVolumeClaim (PVC)** that requests storage from the defined `StorageClass`.
3. Verified that Kubernetes dynamically provisioned a **PersistentVolume (PV)** based on the PVC.
4. Created a pod that used the dynamically provisioned volume.
5. Tested persistent storage by verifying the mount inside the pod.

