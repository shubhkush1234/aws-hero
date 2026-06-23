### Notes for "332 P1 Kubernetes class 2 veera 06.05.2026_07.45.47_REC.mp4"

#### **To-the-Point Summary**

This session covers the foundational architecture of Kubernetes, differentiating between the Control Plane (Master) and Worker Nodes. It details the specific components running on each node type, such as the API Server, ETCD, Kubelet, and Kube-proxy. The class further explores deployment strategies, contrasting self-managed clusters (like KOPS) with Cloud-managed Kubernetes services (EKS, AKS, GKE) where the cloud provider manages the Control Plane. Finally, it addresses the shift in container runtimes, explaining how Kubernetes interfaces with ContainerD via the Container Runtime Interface (CRI) instead of relying directly on Docker, and concludes with practical `eksctl` commands for cluster creation.

---

### **Detailed Notes**

#### **1. Kubernetes High-Level Architecture**

Kubernetes operates on a client-server architecture consisting of two primary types of nodes:

* **Control Node (Control Plane)**: Manages the cluster, orchestrates workloads, and stores the cluster's state.
* **Worker Node**: The actual machines (VMs or physical) where your containerized applications (Pods) run.

```text
+-------------------------------------------------------------+
|                   KUBERNETES CLUSTER                        |
|                                                             |
|  +-----------------------+       +-----------------------+  |
|  |     CONTROL PLANE     |       |     WORKER NODE(S)    |  |
|  |                       |       |                       |  |
|  | [ API Server ]        | <---> | [ Kubelet ]           |  |
|  | [ ETCD ]              |       | [ Kube-proxy ]        |  |
|  | [ Control Manager ]   |       | [ Container Runtime ] |  |
|  | [ Scheduler ]         |       | [ Pods & Containers ] |  |
|  +-----------------------+       +-----------------------+  |
+-------------------------------------------------------------+

```

#### **2. Control Plane Components**

The Control Plane is the brain of Kubernetes. It consists of four main components:

* **API Server (`kube-apiserver`)**: The main component and front-end of the control plane. All communication within the cluster, and from external clients (like `kubectl`), passes through the API server. No other components communicate directly without API server involvement.
* **ETCD**: A highly available, distributed key-value stored database. It stores all cluster configurations, metadata, and state (e.g., which pod is on which node, YAML configurations). Application data is *not* stored here.
* **Control Manager**: Ensures the desired state matches the current state. If you request 3 replicas of a Pod and one crashes, the Control Manager detects this and automatically spins up a new one to maintain the desired state.
* **Scheduler**: Responsible for scheduling Pods onto Worker Nodes based on resource availability (CPU/Memory requirements). It evaluates node capacity and assigns the Pod accordingly.

#### **3. Worker Node Components**

Worker nodes execute the workloads assigned by the Control Plane.

