# Ultimate Azure Governance & Identity Management Study Notes

## Comprehensive Revision Masterclass Notes

---

### To-the-Point Summary

This masterclass provides a deep architectural review of the **Microsoft Azure Governance Hierarchy, Identity Operations via Microsoft Entra ID (formerly Azure Active Directory), Role-Based Access Control (RBAC)**, and their direct translation into **Infrastructure as Code (IaC) via Terraform**.

The core learning objectives emphasize that production-grade cloud management depends on structural boundaries: **Management Groups** control enterprise policy across multiple **Subscriptions**, which function as billing and resource limits; **Resource Groups** aggregate entities sharing a unified operational lifecycle; and **Resources** represent the deployed infrastructure.

Identity management is explored via single and bulk onboarding techniques using Microsoft Entra ID templates. Access management is framed through **RBAC**, which relies on three variables: **Security Principal** (Who), **Role Definition** (What), and **Scope** (Where). Finally, these manual configurations are mapped to declarative configurations in **Terraform**, demonstrating how graphical UI fields map to code arguments (`resource`, `name`, `location`).

---

### Section 1: Detailed Class Notes & Structural Step-by-Step Workflows

#### 1. The Azure Enterprise Governance Hierarchy

Managing an enterprise cloud footprint requires a structured, multi-tier organizational model to enforce security, compliance, and budget parameters. Azure organizes all entities into a mandatory four-layer hierarchy:

```
[Tenant Root Group (Root Management Group)]
                    │
         ┌──────────┴──────────┐
   [Sales Group]         [HR Group]   <-- Management Groups (Policy/Governance)
         │                     │
   [Subscription 1]      [Subscription 2] <-- Subscriptions (Billing/Access Boundary)
         │                     │
   [Resource Group A]    [Resource Group B] <-- Resource Groups (Lifecycle Boundary)
         │                     │
    [Virtual Machine]     [Storage Account] <-- Resources (Actual Infrastructure)

```

* **Level 1: Management Groups**: Logical containers used to govern configurations across multiple subscriptions. Policies and compliance guardrails defined here automatically inherit downward.
* **Level 2: Subscriptions**: Primary billing and access control boundaries. Every subscription is linked to a specific payment instrument and an Entra ID Tenant.
* **Level 3: Resource Groups**: Logical folders that aggregate resources sharing the same lifecycle. Deleting a resource group cleans up all child resources instantly.
* **Level 4: Resources**: Individual service instances (Virtual Machines, Databases, VNets) deployed inside a chosen Resource Group and region.

---

#### 2. Microsoft Entra ID (Azure AD) Deep Dive

Microsoft Entra ID functions as the central identity, authentication, and directory service environment for the Azure Cloud.

##### Core Tenant Identifiers & Properties

* **Tenant ID**: A globally unique 128-bit Guid (`4f7e73f5-2df7-4c56-918b-8f79e3fd6708`) that isolates your organization's directory data.
* **Primary Domain**: The fallback internal DNS identity formatted as `[tenant-name].onmicrosoft.com` (e.g., `sssce2007gmail.onmicrosoft.com`).
* **License Layer**: Free tiers handle standard identity management, while Premium (`P1/P2`) layers unlock Conditional Access and Self-Service Password Reset (SSPR).

##### User Management & Provisioning Matrix

The trainer demonstrated two separate methods for provisioning directory users within the portal interface:

##### Method A: Single User Provisioning (Manual Click-Ops)

1. Navigate to **Microsoft Entra ID** -> **Users** -> **All Users**.
2. Click **New User** -> Select **Create new user** or **Invite external user**.
3. Define the User Principal Name (UPN) (e.g., `saathae21@18g24.online`), display name, password profiles, and property parameters.
4. Finalize creation to commit the identity object to the tenant directory cache.

##### Method B: Automated Bulk Operations via CSV Provisioning

To onboard dozens or thousands of directory objects simultaneously, manual data entry becomes inefficient. The portal provides dedicated workflows to automate this:

