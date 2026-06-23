Here are the comprehensive notes based on the training session and internet research, structured for clarity and scannability.

### To-the-Point Summary

This training session provides a hands-on masterclass on configuring a multi-cloud CI/CD pipeline using **Azure DevOps**. The trainer begins by comparing legacy CI/CD tools like Jenkins with modern, integrated platforms like Azure DevOps, GitLab CI/CD, and GitHub Actions. The core practical demonstration covers the end-to-end architecture: creating an Azure DevOps organization, setting up a project, cloning an Azure Repo locally via SSH, and designing a pipeline. The pipeline's objective is to build a Docker image using a Microsoft-hosted Ubuntu agent, push the image to an **Azure Container Registry (ACR)**, and seamlessly deploy the containerized application onto an **AWS EC2 instance** using an SSH service connection.

---

### Detailed Class Notes

#### 1. The CI/CD Tool Landscape

* **The Shift from Legacy Tools:** Jenkins is slowly deprecating in favor of more robust, cloud-native platforms. Relying solely on Jenkins on a resume limits job prospects.
* **Modern Alternatives:** You must update your resume to include modern CI/CD tools:
* **GitHub Actions**
* **GitLab CI/CD**
* **Azure DevOps**



#### 2. What is Azure DevOps?

Azure DevOps is not just a single CI/CD tool; it is an integrated suite of development tools designed to **Plan, Build, Test, Release, and Deploy** software.

* **Components:** It acts as an orchestrator that manages source code, triggers builds, runs tests, and pushes artifacts to target environments.
* **Multi-Cloud Capability:** Despite being a Microsoft product, Azure DevOps can deploy workloads to **AWS**, **GCP**, or on-premises servers.

#### 3. Multi-Cloud Architecture Workflow

The trainer explained a real-world multi-cloud deployment architecture:

1. **Code:** Developers write code in VS Code and push it to **Azure Repos**.
2. **Build:** An Azure DevOps Pipeline triggers a `docker build` process on a temporary Ubuntu agent.
3. **Push:** The resulting Docker image is pushed to the **Azure Container Registry (ACR)**.
4. **Connect:** The pipeline connects to an **AWS EC2** instance via SSH.
5. **Deploy:** The EC2 instance pulls the image from ACR and runs it as a container (`docker run`).

#### 4. Hands-on Configuration Steps

* **Step 1: Azure DevOps Organization & Project Setup**
* Navigate to `portal.azure.com` and search for "Azure DevOps Organizations".
* **Hierarchy:** `Azure DevOps -> Organization -> Projects`.
* Create a new project (e.g., `veera`) inside your organization.


* **Step 2: Resolving Subscription Limits**
* If you hit a "Maximum number of active/deleted organizations reached" error, use an existing organization instead of creating a new one to avoid billing blocks.


* **Step 3: SSH Key Configuration**
* To push/pull code from Azure Repos, generate an SSH key pair locally.
* Navigate to **User Settings > SSH Public Keys** in Azure DevOps and paste the public key.
* Clone the repository locally using `git clone <ssh-url>`.


* **Step 4: ACR Setup**
* Create an **Azure Container Registry (ACR)** in the Azure Portal to store Docker images securely.


* **Step 5: Target Server (AWS EC2) Setup**
* Launch an AWS EC2 instance (e.g., `t2.medium` Ubuntu). This acts as the permanent target server.


* **Step 6: Service Connections in Azure DevOps**
* Navigate to **Project Settings > Service Connections**.
* Create a **Docker Registry** service connection to authenticate the pipeline with ACR.
* Create an **SSH** service connection (using the EC2 public IP, username, and `.pem` private key) to authenticate the pipeline with the AWS server.



---

### Interview Questions Discussed by Trainer

* **Q: Why are companies migrating away from Jenkins?**
* **A:** Jenkins is becoming a legacy tool, requiring heavy maintenance of plugins and separate servers. Modern tools like Azure DevOps and GitHub Actions offer fully managed, cloud-native CI/CD environments out-of-the-box.


