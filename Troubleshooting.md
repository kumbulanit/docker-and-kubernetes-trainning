### Practical Working Example: Troubleshooting in Kubernetes

Kubernetes troubleshooting involves identifying and resolving issues that arise in your cluster, whether they're related to pod failures, networking, resource usage, or misconfigurations. In this example, we'll go through the steps of troubleshooting common issues in a Kubernetes cluster and how to resolve them.

#### **Prerequisites:**

- A running Kubernetes cluster (Minikube or other).
- `kubectl` installed and configured.
- Basic understanding of Kubernetes objects like Pods, Deployments, and Services.

### **Troubleshooting Common Kubernetes Issues**

#### **Step 1: Troubleshooting Pod Failures**

1. **Scenario: Pod CrashLoopBackOff**

   A pod can enter a **CrashLoopBackOff** state if the container fails to start repeatedly. To investigate this:

   1. **Check the Pod Status**:

      Run the following command to check the status of your pods:

      ```bash
      kubectl get pods
      ```

      Look for pods that are in the `CrashLoopBackOff` state.

   2. **Get Pod Logs**:

      To understand why the pod is failing, view the logs:

      ```bash
      kubectl logs <pod_name>
      ```

      Example:

      ```bash
      kubectl logs my-app-pod-12345
      ```

      This will provide error messages or stack traces that could explain the failure.

   3. **Examine Events**:

      If logs are insufficient, check the Kubernetes events for more details:

      ```bash
      kubectl describe pod <pod_name>
      ```

      This will give detailed information about the pod's lifecycle, including any recent errors or warnings that may explain the crash.

   4. **Example of troubleshooting a crash**:
      
      For example, a pod might fail due to a missing environment variable or a misconfigured image. In such cases:
      
      - Double-check your deployment YAML files or Helm chart for correct environment variable definitions.
      - If the image is missing or invalid, verify the image is available on the registry.

#### **Step 2: Troubleshooting Networking Issues**

1. **Scenario: Service is Unreachable**

   When services are not reachable, the issue can be due to incorrect service configuration or network policies.

   1. **Check the Service and Endpoint**:

      First, check if the service exists and has endpoints:

      ```bash
      kubectl get svc
      ```

      Ensure that the service is listed and has endpoints.

   2. **Check Pod Network Connectivity**:

      Verify that the pod can access the service by getting a shell inside the pod:

      ```bash
      kubectl exec -it <pod_name> -- /bin/sh
      ```

      Once inside, test network connectivity to the service using `curl` or `wget`:

      ```bash
      curl http://<service_name>:<port>
      ```

   3. **Check Network Policies**:

      If network policies are in use, verify that they are not blocking traffic. To list network policies:

      ```bash
      kubectl get netpol
      ```

      Review the network policies to ensure that they are not overly restrictive.

#### **Step 3: Troubleshooting Resource Issues**

1. **Scenario: Pods are Being OOMKilled (Out of Memory)**

   Pods can get killed due to resource constraints like memory limits. To diagnose:

   1. **Check Pod Events**:

      Get the description of the pod and look for `OOMKilled`:

      ```bash
      kubectl describe pod <pod_name>
      ```

      In the events section, you may see something like:

      ```
      FailedKillPodError: pod <pod_name> was OOMKilled
      ```

   2. **Check Resource Requests and Limits**:

      Ensure that resource requests and limits are set correctly in your pod configuration. Review the `resources` section in the pod specification:

      ```yaml
      resources:
        requests:
          memory: "64Mi"
        limits:
          memory: "128Mi"
      ```

   3. **Adjust Resource Limits**:

      If the memory limits are too low, adjust them in your YAML file and reapply:

      ```bash
      kubectl apply -f <deployment_yaml>
      ```

#### **Step 4: Troubleshooting Node Issues**

