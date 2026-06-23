Here are the consolidated, detailed notes covering both sessions of the Kubernetes training, enhanced with High-Level Design (HLD), Low-Level Design (LLD), configuration snippets, and an expanded interview section.

---

# Kubernetes Architecture & Practical Deployment Strategies

### **To-the-Point Summary**

These sessions cover the foundational architecture and practical deployment of Kubernetes. The architecture is split between the Control Plane (API Server, ETCD, Control Manager, Scheduler), which manages the cluster state, and Worker Nodes (Kubelet, Kube-proxy, Container Runtime), which execute workloads. Deployment strategies contrast self-managed clusters (KOPS) with Cloud-Managed solutions (AWS EKS, Azure AKS, GCP GKE). The transition from Docker to the Container Runtime Interface (CRI) with `containerd` is highlighted. Practically, the sessions demonstrate using `eksctl` to provision AWS infrastructure and `kubectl` to deploy a foundational Nginx Pod using declarative YAML configurations, emphasizing the shift from imperative commands to state-driven management.

---

### **System Design: High-Level (HLD) & Low-Level (LLD)**

#### **High-Level Design (HLD): Cloud-Managed Kubernetes (AWS EKS)**

The HLD illustrates how external traffic routes through cloud infrastructure into the managed Kubernetes environment.

```text
[External Users]
       | (HTTPS)
[ AWS Elastic Load Balancer (ELB) ]
       |
+-------------------------------------------------------+
|                    AWS VPC BOUNDARY                   |
|                                                       |
|  +-------------------------------------------------+  |
|  |             EKS CONTROL PLANE (Managed)         |  |
|  |  [API Server]  [ETCD]  [Scheduler] [Manager]    |  |
|  +------------------------+------------------------+  |
|                           | (gRPC / TLS)              |
|  +------------------------v------------------------+  |
|  |               WORKER NODE GROUP (EC2)           |  |
|  |                                                 |  |
|  |  [ Node 1: Kubelet, Proxy, Pods ]               |  |
|  |  [ Node 2: Kubelet, Proxy, Pods ]               |  |
|  +-------------------------------------------------+  |
+-------------------------------------------------------+

```

#### **Low-Level Design (LLD): Worker Node & Pod Execution**

The LLD focuses on the internal mechanics of a single Worker Node, specifically how Kubelet interacts with the CRI to run containers.

```text
+-------------------------------------------------------+
|                    WORKER NODE                        |
|                                                       |
|  [ Kube-Proxy ] <--- Manages iptables / networking    |
|                                                       |
|  [ Kubelet ] <--- Receives Pod specs from API Server  |
|       |                                               |
|  (CRI - gRPC)                                         |
|       v                                               |
|  [ ContainerD ] <--- Container Runtime                |
|       |                                               |
|       +---> [ Pod (Network Namespace) ]               |
|                   |                                   |
|                   +---> [ Container: Nginx ]          |
|                   +---> [ Container: Log Agent ]      |
+-------------------------------------------------------+

```

---

### **Detailed Notes: Step-by-Step Explanation**

#### **1. Kubernetes Component Architecture**

Kubernetes operates on a client-server model:

* **Control Plane (Master Node)**:
* **API Server (`kube-apiserver`)**: The central communication hub. All internal components and external users (via `kubectl`) communicate *only* through here.
* **ETCD**: A highly available, distributed key-value store. It persists cluster state, YAML configurations, and metadata. It does *not* store application user data.
* **Control Manager**: Constantly monitors the cluster to ensure the "Current State" matches the "Desired State" defined in your YAML.
* **Scheduler**: Evaluates CPU/RAM requirements of new Pods and assigns them to the optimal Worker Node.


* **Worker Nodes (The Muscle)**:
* **Kubelet**: The primary node agent. It takes orders from the API Server and ensures containers are running and healthy.
* **Kube-proxy**: Maintains network routing rules, allowing Pods to talk to each other and external services.
* **Container Runtime**: Pulls images and runs them (via CRI).



#### **2. The Container Runtime Interface (CRI) Evolution**

Kubernetes no longer relies directly on Docker. To remain modular, it introduced the CRI.

* **Workflow**: `Developer` -> `API Server` -> `Kubelet` -> `CRI Plugin` -> `ContainerD` -> `Spins up Containers`.
* This removed the heavy `dockershim` layer, making node execution faster and more secure.

#### **3. Deployment Models**

* **Self-Managed (KOPS)**: You build and maintain both the Control Plane and Worker Nodes. High overhead.
* **Cloud-Managed (EKS/AKS/GKE)**: The cloud provider manages the Control Plane for high availability. You only provision and manage the Worker Nodes.

#### **4. Practical Setup: Infrastructure Provisioning**

To create the AWS EKS cluster, the CLI tool `eksctl` is used. It automates underlying CloudFormation templates.