```
[Download CSV Template from Portal] 
               │
               ▼
[Populate Fields: Name, UPN, Password] 
               │
               ▼
[Upload File to Portal via Bulk Create] 
               │
               ▼
[Entra ID Batch Processor Validates Logs]

```

1. Navigate to **Users** -> **All Users** -> Select **Bulk operations** -> Click **Bulk create**.
2. Click **Download** to obtain the official `UserCreateTemplate.csv` structural matrix file.
3. Open the file in an editor (like Excel or Visual Studio Code) and populate the required system parameters:
```csv
version:v1.0
Name,User principal name,Initial password,Block sign in (Yes/No),First name,Last name,Job title,Department
Aastha Sharma,saathae21@18g24.online,Password123!,No,Aastha,Sharma,DevOps,Engineering
Ajay Singh,ajay.devops@18g24.online,Password123!,No,Ajay,Singh,Architect,Operations

```


4. Save and upload the finalized CSV dataset to the portal interface.
5. The Entra ID batch processor validates line elements, loops through records, and creates individual identity objects asynchronously.

---

#### 3. Access Control Architecture: Azure RBAC

**Important: Asked in multiple cloud security and operations reviews.** Azure Role-Based Access Control (RBAC) enforces fine-grained authorization boundaries across cloud infrastructures. Authorization is determined by evaluating three primary variables:

$$\text{Role Assignment} = \text{Security Principal (Who)} + \text{Role Definition (What)} + \text{Scope (Where)}$$

##### The Three Core Pillars of RBAC

* **1. Security Principal (Who):** The identity requesting access. This can be a User, a Group, a Service Principal (automated application accounts), or a Managed Identity.
* **2. Role Definition (What):** A collection of permissions outlining allowed actions (e.g., `Microsoft.Compute/virtualMachines/write`). Predefined structural definitions include:
* **Owner:** Full administrative authority, including the right to assign roles and manage access for others.
* **Contributor:** Full authority to create, configure, and manage cloud infrastructure resources, but **cannot** delegate permissions to others or manage access configurations.
* **Reader:** Read-only access to view resources without permission to modify or delete anything.


* **3. Scope (Where):** The specific structural layer where the access configuration is active. This can be applied at the Management Group, Subscription, Resource Group, or individual Resource level.

---

#### 4. Mapping Graphical Portal Inputs to Infrastructure as Code (Terraform)

The masterclass highlighted a key architectural lesson: every manual configuration field required in the Azure Graphical User Interface (GUI) corresponds to a code argument inside an Infrastructure as Code (IaC) configuration file.

When creating a Resource Group or a Virtual Machine via the portal, fields highlighted with red asterisks are mandatory values. These properties translate into required arguments within a declarative Terraform configuration file:

```hcl
# Automated Terraform Declarative Block representing a Resource Group environment
# Source: Internet

resource "azurerm_resource_group" "production_environment" {
  name     = "DemoRGVirtualClass"  # Maps exactly to 'Resource Group Name' in Portal Form
  location = "East US"            # Maps exactly to 'Region' Dropdown Selection in Portal Form

  tags = {
    Environment = "MasterclassRevision"
    Architect   = "DevOpsInsiders"
  }
}

```

By standardizing these configurations into code repositories, organizations can shift away from manual "Click-Ops" toward version-controlled automation. This allows team members to deploy identical setups safely and repeatedly.

---

### Section 2: Interview Questions Section (Trainer Discussed)

1. What is the fundamental difference between an Azure Owner and an Azure Contributor role definition?
2. Explain the structural hierarchy used to organize environments in Microsoft Azure.
3. How do you provision multiple corporate user directory accounts simultaneously within Microsoft Entra ID?
4. What are the mandatory parameters required to execute a Role Assignment inside Azure access control?
5. How do input properties selected in the Azure Portal interface map to declarative arguments within Terraform files?

---

