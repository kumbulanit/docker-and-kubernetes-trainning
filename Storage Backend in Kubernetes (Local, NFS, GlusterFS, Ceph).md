### Practical Example: Storage Backend in Kubernetes (Local, NFS, GlusterFS, Ceph) on a Single Ubuntu Server Instance

In this exercise, we will demonstrate how to configure multiple storage backends in Kubernetes on a single Ubuntu server. We will cover the following storage backends:

1. **Local storage** (using hostPath)
2. **NFS** (Network File System)
3. **GlusterFS** (a scalable network filesystem)
4. **Ceph** (distributed storage system)

We will go through the steps to set up each of these backends, assuming you are working with a single Ubuntu server instance running Kubernetes.

### Prerequisites

- A single Ubuntu server with **Kubernetes** installed (e.g., using **minikube** or **kubeadm**).
- `kubectl` command line tool configured to access the Kubernetes cluster.
- **Root access** on the Ubuntu server.

---

### **Step 1: Setup Local Storage (hostPath)**

The **hostPath** volume uses local storage on the Kubernetes node itself. It’s suitable for testing but not for production environments.

#### **1.1 Create a Local Storage Volume (hostPath)**

First, ensure there is a directory on your local filesystem that Kubernetes can mount. For example, `/mnt/data` on your server.

Create a Persistent Volume (PV) using **hostPath**:

```yaml
# local-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: local-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  storageClassName: manual
  hostPath:
    path: /mnt/data
```

Apply this file to create the volume:

```bash
kubectl apply -f local-pv.yaml
```

#### **1.2 Create a Persistent Volume Claim (PVC)**

Create a PVC that requests the local storage:

```yaml
# local-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: local-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: manual
```

Apply the PVC:

```bash
kubectl apply -f local-pvc.yaml
```

Now, you can create a pod that uses the PVC.

---

### **Step 2: Setup NFS (Network File System)**

**NFS** is a distributed file system protocol that allows a system to share directories and files with others over a network.

#### **2.1 Install NFS on the Ubuntu Server**

Install the NFS server:

```bash
sudo apt update
sudo apt install nfs-kernel-server
```

Create the directory to be shared:

```bash
sudo mkdir -p /mnt/nfs_share
sudo chmod 777 /mnt/nfs_share
```

Export the directory (make it available over the network):

```bash
echo "/mnt/nfs_share *(rw,sync,no_subtree_check)" | sudo tee -a /etc/exports
```

Apply the export:

```bash
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

#### **2.2 Configure Kubernetes to Use NFS**

Create a Persistent Volume (PV) for NFS:

```yaml
# nfs-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: nfs-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: nfs-storage
  nfs:
    path: /mnt/nfs_share
    server: <NFS_SERVER_IP>
```

Replace `<NFS_SERVER_IP>` with the IP address of your NFS server (the current Ubuntu machine).

Apply the NFS PV:

```bash
kubectl apply -f nfs-pv.yaml
```

#### **2.3 Create a PVC for NFS**

```yaml
# nfs-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: nfs-storage
```

Apply the PVC:

```bash
kubectl apply -f nfs-pvc.yaml
```

Now you can create a pod that mounts the NFS storage.

---

### **Step 3: Setup GlusterFS**

**GlusterFS** is a scalable network filesystem suitable for handling large amounts of data across multiple servers. For this example, we will configure it on a single server to simulate the GlusterFS setup.

#### **3.1 Install GlusterFS on Ubuntu**

Install the necessary GlusterFS packages:

```bash
sudo apt update
sudo apt install glusterfs-server
```

Start the GlusterFS service:

```bash
sudo systemctl start glusterd
sudo systemctl enable glusterd
```

#### **3.2 Create a GlusterFS Volume**

Create a directory to store the GlusterFS data:

```bash
sudo mkdir -p /mnt/glusterfs
sudo chmod 777 /mnt/glusterfs
```

Create a GlusterFS volume:

```bash
sudo gluster volume create gv0 transport tcp localhost:/mnt/glusterfs
sudo gluster volume start gv0
```

#### **3.3 Configure Kubernetes to Use GlusterFS**

Create a Persistent Volume (PV) for GlusterFS:

```yaml
# glusterfs-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: glusterfs-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteMany
  persistentVolumeReclaimPolicy: Retain
  storageClassName: glusterfs-storage
  glusterfs:
    endpoints: glusterfs-cluster
    path: gv0
    readOnly: false
```

#### **3.4 Create the PVC for GlusterFS**

```yaml
# glusterfs-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: glusterfs-pvc
spec:
  accessModes:
    - ReadWriteMany
  resources:
    requests:
      storage: 1Gi
  storageClassName: glusterfs-storage
```

Apply the GlusterFS PV and PVC:

```bash
kubectl apply -f glusterfs-pv.yaml
kubectl apply -f glusterfs-pvc.yaml
```

---

### **Step 4: Setup Ceph Storage**

**Ceph** is a distributed storage system that provides highly scalable object, block, and file-based storage.

#### **4.1 Install Ceph on Ubuntu**

Install the Ceph packages:

```bash
sudo apt update
sudo apt install ceph ceph-deploy
```

Initialize the Ceph cluster (for this setup, we will use only one node):

```bash
ceph-deploy new <hostname>
ceph-deploy install <hostname>
ceph-deploy mon create-initial
```

Verify the Ceph status:

```bash
ceph -s
```

#### **4.2 Configure Kubernetes to Use Ceph**

Create a Ceph StorageClass and PV configuration:

```yaml
# ceph-pv.yaml
apiVersion: v1
kind: PersistentVolume
metadata:
  name: ceph-pv
spec:
  capacity:
    storage: 1Gi
  volumeMode: Filesystem
  accessModes:
    - ReadWriteOnce
  persistentVolumeReclaimPolicy: Retain
  cephfs:
    monitors:
      - <ceph_monitor_ip>
    user: admin
    secretRef:
      name: ceph-secret
    path: /
```

Replace `<ceph_monitor_ip>` with the IP address of your Ceph monitor.

Create the PVC for Ceph:

```yaml
# ceph-pvc.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: ceph-pvc
spec:
  accessModes:
    - ReadWriteOnce
  resources:
    requests:
      storage: 1Gi
  storageClassName: ceph-storage
```

Apply the Ceph PV and PVC:

```bash
kubectl apply -f ceph-pv.yaml
kubectl apply -f ceph-pvc.yaml
```

---

### **Step 5: Create Pods that Use These Storage Backends**

Finally, create pods that will mount the persistent volumes you configured.

For example, you can use a simple Nginx pod that mounts each of these volumes.

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: nginx-pod
spec:
  containers:
    - name: nginx
      image: nginx
      volumeMounts:
        - mountPath: /usr/share/nginx/html
          name: storage-volume
  volumes:
    - name: storage-volume
      persistentVolumeClaim:
        claimName: <pvc-name>  # Replace with the PVC name for the chosen storage backend
```

Apply the pod configuration for each storage backend and verify that the pod can access the persistent volume.

---

### **Summary**

This guide demonstrated how to set up different storage backends in Kubernetes on a single Ubuntu server, including:

1. **Local storage** using `hostPath`.
2. **NFS** by configuring an NFS server and creating PVs and PVCs

 in Kubernetes.
3. **GlusterFS** by configuring a GlusterFS volume and integrating it with Kubernetes.
4. **Ceph** by setting up a Ceph cluster and configuring Kubernetes to use Ceph for storage.

