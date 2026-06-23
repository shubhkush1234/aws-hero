# Advanced DevOps Engineering Notes: Self-Hosted CI/CD Runners Across Azure DevOps, GitHub Actions, and GitLab CI/CD

## To-the-Point Summary

This comprehensive technical guide details the transition from cloud-hosted CI/CD environments to custom, private infrastructure using **Self-Hosted Agents (Runners)**. It covers resolving compute resource bottlenecks (e.g., Azure DevOps parallelism limits) by provisioning dedicated **AWS EC2 target instances**, installing execution agents via **Personal Access Tokens (PAT)**, and hardening execution environments.

Crucially, the final module of the workshop unifies this architecture across multiple platforms. It proves that the underlying CI/CD lifecycle—Docker build, push to Azure Container Registry (ACR), SSH to target server, pull, and run—is identical across **Azure DevOps**, **GitHub Actions**, and **GitLab CI/CD**. The only variables that change are the repository hosting the code, the name of the YAML configuration file, and the terminology used for the execution server (Agent vs. Runner).

---

## Detailed Step-by-Step Training Notes

### Step 1: Extracting Azure Container Registry (ACR) Access Keys

To establish a secure connection from any container build agent to the container registry, administrative authentication credentials must be gathered.

1. Navigate to the **Azure Portal** and open your **Container Registry** (Registry Name: `narni`).
2. Under the **Settings** menu on the left sidebar, click on **Access keys**.
3. Toggle the **Admin user** setting to **Enabled**.
4. Copy the connection properties: **Login server** (`narni.azurecr.io`), **Username**, and **Password**.

### Step 2: Configuring Variable Groups in Azure DevOps

Storing secrets directly inside pipeline source code is an anti-pattern.

1. Open Azure DevOps project -> **Pipelines** -> **Library**.
2. Click **+ Variable group** and set the Name to `acrlibrary`.
3. Add variables: `ACR_USERNAME` and `ACR_PASSWORD`.
4. Save the group.

### Step 3: Troubleshooting the Pipeline Parallelism Bottleneck

When executing a standard containerized pipeline (`azure-pipelines.yml`) using Microsoft-hosted infrastructure, free-tier accounts hit compute restrictions:

> `##[error]No hosted parallelism has been purchased or granted.`

* **The Manual Fix:** Request a free parallelism grant from Microsoft (takes 24-48 hours).
* **The Architect's Resolution:** Bypass external infrastructure limits completely by introducing a **Custom Self-Hosted Agent** running inside your own private VPC network.

### Step 4: Setting Up an Azure DevOps Self-Hosted Agent Pool

1. Navigate to **Project Settings** -> **Agent pools**.
2. Click **Add pool** -> Set **Pool type** to `Self-hosted` -> Name it `nit`.
3. Check **Grant access permission to all pipelines** and click **Create**.

### Step 5: Provisioning and Setting Up the AWS EC2 Host

1. Log into **AWS Management Console** -> **EC2 Dashboard**.
2. Launch an `Amazon Linux 2023` (or Ubuntu) instance named `ADO-agent` of size `t2.medium`.
3. Connect to the instance terminal.

### Step 6: Installing and Configuring the Agent Binary

1. Generate an Azure DevOps **Personal Access Token (PAT)** with Full Access.
2. In the EC2 terminal, download and extract the Linux guest execution agent package.
3. Run `./config.sh`.
4. Enter your Azure DevOps URL, select PAT authentication, paste your token, and link it to the `nit` pool.
5. Run `./run.sh` to start the daemon.

### Step 7: Troubleshooting Runtime Failures on Private Agents

* **Missing Git:** Run `sudo yum install git -y` on the EC2 host.
* **Missing Docker Engine:** Run `sudo yum install docker -y` and `sudo systemctl start docker`.
* **Socket Permission Block:** The agent daemon runs under a non-root user and cannot access Docker. Run `sudo chmod 666 /var/run/docker.sock` to bypass permission denied errors. *(Note: See Interview Q2 for why this is bad in production).*

### Step 8: The Unified Multi-Cloud CI/CD Architecture (Final Lecture Segment)

In the final segment, the trainer emphasizes that DevOps is a methodology, not a specific tool. The architectural diagram of the pipeline remains exactly the same whether you use Azure, GitHub, or GitLab.

#### The Core Execution Flow (Identical across tools):

1. **Source Code Check-in**
2. **Docker Build** on Self-Hosted Agent.
3. **Docker Push** to ACR.
4. **SSH into Target Server** (from the Agent).
5. **Docker Pull & Run** on the Target Server.

#### The Platform Differences Matrix:

| Feature | Azure DevOps | GitHub Actions | GitLab CI/CD |
| --- | --- | --- | --- |
| **Pipeline File Path** | `/azure-pipelines.yml` | `/.github/workflows/deploy.yml` | `/.gitlab-ci.yml` |
| **Agent / Worker Name** | Self-Hosted Agent | Self-Hosted Runner | GitLab Runner |
| **Code Hosting** | Azure Repos | GitHub | GitLab |

*If the Self-Hosted Agent/Runner and the Target Server are in different Virtual Private Clouds (VPCs), you must establish a **VPC Peering Connection** or use an **AWS Transit Gateway** to allow the agent to SSH into the target server.*

---

## Trainer-Discussed Interview Questions Sections

*These practical application questions were explicitly discussed during the step-by-step troubleshooting sequence:*

1. **Why does a newly provisioned self-hosted agent throw a "File not found: git" error, and how do you resolve it?**
2. **If a pipeline requires running container commands, why isn't simply running `yum install docker` enough for a self-hosted agent?**
3. **What is the exact security issue surrounding `/var/run/docker.sock` encountered during self-hosted builds, and how do you quickly address it in a lab vs. production environment?**
4. **How do the configuration directory paths and structural naming conventions differ when setting up pipelines for GitHub Actions versus GitLab CI/CD?**

---

## L2 & L3 Interview Questions & Answers

*[Source: Internet]*

### Q1: Advanced Multi-Cloud Pipeline Security Mapping [Important]

> **Asked at: Barclays, HSBC**
> **Question:** When setting up a multi-cloud CI/CD pipeline using a self-hosted runner inside an AWS private subnet that targets an Azure Container Registry (ACR), how do you design authentication to avoid storing long-lived access keys on the host runner?

**Answer:**
You should disable the ACR Admin account password to prevent key-leak vectors. Implement tokenless authentication using **OpenID Connect (OIDC)** and **AWS IAM Identity Center** coupled with cross-cloud federation. Assign an AWS IAM Role to the EC2 runner. The runner uses its local AWS metadata token to request a federated, short-lived Azure Entra ID access token to push the image.

```text
┌────────────────────────┐              Trust Setup              ┌────────────────────────┐
│   AWS Private Subnet   │ ────────────────────────────────────> │     Azure Entra ID     │
│  ┌──────────────────┐  │                                       └────────────────────────┘
│  │Self-Hosted Runner│  │ <─ 1. Exchange Federated Token ───────────┘         │
│  │ (Assumed IAM Role)│  │                                                    │ 2. Scoped Token
│  └──────────────────┘  │ ── 3. Push Container Image (Secure Token) ───────> ▼
└────────────────────────┘                                       ┌────────────────────────┐
                                                                 │ Azure Container Registry│
                                                                 └────────────────────────┘

```

### Q2: Production Isolation of the Docker Daemon Socket [Important]

> **Asked at: Walmart Global Tech, Ericsson**
> **Question:** Executing `chmod 666 /var/run/docker.sock` resolves permission blocks but introduces vulnerabilities. Explain why this is dangerous and describe an enterprise-grade remediation strategy.

**Answer:**
`chmod 666` allows any local user to write to the Docker UNIX socket, enabling complete host privilege escalation (e.g., spinning up a container with the root filesystem mounted). In production, you must use **Docker Rootless Mode** or add the specific runner user account to a dedicated POSIX `docker` group (`sudo usermod -aG docker azdevops-agent`) rather than opening permissions globally.

```text
[ Insecure Vulnerable Method ]
  Standard User Host ──> Write to Socket (666 Permission) ──> Daemon Processes as System Root ──> ❌ Full Escalation

[ Production Remediation Method ]
  Runner Service User ──> Member of Restricted 'docker' Group ──> Scoped UNIX Group Verification ──> ✅ Host Isolated

```

### Q3: High-Availability Autoscaling Matrix for Custom Runners

> **Asked at: Netflix, Uber**
> **Question:** How do you design a highly available, self-healing pool of self-hosted runners that scales to zero during off-peak hours?

**Answer:**
Move away from static EC2 hosts and adopt an event-driven design using **Actions Runner Controller (ARC)** on Kubernetes (EKS) or an **AWS Auto Scaling Group (ASG)**. Using Kubernetes Event-driven Autoscaling (KEDA), the cluster monitors queue metrics via CI/CD APIs. When the queue is empty, replica counts drop to zero. As jobs arrive, ephemeral runner pods are provisioned on demand.