* **Q: Can Azure DevOps deploy resources to AWS or GCP?**
* **A:** Yes. Azure DevOps is an orchestrator. By configuring the correct Service Connections (like SSH or AWS credentials), you can deploy applications to any cloud provider.


* **Q: Where should the `docker build` command run? On the target server or the build agent?**
* **A:** The `docker build` command should run on the temporary **Build Agent** (e.g., Microsoft-hosted Ubuntu). The target server (EC2) should only run the container (`docker pull` and `docker run`).



---

### L2 & L3 Interview Questions (Source: Internet)

*Note: The following advanced questions are frequently asked at top IT firms. Parts marked as **Important** indicate high-frequency topics.*

#### 1. What is the difference between Azure DevOps Services and Azure DevOps Server? (Asked at Microsoft, TCS)

**Answer:** Azure DevOps Services is a cloud-hosted SaaS offering managed by Microsoft, meaning there is no infrastructure to maintain and it is accessible from anywhere. Azure DevOps Server (formerly TFS) is an on-premises version that organizations install and manage on their own internal infrastructure.
**Important:** Most companies today prefer Azure DevOps Services to avoid infrastructure overhead.

| Aspect | Azure DevOps Services | Azure DevOps Server |
| --- | --- | --- |
| **Hosting** | Cloud (Microsoft Managed) | On-Premises (Self-Managed) |
| **Maintenance** | Automatic Updates | Manual Upgrades Required |
| **Access** | Over the Internet | Internal Intranet / VPN |

#### 2. What is the role of Azure Pipelines in Azure DevOps? (Asked at Capgemini, Amazon)

**Answer:** Azure Pipelines is a service that helps automate the build and deployment processes. It supports continuous integration and continuous deployment (CI/CD) to build, test, and deploy code to any platform, ensuring faster and more reliable software releases.

```text
[Source Code in Repos] --> (Trigger) --> [Azure Pipeline]
                                              |
                                              +--> [Build Artifact] --> [Deploy to Cloud]

```

#### 3. What is a build agent in Azure Pipelines? (Asked at TCS, Capgemini)

**Answer:** A build agent is a machine that runs the pipeline jobs. Azure provides Microsoft-hosted agents, which are pre-configured virtual machines that spin up on demand and are discarded after each job. You can also set up self-hosted agents on your own machines or VMs when you need custom software, more control, or better performance.
**Important:** Use self-hosted agents when you need to connect to private networks or require specific, unchangeable caching.

| Agent Type | Maintenance | Cost | Use Case |
| --- | --- | --- | --- |
| **Microsoft-Hosted** | Zero | Pay-as-you-go | Standard builds, SaaS deployments |
| **Self-Hosted** | High | Infrastructure cost | Custom software, Private networks |

#### 4. What is the difference between a Classic pipeline and a YAML pipeline? (Asked at Microsoft, Amazon)

**Answer:** Classic pipelines use a GUI-based editor, making them easy to set up but harder to version control. YAML pipelines define the entire pipeline as code in a `.yml` file stored directly in the repository. They are version-controlled, reviewable via pull requests, and follow infrastructure-as-code principles.

```text
Classic Pipeline: Click & Drag UI ---> Hard to audit changes
YAML Pipeline: Code (.yml file)  ---> Version Controlled in Git

```

#### 5. What is a Service Connection in Azure DevOps? (Asked at Microsoft, Capgemini)

**Answer:** A Service Connection stores credentials that allow Azure Pipelines to connect securely to external services, such as Azure subscriptions, GitHub, Docker Hub, Kubernetes clusters, and external servers via SSH. Instead of hardcoding secrets in pipeline YAML, you reference the service connection by name, and Azure DevOps manages the authentication.

```text
[Azure Pipeline YAML] ---> (Requests Access) ---> [Service Connection]
                                                        |
                                                        V
                                              [External Service: AWS / ACR]

```