* **Step**: Attach an IAM role to your client machine (EC2) for programmatic access.
* **Command snippet**:
```bash
eksctl create cluster \
  --name my-cluster \
  --region us-east-1 \
  --node-type t3.medium \
  --nodes 2 \
  --nodes-max 4

```



#### **5. Declarative Workload Deployment (YAML)**

Instead of imperative commands (`kubectl run`), production environments use declarative YAML.

* **The 4 Mandatory YAML Fields**:
1. `apiVersion`: e.g., `v1`, `apps/v1`
2. `kind`: e.g., `Pod`, `Deployment`
3. `metadata`: `name`, `labels`
4. `spec`: The desired state (containers, images, ports).


* **Step**: Create `pod.yml` for Nginx:
```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-first-pod
  labels:
    app: webapp
    tier: frontend
spec:
  containers:
  - name: nginx-container
    image: nginx:latest
    ports:
    - containerPort: 80

```



#### **6. Execution & Inspection Steps**

* **Deploy**: `kubectl apply -f pod.yml` (Sends spec to API Server -> stored in ETCD -> Scheduled).
* **Verify**: `kubectl get pods` (Shows status like `ContainerCreating` or `Running`).
* **Deep Dive**: `kubectl describe pod my-first-pod`. Crucial for debugging. The `Events` section shows the timeline: Scheduled -> Image Pulled -> Container Created -> Started.
* **Teardown**: `kubectl delete -f pod.yml`.

---

### **Trainer Discussed Interview Questions**

1. **What is the function of the API Server?**
* *Answer*: It acts as the cluster gatekeeper. No internal components can talk to each other directly; everything goes through the API server via REST requests.


2. **Does ETCD store application data?**
* *Answer*: No, strictly cluster metadata, state, and declarative configurations.


3. **Why did Kubernetes deprecate direct Docker support?**
* *Answer*: Docker is a massive toolset. K8s shifted to CRI to talk directly to lightweight runtimes like `containerd`, removing unnecessary bloat.


4. **What is the difference between `kubectl run` and `kubectl apply`?**
* *Answer*: `run` is imperative (manual command, hard to track). `apply` is declarative (uses YAML files, enabling GitOps and version control).



---

### **Internet Interview Questions & Answers**

**1. What is a Pod and how does it differ from a container?**

* **Asked at**: IBM, Cisco
* `source: internet`
* **Answer**: A Pod is the smallest deployable unit. It acts as an envelope that can hold one or multiple tightly coupled containers. Containers inside a Pod share the same network namespace (IP) and storage volumes.

```text
[ Diagram 1: Pod Architecture ]
+-----------------------------------+
|               POD                 |
|  [ Shared IP: 10.0.1.5 ]          |
|                                   |
|  +-------------+   +-----------+  |
|  | Container 1 |   | Container2|  |
|  |  (App)      |   | (Sidecar) |  |
|  +-------------+   +-----------+  |
|          \            /           |
|         [ Shared Volume ]         |
+-----------------------------------+

```

**2. Explain the difference between a Deployment and a StatefulSet.**

* **Asked at**: Amazon, Netflix
* `Important` `source: internet`
* **Answer**: A Deployment is for stateless applications (like web servers) where Pods are interchangeable. A StatefulSet is for stateful applications (databases) where Pods require persistent, predictable identities (e.g., db-0, db-1) and stable persistent storage.

```text
[ Diagram 2: Deployment vs StatefulSet ]
DEPLOYMENT (Stateless)          STATEFULSET (Stateful)
+-------+ +-------+             +-------+ +-------+
| Pod-A | | Pod-B |             | DB-0  | | DB-1  |
| (Web) | | (Web) |             | (Vol) | | (Vol) |
+-------+ +-------+             +-------+ +-------+
* Random suffixes               * Predictable names

```

**3. What is Horizontal Pod Autoscaler (HPA)?**

* **Asked at**: Salesforce, Oracle
* `source: internet`
* **Answer**: HPA automatically scales the number of Pods in a Deployment or ReplicaSet up or down based on observed metrics like CPU utilization or custom API metrics.

```text
[ Diagram 3: HPA Flow ]
[ Metrics Server ] ---> (CPU > 75%)
       |
       v
     [ HPA ] ---> Evaluates Threshold
       |
       v
 [ Deployment ] ---> Scales: [P1] [P2] -> [P1] [P2] [P3]

```

**4. How do liveness and readiness probes differ?**

* **Asked at**: LinkedIn, Apple
* `Important` `source: internet`
* **Answer**: A Liveness Probe restarts a pod if it is dead/stuck. A Readiness Probe temporarily removes a pod from the Service load balancer if it is alive but not yet ready to process traffic.

```text
[ Diagram 4: Probes ]
     [ External Traffic ]
             |
+-----------------------------+
|          SERVICE            |
+-----------------------------+
      | (Readiness Pass)
      v
+-----------------------------+
|            POD              | <-- (Liveness Fail? -> Restart Pod)
|  [ App Container ]          |
+-----------------------------+

```