### Section 3: Industry Interview Questions (Top Tech Companies Sourced)

#### 1. How does Azure RBAC evaluate permissions when multiple roles overlap, and how do Deny Assignments interact with this?

* **Asked At:** Microsoft, Accenture
* **Answer:** Azure RBAC uses an additive allow model. If a user is assigned multiple roles across overlapping scopes, their effective permissions are the union of those assignments. For example, if a user has **Reader** access at the Subscription level but is granted **Contributor** access inside a specific Resource Group, they hold full contributor rights within that group. However, **Deny Assignments** are a special override mechanism used by system processes (such as Azure Blueprints or Azure Deployment Environments). They take absolute precedence over allow rules, blocking specified identities from performing actions even if an RBAC role explicitly permits it.

#### 2. What is the difference between an Azure Active Directory (Entra ID) Role and an Azure RBAC Role?

* **Asked At:** Capgemini, Cognizant
* **Answer:** Azure Entra ID roles manage identity lifecycle, directory-level configurations, and global tenant management tasks (e.g., Global Administrator, User Administrator, Application Developer). These roles apply at the tenant level. Azure RBAC roles manage access to cloud infrastructure resources (e.g., Owner, Contributor, Reader applied to Subscriptions, Resource Groups, or specific resources). An Entra ID User Administrator cannot manage a virtual machine unless explicitly granted an RBAC role on that subscription or resource group.

#### 3. What are Management Groups in Azure, and what are their architectural scale limits?

* **Important - Asked in multiple enterprise scale design interviews**
* **Asked At**: Microsoft, Capgemini, TCS
* Answer: Management Groups are containers that help manage access, policy, and compliance across multiple subscriptions. In large enterprises, they provide a framework to enforce governance standards. Management groups support a tree structure up to six levels of depth (excluding the root node and the resource level). All subscriptions within a Management Group automatically inherit the conditions, Azure Policies, and RBAC role assignments defined at the parent level.

#### 4. How does Azure AD Connect (or Cloud Sync) sync on-premises identities with Microsoft Entra ID?

* **Asked At**: Wipro, Infosys
* Answer: Azure AD Connect synchronizes identities from on-premises Active Directory Domain Services (AD DS) to cloud-based Microsoft Entra ID. It hashes the password hash already stored on-premises an additional 2 minutes over 1000 iterations before writing it securely to the cloud directory database. Alternatively, Microsoft recommends **Azure AD Connect Cloud Sync** for hybrid deployments. It moves the processing load from a heavy local server to the Azure cloud using a lightweight agent, simplifying management across multi-forest environments.

#### 5. What are Managed Identities, and why are they preferred over Azure Service Principals for app authentication?

* **Important - Multiple Security reviews**
* **Asked At**: Accenture, Cognizant, Microsoft
* Answer: Managed Identities provide an automated identity in Microsoft Entra ID for applications to use when connecting to resources that support Entra ID authentication. They eliminate the security risks of traditional Service Principals because developers **never** have to manage credentials, connection secrets, or certificate rotation cycles. There are two types:
* **System-Assigned Managed Identity:** Tied directly to the lifecycle of a specific resource instance. If the resource is deleted, the identity is automatically cleaned up.
* **User-Assigned Managed Identity:** Created as a standalone Azure resource that can be shared across multiple infrastructure components.



#### 6. How can you prevent critical cloud resource architectures from being accidentally modified or deleted?

* **Asked At**: TCS, Infosys, Wipro
* Answer: Administrators can apply **Azure Resource Locks** at the Subscription, Resource Group, or individual Resource layer. These locks override any user permissions granted through RBAC roles. There are two types of resource locks:
* **CanNotDelete (Delete Lock):** Authorized users can read and modify a resource, but they cannot permanently delete it.
* **ReadOnly (Read-Only Lock):** Users can view resources but cannot modify or delete them. This lock effectively downgrades even an administrator's permissions to matching a Reader role.