#### 6. What are Azure Artifacts? (Asked at TCS)

**Answer:** Azure Artifacts is a package management service that lets teams create, host, and share packages, including npm, NuGet, Maven, Python, and Universal packages. It supports upstream sources so you can proxy public registries and use a single feed for both internal and external packages, integrating directly with Azure Pipelines.

| Internal Pipeline |  | Azure Artifacts Feed |  | External Developers |
| --- | --- | --- | --- | --- |
| Publishes `.nupkg` | ---> | Stores Securely | <--- | Consumes Package |

#### 7. What is a Pull Request in Azure DevOps? (Asked at Amazon)

**Answer:** A Pull Request (PR) in Azure DevOps is a method used to review and merge code changes in a Git repository. When developers complete their code changes, they create a PR to notify others. Team members can then review the code, suggest changes, and approve them before merging into the main codebase.

```text
[Feature Branch] ---> (Create PR) ---> [Code Review / Build Validation] ---> (Merge) ---> [Main Branch]

```

#### 8. What is Infrastructure as Code (IaC) in DevOps? (Asked at Capgemini)

**Answer:** Infrastructure as Code (IaC) is a key DevOps practice that involves managing and provisioning computing infrastructure through machine-readable scripts and configuration files, rather than through physical hardware configuration or interactive configuration tools. This allows for consistent and repeatable setups, reducing errors and speeding up deployments.

| Traditional Ops | Infrastructure as Code |
| --- | --- |
| Manual Portal Clicks | Code Execution (Terraform/ARM) |
| High Human Error | Repeatable & Automated |
| Slow Provisioning | Instant Provisioning |

#### 9. Can you define continuous integration and continuous deployment (CI/CD)? (Asked at TCS)

**Answer:** Continuous Integration (CI) is a practice where developers integrate their code changes into a shared repository frequently. These changes are automatically verified by running tests and building the project. Continuous deployment (CD) automatically deploys all validated code changes to a test or production environment after the build stage, ensuring the codebase is always deployable.

```text
[Code Push] --> [CI: Build & Test] --> [Artifact] --> [CD: Deploy to Staging] --> [CD: Deploy to Prod]

```

#### 10. What are Azure Boards? (Asked at Microsoft)

**Answer:** Azure Boards is a component of the Azure DevOps suite used for managing projects and software development. Essential features include reporting, dashboards, project planning, and bug tracking. These tools allow cross-functional Agile teams to manage user stories, sprints, and tasks effectively.
**Important:** Azure Boards integrate seamlessly with Repos and Pipelines, allowing a developer to trace a deployed feature all the way back to the initial user story.

| Board Component | Purpose |
| --- | --- |
| **Work Items** | Track bugs, tasks, and features |
| **Backlogs** | Prioritize work for upcoming sprints |
| **Sprints** | Manage work assigned to a specific iteration |

---

### Tips from a DevOps Architect

As an architect with over 20 years of experience, here are my top 3 strategies for mastering Azure DevOps:

1. **Never Hardcode Secrets:** Always use **Azure Key Vault** integrated with Azure DevOps Library Variable Groups. Rely on Service Connections to authenticate with external clouds (like AWS or GCP) rather than placing raw credentials inside your YAML files.
2. **Decouple Build and Deploy:** Treat your CI (Build) and CD (Deploy) as two separate, modular entities. Your CI pipeline should strictly focus on generating an immutable artifact (like a Docker Image), while your CD pipeline should handle the specific deployment logic. This allows you to roll back easily without rebuilding the code.
3. **Adopt Ephemeral Build Agents:** Rely heavily on Microsoft-hosted (ephemeral) agents for standard builds. They guarantee a clean environment, preventing "it works on my machine" issues. Reserve self-hosted agents strictly for tasks that require access to isolated corporate networks.

---

Do you have any specific questions about configuring the SSH Service Connection or the YAML pipeline syntax discussed in the session?