1. **Scenario: Node Not Ready**

   If a node is in the `NotReady` state, it can affect the pod scheduling.

   1. **Check Node Status**:

      Use the following command to check the status of all nodes:

      ```bash
      kubectl get nodes
      ```

      If a node is `NotReady`, describe the node to check the status:

      ```bash
      kubectl describe node <node_name>
      ```

      Look for events that could indicate why the node is not ready, such as issues with the kubelet, disk pressure, or memory pressure.

   2. **Check Kubelet Logs**:

      If the node is not ready, access the node’s kubelet logs for further information. Depending on your setup (Minikube or another platform), access the logs through the systemd journal or directly on the node.

      ```bash
      journalctl -u kubelet
      ```

      This will show logs related to the kubelet, which can reveal why the node is in a `NotReady` state.

#### **Step 5: Troubleshooting Pod Scheduling Issues**

1. **Scenario: Pods are Pending**

   If pods remain in a `Pending` state, it may be due to insufficient resources or taints on nodes.

   1. **Check Pod Status**:

      ```bash
      kubectl get pods
      ```

      Look for pods in the `Pending` state.

   2. **Check Events**:

      Run `kubectl describe pod <pod_name>` to get more details. If there's a resource issue, you'll see an event like:

      ```
      Warning  FailedScheduling  pod <pod_name> has Insufficient cpu/ memory.
      ```

   3. **Check Node Resources**:

      Verify that the node has enough resources (CPU, memory) to schedule the pod. You can check node capacity by running:

      ```bash
      kubectl describe node <node_name>
      ```

      If the node has insufficient resources, you may need to scale your cluster or adjust resource requests.

#### **Step 6: Troubleshooting ConfigMap and Secret Issues**

1. **Scenario: Pod Cannot Access ConfigMap or Secret**

   A pod may fail to access a ConfigMap or Secret due to incorrect references or permissions.

   1. **Check ConfigMap or Secret Existence**:

      First, verify that the ConfigMap or Secret exists:

      ```bash
      kubectl get configmap <configmap_name>
      kubectl get secret <secret_name>
      ```

   2. **Check Pod Configuration**:

      Ensure that the ConfigMap or Secret is correctly mounted in the pod configuration. For example:

      ```yaml
      volumes:
        - name: config-volume
          configMap:
            name: <configmap_name>
      ```

   3. **Check Pod Logs for Errors**:

      If there are issues, the pod logs may show error messages indicating the problem, such as "File not found" or "Permission denied."

      ```bash
      kubectl logs <pod_name>
      ```

#### **Step 7: Troubleshooting RBAC (Role-Based Access Control) Issues**

1. **Scenario: Unauthorized Access**

   If you’re getting permission issues, it could be due to RBAC misconfigurations.

   1. **Check RBAC Roles and Bindings**:

      Check the roles and role bindings:

      ```bash
      kubectl get roles,rolebindings -n <namespace>
      kubectl get clusterroles,clusterrolebindings
      ```

   2. **Verify Service Account**:

      Ensure that the service account associated with the pod has the correct roles assigned.

#### **Step 8: Verify and Test the Troubleshooting Fixes**

Once you've identified the problem and made adjustments, always verify and test the fixes:

1. **Check Pod Restart Count**:

   If a pod was restarted due to a failure, verify if it’s still restarting:

   ```bash
   kubectl get pod <pod_name> --output=wide
   ```

   Look at the restart count under the `RESTARTS` column.

2. **Test Connectivity**:

   Use `kubectl exec` to test connectivity to other services or nodes:

   ```bash
   kubectl exec -it <pod_name> -- /bin/sh
   ```

   Then try to reach other services, like:

   ```bash
   curl http://<service_name>:<port>
   ```

---

### **Conclusion**

In this exercise, we covered common Kubernetes issues such as pod crashes, networking problems, resource constraints, node issues, pod scheduling failures, and access problems with ConfigMaps, Secrets, and RBAC. 

By using `kubectl describe`, `kubectl logs`, `kubectl get events`, and other commands, we were able to troubleshoot and resolve various issues. Always verify and test your fixes to ensure the stability and health of your Kubernetes cluster.