```text
                                        ┌──────────────────────────────┐
                                        │  Webhook Event Controller    │
                                        └──────────────────────────────┘
                                                       │
                                        Monitors Queue │ (Scale Up / Down Pods)
                                                       ▼
┌────────────────────────────────────────────────────────────────────────────────────────┐
│ Kubernetes Compute Cluster (EKS)                                                        │
│   ▲                          ▲                          ▲                          ▲   │
│ ┌─┴────────────┐           ┌─┴────────────┐           ┌─┴────────────┐           ┌─┴─┐ │
│ │ Runner Pod 1 │           │ Runner Pod 2 │           │ Runner Pod 3 │           │...│ │
└────────────────────────────────────────────────────────────────────────────────────────┘

```

### Q4: Cross-VPC Subnet Networking & Proxy Routing Matrix

> **Asked at: Cisco Systems, AWS Core Delivery Teams**
> **Question:** A self-hosted runner is placed in an isolated private subnet with no direct internet access. How can it securely communicate with GitHub/Azure endpoints?

**Answer:**
Use PrivateLink VPC Endpoints for internal AWS services (ECR, S3). For external APIs, route traffic through an authenticated corporate forward proxy (e.g., Squid) in a transit VPC. Configure the agent daemon to respect the proxy by creating a `.proxy` file in the agent root directory.

```text
┌─────────────────────────────────────┐            Forward Proxy           ┌────────────────────────┐
│ Private Subnet (No NAT Gateway)     │            Traffic Link            │ Public Web Endpoints   │
│  ┌────────────────────────┐         │     ┌────────────────────────┐     │ ┌────────────────────┐ │
│  │   Self-Hosted Agent    │ ───────┼───> │ Corporate Squid Proxy  │ ───> │ │ dev.azure.com      │ │
│  │ (Reads .proxy Config)  │         │     │     (Transit VPC)      │     │ └────────────────────┘ │
│  └────────────────────────┘         │     └────────────────────────┘     └────────────────────────┘
└─────────────────────────────────────┘

```

### Q5: Stateful Build Cache Persistency Across Ephemeral Infrastructure

> **Asked at: Spotify, Adobe**
> **Question:** Ephemeral runner architectures slow down pipelines because dependencies must be redownloaded. How do you implement a scalable caching layer?

**Answer:**
Mount an **Amazon EFS (Elastic File System)** volume to the underlying cluster nodes, mapping cache directory paths directly into the runner pod templates. Alternatively, use native pipeline caching commands (e.g., `Cache@2` in Azure, `actions/cache` in GitHub) backed by an S3 bucket to push/pull dependency layers dynamically.

```text
┌────────────────────────┐     ┌────────────────────────┐
│     Ephemeral Pod A    │     │     Stateful Pod B     │
└────────────────────────┘     └────────────────────────┘
            │                              │
            └──────────────┬───────────────┘
                           │ Network Mount Link (Concurrent Read/Write)
                           ▼
              ┌────────────────────────┐
              │  Amazon EFS Volume     │
              │ (/var/ci-build/caches) │
              └────────────────────────┘

```

### Q6: Zero-Downtime Agent Migrations & Pool Draining [Important]

> **Asked at: Salesforce, Capital One**
> **Question:** How do you perform maintenance on an EC2 instance hosting a long-running runner pool without interrupting active builds?

**Answer:**
Execute a graceful node drain. First, disable the agent pool in the platform UI so no new jobs are assigned. Second, deploy a script that monitors the system process tree (e.g., `pgrep -f "Agent.Worker"`). The script loops and sleeps until the active process terminates, at which point it safely executes system teardown operations.

```text
                                     Pipeline Jobs Flow
                                              │
                    New Jobs Allowed   ┌──────┴──────┐  ❌ Blocked (Pool Set to Disabled)
                                       ▼             ▼
                             ┌──────────────┐   ┌──────────────┐
                             │ Active Host  │   │ Draining Node│ ──> Teardown Once
                             │  (Runner)    │   │  (EC2 Host)  │     Jobs Hit 0
                             └──────────────┘   └──────────────┘

```

### Q7: Managing Conflicting Sub-Dependency Binaries

> **Asked at: Capgemini, Accenture**
> **Question:** A static self-hosted runner handles two pipelines: one needs Python 3.9, the other Python 3.11. How do you prevent conflicts?

**Answer:**
Do not install toolchains globally. Use **Containerized Execution Steps** (e.g., `container: image: python:3.11-slim` in GitHub Actions) so that jobs execute inside isolated Docker environments rather than the host shell. Alternatively, leverage virtual environment managers like `asdf` or `pyenv` scoped to the specific pipeline workspace.

```text
                                 [ Shared Subnet Shell Run Host ]
                                                 │
                        ┌────────────────────────┴────────────────────────┐
                        ▼                                                 ▼
          ┌──────────────────────────┐                      ┌──────────────────────────┐
          │ Pipeline A Workspace     │                      │ Pipeline B Workspace     │
          │ ┌──────────────────────┐ │                      │ ┌──────────────────────┐ │
          │ │ Container: Python3.9 │ │                      │ │ Container: Python3.11│ │
          │ └──────────────────────┘ │                      │ └──────────────────────┘ │
          └──────────────────────────┘                      └──────────────────────────┘

```