**5. What are the different types of Kubernetes Services?**

* **Asked at**: Uber, Spotify
* `Important` `source: internet`
* **Answer**:
* `ClusterIP`: Internal access only (default).
* `NodePort`: Exposes service on a static port on the Node's IP.
* `LoadBalancer`: Provisions a cloud provider's external load balancer.



```text
[ Diagram 5: Services ]
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

**6. What are taints and tolerations?**

* **Asked at**: Infosys, Tech Mahindra
* `Important` `source: internet`
* **Answer**: Taints are applied to Nodes to repel Pods (e.g., reserving a node for GPU workloads). Tolerations are applied to Pods, allowing them to schedule on tainted nodes.

```text
[ Diagram 6: Taints & Tolerations ]
NODE A (Taint: gpu=true)         NODE B (No Taints)
       |                                |
  [Repels Pod 1]                  [Accepts Pod 1]
  [Accepts Pod 2]                 [Accepts Pod 2]

* Pod 2 has Toleration: gpu=true

```

**7. What is a DaemonSet?**

* **Asked at**: Walmart, Target
* `source: internet`
* **Answer**: A DaemonSet ensures that exactly one copy of a Pod runs on all (or selected) nodes in the cluster. Used for monitoring agents or log collectors.

```text
[ Diagram 7: DaemonSet ]
+--------------+      +--------------+
|   NODE 1     |      |   NODE 2     |
| [App Pods]   |      | [App Pods]   |
| [Daemon Pod] |      | [Daemon Pod] |
+--------------+      +--------------+

```

**8. Explain the role of `eksctl` vs `kubectl`.**

* **Asked at**: Amazon, Capital One
* `source: internet`
* **Answer**: `eksctl` provisions AWS infrastructure (VPCs, EC2 instances, EKS Control plane). `kubectl` interacts with the Kubernetes API Server to deploy and manage application workloads on the cluster.

```text
[ Diagram 8: CLI Tools ]
[ eksctl ] ---> Creates ---> [ AWS Infrastructure (VPC, EC2) ]
                                      |
[ kubectl ] --> Deploys ---> [ K8s API Server / Workloads ]

```

**9. How does Kubernetes handle persistent storage?**

* **Asked at**: NetApp, Oracle
* `source: internet`
* **Answer**: Storage is decoupled from Pods using Persistent Volumes (PV - the actual storage block) and Persistent Volume Claims (PVC - the request for storage by the Pod).

```text
[ Diagram 9: Storage Architecture ]
[ Pod ] ---> Mounts ---> [ PVC ] ---> Binds to ---> [ PV ] ---> [ Cloud Storage (e.g., EBS) ]

```

**10. What is a Namespace?**

* **Asked at**: JP Morgan, Capital One
* `source: internet`
* **Answer**: Namespaces are virtual clusters backed by the same physical cluster hardware. They provide logical isolation for different environments (Dev, Test, Prod) or teams.

```text
[ Diagram 10: Namespaces ]
+---------------------------------------------+
|              PHYSICAL CLUSTER               |
|  +-----------------+   +-----------------+  |
|  | Namespace: DEV  |   | Namespace: PROD |  |
|  | [Pod: Web-App]  |   | [Pod: Web-App]  |  |
|  +-----------------+   +-----------------+  |
+---------------------------------------------+

```

---

### **Tips from a 20-Year DevOps Architect**

1. **Embrace Multi-Cloud Infrastructure as Code (IaC):** While `eksctl` is fantastic for spinning up an AWS EKS cluster quickly, enterprise environments demand a multi-cloud strategy (e.g., balancing workloads between AWS EKS and Azure AKS). Transition from manual CLI creation to using **Terraform**. Writing modular Terraform code allows you to maintain the state of your infrastructure securely and replicate identical clusters across different cloud providers seamlessly.
2. **Integrate CI/CD Pipelines with GitOps:** When deploying applications, such as a ReactJS frontend coupled with a MongoDB backend (MERN stack), manual `kubectl apply` commands should be avoided. Implement robust CI/CD pipelines (using tools like GitHub Actions or Jenkins). Build your Docker containers, push them to a registry, and use GitOps tools like ArgoCD. ArgoCD watches your Git repository for YAML changes and automatically syncs the cluster state, ensuring your pipeline handles the end-to-end delivery without human intervention.
3. **Prioritize Observability and Deep Live Training:** Setting up the architecture is only the beginning; operating it safely is the true challenge. Implement observability stacks (Prometheus/Grafana) immediately to monitor cluster health. Furthermore, mastering these complex interactions requires structured, continuous learning. Treat your architecture education like live instructor-led training batches—where you participate in live labs, tackle LLMOps integration, and constantly validate your configurations in real-time sandbox environments.