* **Kubelet**: The primary agent running on each worker node. It receives instructions from the Control Plane (API Server) and ensures that the containers described in those instructions are running and healthy.
* **Container Runtime**: The engine responsible for actually running the containers (creating Pods as per Kubelet's instructions).
* **Kubeproxy**: Manages networking rules on the node, enabling internal and external service access and communication to the Pods.

#### **4. Kubernetes Deployment Strategies**

* **KOPS (Kubernetes Operations)**: A tool used to create self-managed clusters. In this model, you are responsible for managing *both* the Control Plane and the Worker Nodes. This can be done on-premises or on cloud servers.
* **Cloud-Managed Kubernetes**: Cloud providers offer managed services where they take full responsibility for the Control Plane. You only manage the Worker Nodes.
* **EKS**: Elastic Kubernetes Service (AWS)
* **AKS**: Azure Kubernetes Service (Azure)
* **GKE**: Google Kubernetes Engine (GCP)



#### **5. Pods vs. Containers and the CRI**

* **Pod**: The smallest deployable unit in Kubernetes. It is an abstract layer on top of a container. A Pod can hold one or multiple containers.
* **Container Runtime Evolution**: Kubernetes is *not* using Docker directly as its containerization platform anymore. It uses its own **Container Runtime Interface (CRI)**.
* *Workflow*: `Developer` -> `API Server` -> `Kubelet` -> `CRI Interface` -> `ContainerD` -> `Creates Containers`.



```text
[ Docker ] ---> [ Dockershim (Deprecated) ]
                       |
[ Kubelet ] ---> [ CRI Plugin ] ---> [ ContainerD ] ---> [ Containers ]

```

#### **6. EKS Cluster Setup Process (`eksctl`)**

To create an Amazon EKS cluster from the CLI, an IAM role must be attached to the EC2 instance (Client machine) giving it programmatic access.

* **Command to create cluster**:
`eksctl create cluster --name <cluster-name> --region <region-name> --node-type <instance-type> --nodes <min> --nodes-max <max>`
* *Example*:
`eksctl create cluster --name test --region us-east-1 --node-type t2.small`

---

### **Trainer Discussed Interview Questions**

1. **What is the function of the API Server in Kubernetes?**
* *Discussion Point*: It acts as the gatekeeper. No internal components can talk to each other without passing through the API server.


2. **Does ETCD store application data?**
* *Discussion Point*: No, ETCD is strictly a key-value store for cluster metadata and configuration (e.g., YAML files, node statuses), not backend application databases.


3. **Why did Kubernetes deprecate direct Docker support, and what is CRI?**
* *Discussion Point*: Docker is a comprehensive toolset, but K8s only needs a runtime. K8s shifted to using the Container Runtime Interface (CRI) to talk directly to lightweight runtimes like `containerd`, removing the heavy `dockershim` layer.


4. **In a Cloud-Managed Kubernetes service (like EKS/AKS), which nodes are you responsible for?**
* *Discussion Point*: The cloud provider manages the Control Plane (API server, ETCD, etc.) completely. You are only responsible for provisioning and maintaining the Worker Nodes.



---

### **Internet Interview Questions & Answers**

*(Minimum 10 questions compiled from leading tech company interviews)*

**1. What is Kubernetes and why do organizations use it?**

* **Asked at**: Google, Microsoft
* `Important` `source: internet`
* **Answer**: Kubernetes is an open-source container orchestration platform that automates deploying, scaling, and managing containerized applications. Organizations use it because it automates container deployment across multiple hosts, provides self-healing (restarts failed containers), enables horizontal scaling, and supports rolling updates with zero downtime.

```text
+---------------------------------------------------+
|               KUBERNETES ORCHESTRATION            |
|  +-------+   +-------+   +-------+   +-------+    |
|  | Pod 1 |   | Pod 2 |   | Pod 3 |   | Pod 4 |    |
|  | (Web) |   | (Web) |   | (API) |   | (API) |    |
|  +-------+   +-------+   +-------+   +-------+    |
|       \         |            |          /         |
|        +--------+--[ Auto-Scaling ]----+          |
+---------------------------------------------------+

```

**2. Explain the difference between a Deployment and a StatefulSet.**

* **Asked at**: Amazon, Netflix
* `Important` `source: internet`
* **Answer**: A Deployment is used for stateless applications (like web servers) where Pods are interchangeable and do not require persistent identities. A StatefulSet is used for stateful applications (like databases) where Pods require persistent, unique network identities (e.g., web-0, web-1) and stable persistent storage.

```text
DEPLOYMENT (Stateless)          STATEFULSET (Stateful)
+-------+ +-------+             +-------+ +-------+
| Pod-A | | Pod-B |             | DB-0  | | DB-1  |
| (Web) | | (Web) |             | (Vol) | | (Vol) |
+-------+ +-------+             +-------+ +-------+
* Random suffixes               * Predictable suffixes
* Shared/No storage             * Dedicated Persistent Volumes

```

**3. What is a Pod and how does it differ from a container?**

* **Asked at**: IBM, Cisco
* `source: internet`
* **Answer**: A Pod is the smallest deployable unit in Kubernetes and represents a logical host for one or more tightly coupled containers. While a container is a single isolated process, a Pod acts as an envelope that allows its containers to share the same network namespace (IP address) and storage volumes.

```text
+-----------------------------------+
|               POD                 |
|  [ IP: 10.0.1.5 ]                 |
|                                   |
|  +-------------+   +-----------+  |
|  | Container 1 |   | Container2|  |
|  |  (App)      |   | (Sidecar) |  |
|  +-------------+   +-----------+  |
|          \            /           |
|         [ Shared Volume ]         |
+-----------------------------------+

```

**4. How do liveness and readiness probes work?**

* **Asked at**: LinkedIn, Apple
* `Important` `source: internet`
* **Answer**: A Liveness Probe checks if the application is alive; if it fails, Kubernetes automatically restarts the pod. A Readiness Probe checks if the application is ready to receive network traffic; if it fails, Kubernetes removes the pod's IP from the service endpoint so it receives no user traffic until it recovers.

```text
     [ Traffic Request ]
             |
             v
+-----------------------------+
|          SERVICE            |
+-----------------------------+
      | (Readiness Pass)
      v
+-----------------------------+
|            POD              |
|  [ App Container ]          | <-- (Liveness Fail? -> Restart!)
+-----------------------------+

```

**5. What is a DaemonSet and when would you use it?**

* **Asked at**: Walmart, Target
* `source: internet`
* **Answer**: A DaemonSet ensures that a copy of a specific Pod runs on all (or selected) nodes in the cluster. It is commonly used for cluster-wide background tasks such as log shippers (e.g., Fluentd), monitoring agents (e.g., Prometheus Node Exporter), and storage daemons.

```text
+--------------+      +--------------+      +--------------+
|   NODE 1     |      |   NODE 2     |      |   NODE 3     |
|              |      |              |      |              |
| [App Pods]   |      | [App Pods]   |      | [App Pods]   |
| [Daemon Pod] |      | [Daemon Pod] |      | [Daemon Pod] |
+--------------+      +--------------+      +--------------+
* Daemon Pod automatically scheduled on every new node.

```

**6. What are the different types of Kubernetes Services?**

* **Asked at**: Uber, Spotify
* `Important` `source: internet`
* **Answer**:
1. **ClusterIP** (default): Exposes the service on a cluster-internal IP, making it only accessible from within the cluster.
2. **NodePort**: Exposes the service on a static port on each Node's IP (between 30000-32767).
3. **LoadBalancer**: Provisions an external load balancer through a cloud provider (AWS/Azure) to expose the service to the internet.
4. **ExternalName**: Maps the service to an external DNS name.



```text
[ External User ]
       |
[ LoadBalancer ] ---> Cloud Provider LB
       |
 [ NodePort ] ------> Node IP:30050
       |
 [ ClusterIP ] -----> Internal IP (10.96.x.x)
       |
    [ Pods ]

```

**7. How is Kubernetes related to Docker, and what is CRI?**

* **Asked at**: RedHat, VMWare
* `Important` `source: internet`
* **Answer**: Historically, Kubernetes heavily relied on Docker to run containers. However, to remain modular and platform-agnostic, Kubernetes developed the Container Runtime Interface (CRI). The CRI is a plugin interface that allows Kubelet to use various container runtimes (like containerd or CRI-O) without needing to recompile Kubernetes, effectively replacing the direct dependency on Docker Engine.

```text
Legacy:  [ Kubelet ] ---> [ Dockershim ] ---> [ Docker ]
Current: [ Kubelet ] ---> [ CRI Plugin ] ---> [ Containerd / CRI-O ]

```

**8. What is Horizontal Pod Autoscaler (HPA), and how does it work?**

* **Asked at**: Salesforce, Oracle
* `source: internet`
* **Answer**: The Horizontal Pod Autoscaler (HPA) automatically scales the number of Pods in a Deployment, ReplicaSet, or StatefulSet based on observed metrics. By default, it scales based on CPU or Memory utilization thresholds, but it can also scale based on custom metrics via external APIs.

```text
[ Metrics Server ] ---> (CPU > 70%)
       |
       v
     [ HPA ] ---> Evaluates Threshold
       |
       v
 [ Deployment ] ---> Scales Pods: [P1] [P2] -> [P1] [P2] [P3] [P4]

```

**9. What are taints and tolerations?**

* **Asked at**: Infosys, Tech Mahindra
* `Important` `source: internet`
* **Answer**: Taints and tolerations work together to ensure pods are not scheduled onto inappropriate nodes. A Taint is applied to a Node to repel a set of pods (e.g., `gpu=true:NoSchedule`). A Toleration is applied to a Pod, allowing (but not requiring) the Pod to be scheduled on a node with a matching taint.

```text
NODE A (Taint: color=blue)         NODE B (No Taints)
       |                                |
  [Repels Pod 1]                  [Accepts Pod 1]
  [Accepts Pod 2]                 [Accepts Pod 2]

* Pod 2 has Toleration: color=blue

```

**10. What is a Namespace and when would you use one?**

* **Asked at**: JP Morgan, Capital One
* `source: internet`
* **Answer**: A Namespace provides a logical partition within a single Kubernetes cluster, allowing multiple teams, projects, or environments (e.g., Dev, Staging, Prod) to share the same physical cluster without interfering with each other. It provides scope for unique object naming and a boundary for applying Resource Quotas and Role-Based Access Control (RBAC).

```text
+-----------------------------------------------------+
|              SINGLE PHYSICAL CLUSTER                |
|                                                     |
|  +-----------------+   +-------------------------+  |
|  | Namespace: DEV  |   | Namespace: PRODUCTION   |  |
|  |                 |   |                         |  |
|  | [Pod: Web-App]  |   | [Pod: Web-App]          |  |
|  | [Secret: db-pw] |   | [Secret: db-pw (Prod)]  |  |
|  +-----------------+   +-------------------------+  |
+-----------------------------------------------------+

```

---

### **Architect Tips for Kubernetes Mastery**

*Assuming the role of a DevOps Architect with 20+ years of industry experience:*

1. **Embrace Multi-Cloud Agnosticism with Terraform:** As you build your expertise across platforms like AWS (EKS) and Azure (AKS), do not rely solely on CLI commands like `eksctl` for production. Transition to Infrastructure as Code (IaC) using Terraform. It allows you to modularize your cluster configurations, maintain state, and deploy identical Kubernetes clusters across different cloud providers using a unified syntax.
2. **Shift Left on Container Security (CRI & CI/CD):** With Kubernetes deprecating Docker in favor of CRI-compliant runtimes like `containerd`, your local build processes still generate images, but production runs them differently. Integrate container scanning and image signing directly into your CI/CD pipelines early. Ensure vulnerabilities are caught *before* the images hit the Kubernetes API server.
3. **Master Observability Beyond Standard Metrics:** Setting up an EKS or AKS cluster is only 20% of the job; operating it at scale is the real challenge. Implement a robust observability stack early on (e.g., Prometheus, Grafana, OpenTelemetry). Correlate application logs with Kubernetes events. If a Pod enters a `CrashLoopBackOff`, your pipeline should alert you with the exact trace context, rather than forcing you to dig manually via `kubectl logs`.