### Q8: Hardening Secret Injections on Self-Hosted Nodes

> **Asked at: Visa, PayPal**
> **Question:** How do you secure sensitive environment variables on long-lived infrastructure to prevent configuration drift or leakage?

**Answer:**
Fetch configurations at runtime using external vaults like **AWS Secrets Manager** or **HashiCorp Vault**. Configure the runner to wipe its `_work` directory post-execution (`cleanup: true`). Utilize pipeline commands (e.g., `echo "::add-mask::$SECRET"`) to explicitly strip secret strings from local memory and standard output logs.

```text
┌────────────────────────┐   1. Fetch Value   ┌────────────────────────┐
│   Self-Hosted Runner   │ ─────────────────> │   AWS Secrets Manager  │
│                        │ <─ 2. Inject (RAM) └────────────────────────┘
│  ┌──────────────────┐  │
│  │::add-mask::****  │  │ ── 3. Stripped / Obfuscated Out ──> ❌ Public Output Logs
│  └──────────────────┘  │
└────────────────────────┘

```

### Q9: Agent to Target Cross-VPC Communication [Important]

> **Asked at: IBM, Deloitte**
> **Question:** If your CI/CD runner is deployed in `VPC A` and your target deployment server is in `VPC B`, how do you enable the runner to SSH into the target server securely?

**Answer:**
Since private IP addresses cannot route over the internet, you must implement a **VPC Peering Connection** (if both VPCs are in the same region without overlapping CIDR blocks) or attach both VPCs to an **AWS Transit Gateway**. You must also update the Security Group of the target server to allow inbound SSH (Port 22) traffic explicitly from the runner's CIDR block.

```text
┌──────────────────────────┐                               ┌──────────────────────────┐
│      VPC A (Agent)       │      VPC Peering Link         │     VPC B (Target)       │
│  ┌────────────────────┐  │ ◄───────────────────────────► │  ┌────────────────────┐  │
│  │ Self-Hosted Runner │  │     (Routes Updated)          │  │   Target EC2 Host  │  │
│  └────────────────────┘  │                               │  └────────────────────┘  │
└──────────────────────────┘                               └──────────────────────────┘

```

### Q10: Disk Space Exhaustion on Static Nodes

> **Asked at: Splunk, Datadog**
> **Question:** Long-running runners frequently crash due to accumulated dangling Docker images. How do you design zero-downtime maintenance?

**Answer:**
Implement proactive cleanup hooks in the pipeline (e.g., `docker image prune -af --filter "until=24h"`) as a conditional post-execution step (`always()`). Furthermore, schedule a host-level system `cron` daemon to purge orphan volumes and networks nightly. Combine this with Datadog/Prometheus alerts that trigger auto-remediation scripts when disk IO capacity drops below 20%.

```text
┌────────────────────────────────────────────────────────┐
│  Self-Hosted Instance Node Running Builds              │
│                                                        │
│  [ Active Workspace Steps ] ──> Leftover Build Waste   │
│                                           │            │
│                                           ▼            │
│  [ Automated Cron Cleanup Engine ] ──> Purges Waste    │
│                                           │            │
│                                           ▼            │
│  [ System Storage Disk Layer ] ── Maintain Clean State │
└────────────────────────────────────────────────────────┘

```

---

## Architect's Tips

*Intelligent suggestions from a DevOps Architect with 20+ years of industry experience:*

1. **Treat Pipelines as Products:** Stop writing monolithic YAML files. Abstract complex bash scripts and Docker commands into modular, version-controlled templates (e.g., GitHub Composite Actions or Azure DevOps YAML Templates). This ensures updates are centralized and standardizes compliance across hundreds of developer repositories.
2. **Enforce Immutable Build Environments:** Never install developer dependencies (`Node`, `Java`, `Maven`) directly onto your self-hosted host OS. Instead, mandate that all jobs run inside scoped Docker containers. If the host environment needs to be rebuilt or scaled, you won't suffer from "it works on my machine" host-drift anomalies.
3. **Decouple CI from CD via GitOps:** In the diagrams discussed, the pipeline uses SSH to manually push changes to a target server (Push-based CD). At an enterprise level, stop pushing. Use the CI pipeline strictly to build the container and update a manifest repository. Rely on a GitOps controller like **ArgoCD** or **Flux** running inside the cluster to *pull* the new state automatically, ensuring higher security and automated drift remediation.
