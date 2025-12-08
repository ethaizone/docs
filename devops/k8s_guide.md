# **Kubernetes Manifests (YAML) and kubectl Guide**

This guide explains the fundamental components to look for when reading Kubernetes configuration files (YAML manifests) and outlines the essential kubectl commands used for managing those resources.

## **1\. How to Read Kubernetes YAML Manifests**

The configuration of any Kubernetes resource follows a predictable structure. Understanding these core fields is key to quickly grasping the resource's purpose.

### **A. Core Fields (All Resources)**

| Field | Description | Purpose |
| :---- | :---- | :---- |
| apiVersion | Specifies the API version, e.g., apps/v1, v1, autoscaling/v2. | Used by Kubernetes to know which API to use when parsing the object. |
| kind | **The type of resource being created.** | **Essential**: Identifies the object, e.g., Deployment, Service, HorizontalPodAutoscaler. |
| metadata | Data about the object, such as name, namespace, labels, and annotations. | metadata.name is the unique identifier (item name) used in kubectl commands. |
| spec | The desired state of the object. | Defines *what* the object should be and how it should behave. |

### **B. Analyzing Specific Resource Kinds**

For this application, we focus on three core kinds: Deployment, Service, and HorizontalPodAutoscaler (HPA).

#### **Deployment (kind: Deployment)**

Deployments manage the desired state of ReplicaSets, which in turn manage Pods.

| Key Field | Description | Relationship to Other Resources |
| :---- | :---- | :---- |
| spec.replicas | The desired number of Pods (replicas) to run. | Directly controls the Pod count, but can be overridden by an HPA. |
| spec.template | The template for the Pods managed by the Deployment. | This is what defines the container image, environment variables, ports, etc. |
| **Pod Control** | A Pod resource (kind: Pod) is typically **created automatically** by a Deployment's ReplicaSet. You usually interact with the Deployment, not the individual Pods. |  |

#### **Service (kind: Service)**

Services define a logical set of Pods and a policy by which to access them.

| Key Field | Description | Networking and Access |
| :---- | :---- | :---- |
| spec.selector | Defines which Pods the Service targets, based on their labels. | **Crucial Link**: Must match the Pod labels defined in the Deployment's spec.template.metadata.labels. |
| spec.type | Defines how the service is exposed (ClusterIP, NodePort, LoadBalancer, etc.). | Affects external accessibility. NodePort often exposes a static port on each Node (e.g., 40001 in your example). |
| spec.ports | Maps the Service port (internal/external) to the target Pod port. | Used for internal/external routing. |

#### **HorizontalPodAutoscaler (HPA) (kind: HorizontalPodAutoscaler)**

HPAs automatically scale the number of Pod replicas in a Deployment or ReplicaSet.

| Key Field | Description | Scaling Configuration |
| :---- | :---- | :---- |
| spec.scaleTargetRef | **The target resource to scale.** | Typically points to the Deployment's API version and name. **It controls the Deployment's spec.replicas.** |
| spec.minReplicas / spec.maxReplicas | The boundaries for the Pod count. | Ensures the application never scales below or above these limits. |
| spec.metrics | The conditions that trigger scaling (e.g., CPU utilization above 80%). | Defines the performance threshold for scaling up or down. |

### **C. Reading ConfigMaps**

ConfigMaps store non-confidential data in key-value pairs. They are often used to set environment variables or configuration files.

**Creation from .env file:** A common approach is to generate the ConfigMap YAML from an environment file (.env) and apply it:

`kubectl -n ${NAMESPACE} create configmap book-api --from-env-file=.env -o yaml --dry-run=client | kubectl apply -f -`

**Viewing values in Kubernetes:** You can read the key-value data stored in a deployed ConfigMap using the describe command:

`kubectl describe configmap book-api`

## **2\. Essential kubectl Commands**

The following commands are essential for managing and debugging resources defined in your YAML manifests.

### **A. Inspection and Viewing**

| Command | Purpose | Example |
| :---- | :---- | :---- |
| `kubectl get {kind}` | Get a list of resources of a specific kind in the current namespace. | `kubectl get deployment` |
| `kubectl get {kind} {item name}` | Get the status of a specific resource. | `kubectl get service backend-api` |
| `kubectl describe {kind} {item name}` | See a detailed description, including events, conditions, and environment variables. | `kubectl describe pod book-api-abcd123-xyz` |

### **B. Applying and Editing Configuration**

**Recommended Workflow (File-based Apply):**

It is strongly recommended to update configuration by editing the source YAML file and using kubectl apply. This ensures your source code repository remains the single source of truth.

`kubectl apply -f deployment/staging/main/service.yaml`

**Direct Editing (Use with Caution):**

The edit command allows you to change the live configuration directly in the cluster. This is generally discouraged for permanent changes as it bypasses the source YAML file, but it's useful for quick temporary fixes.

`kubectl edit {kind} {item name}`

### **C. Deletion and Cleanup**

| Command | Purpose | Example |
| :---- | :---- | :---- |
| `kubectl delete {kind} {item name}` | Permanently delete the specified resource. | `kubectl delete deployment book-api` |
| `kubectl delete -f {file}` | Delete resources defined in a specific YAML file. | `kubectl delete -f deployment/staging/main/service.yaml` |

### **D. Debugging and Networking**

#### **Port Forwarding (Local Access)**

Port forwarding creates a secure tunnel from your local machine to a resource (usually a Service or Pod) inside the cluster, allowing you to make requests to it as if it were running locally. This is typically used for local development and testing.

\# This forwards requests from localhost:4012 to the container port (e.g., 40001\)  
\# exposed by the service/backend-api.  
\# Access URL: `http://localhost:4012`

`kubectl port-forward service/backend-api 4012:40001`

#### **Internal Service Call Construction**

When making a request from one Pod to another Service **within the same cluster**, the URL should be constructed using the Kubernetes Service Discovery pattern:

`http://{service name}.{namespace}.svc:{service port}`

**Example:** If your backend Service is named backend-api in the book namespace, and it exposes port 8080, the URL for internal calls would be:

[http://backend-api.book.svc:8080](http://backend-api.book.svc:8080)

This pattern resolves the service name to the correct cluster IP, enabling reliable communication between your microservices.

**Summary of Key Takeaways:**

1. **YAML Flow:** kind defines *what* it is, and metadata.name defines *its name*.  
2. **Deployment \-\> Service Link:** The Service's selector must match the Pod's labels defined in the Deployment.  
3. **Source of Truth:** Always apply changes from your version-controlled YAML files using kubectl apply \-f ... instead of direct editing.