#### 7. What is the process for performing bulk user provisioning in Microsoft Entra ID using automation scripts instead of the Portal GUI?

* **Asked At**: Cognizant, Capgemini
* Answer: For programmatic bulk provisioning, engineers bypass the portal and upload structural configurations using automated **Azure PowerShell** modules or the **Microsoft Graph CLI/API**. You can parse a standard identity data file (such as a CSV or JSON payload) and loop through the records to generate new Entra ID users programmatically, as shown below:
```powershell
# PowerShell script to automate bulk Entra ID user provisioning
# Source: Internet
Import-Csv -Path "C:\Data\UserOnboardingList.csv" | ForEach-Object {
    $PasswordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
    $PasswordProfile.Password = $_.InitialPassword
    New-AzureADUser -DisplayName $_.Name -UserPrincipalName $_.UserPrincipalName -MailNickName $_.FirstName -PasswordProfile $PasswordProfile -AccountEnabled $true
}

```



#### 8. How do Azure Policies differ from Azure RBAC roles, and how do they work together to secure resources?

* **Important - Asked across cloud security governance architectures**
* **Asked At**: Microsoft, Accenture
* Answer: The distinction is centered on the difference between operational operations and property parameters: **RBAC controls who can take actions on resources**, focusing on user capabilities across scopes. **Azure Policy controls resource properties and structural configurations**, focusing on resource standards regardless of who initiates the deployment. For example, RBAC defines whether a user can create a virtual machine, while an Azure Policy ensures that the virtual machine uses approved sizes (such as `Skus`) or requires specific tagging standards before it can be deployed.

#### 9. What is Azure Root Management Group, and why must you use caution when modifying it?

* **Asked At**: Accenture, TCS
* Answer: The **Tenant Root Group** is the top-level container in the management hierarchy. Every management group and subscription in the tenant rolls up to this root node. Any access configurations, role assignments, or compliance policies applied here are inherited by **every resource** across the entire cloud footprint. Modifying access rules at the root group can accidentally lock out users or enforce strict compliance restrictions tenant-wide, which can disrupt automated deployment pipelines and running applications.

#### 10. How do you resolve a "Validation Failed" error when deploying resources via an ARM Template or Terraform?

* **Asked At**: Infosys, Wipro, Cognizant
* Answer: A "Validation Failed" error indicates that the management plane rejected the deployment request before provisioning resources. This usually happens because mandatory parameters are missing, or the request violates corporate compliance rules (such as an Azure Policy restricting regions or resource sizes). To fix this, look at the error details in the Azure portal activity logs or review the output from your deployment tool. Ensure all required fields match template specifications and comply with target subscription policy rules.

---

### Section 4: DevOps Architect Pro-Tips & Strategic Design Principles

*From a DevOps Architect with 20+ years of cloud infrastructure experience:*

1. **Enforce the Principle of Least Privilege (PoLP) and Minimize Direct Subscription Owner Assignments**
Never grant global **Owner** roles to team members for daily infrastructure management. Instead, use custom **RBAC Role Definitions** targeted at specific **Resource Group Scopes**. This approach ensures team members have the exact access required for their tasks, protecting critical billing assets and subscription settings from accidental configuration errors.
2. **Standardize Infrastructure Deployment using Declarative Pipelines**
Avoid creating cloud resources manually through the Portal user interface. Instead, export infrastructure requirements directly into declarative code using **Terraform configurations**. Managing your cloud environment through version-controlled source code repositories allows you to establish reliable peer review processes, track tracking logs over time, and build automated deployment pipelines that can reproduce environments across different environments.
3. **Incorporate Multi-Tier Governance and Resource Locks in Production Environments**
Do not rely entirely on user permission rules to protect your most critical infrastructure assets. Apply **CanNotDelete Resource Locks** directly to foundational production components, such as your core network zones, master database clusters, and shared enterprise resource groups. This safety standard adds an explicit verification step before critical services can be modified or deleted, helping protect your infrastructure from automation runtime errors or human mistakes.
