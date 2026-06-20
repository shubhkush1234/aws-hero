# Terraform and Azure Architecture Mock Interview - Part 1

Here is the breakdown of your mock interview session, including the topics covered, the questions asked, and interview-ready answers with diagrams that you can use in your actual interviews.

---

## 1. Interview Topics Covered

* **Terraform Fundamentals:** Data Blocks
* **Terraform Meta-arguments:** `count` vs `for_each`
* **Terraform Data Types:** `list` vs `map`
* **Terraform Core Commands:** `terraform init` process
* **Azure High Availability:** Redundancy concepts
* **Azure Storage Accounts:** Required arguments and Replication Types (LRS, ZRS, GRS, GZRS)
* **Azure Governance:** Azure Policy
* **Azure Resource Management:** Scope Hierarchy
* **Azure Networking:** Azure Front Door (Layer 7 Load Balancing)
* **Azure Architecture:** Hub and Spoke Network Topology

---

## 2. Interview Questions Asked

1. In which scenario we have to use data block?
2. Tell me the difference between count and for_each.
3. What is the difference between list and map?
4. When we do `terraform init`, what happens?
5. What is redundancy?
6. In a storage account, what are the required arguments?
7. What are the application replication types?
8. Do you know the difference between GRS and ZRS?
9. What is Azure policy?
10. Can you explain the Microsoft hierarchy (Scope hierarchy)?
11. What is Azure Front Door?
12. Can you explain the Hub and Spoke model?

---

## 3. Interview-Ready Answers and Diagrams

### Q1: In which scenario we have to use a data block?

**Answer:**
We use a data block in Terraform when we need to fetch information about resources that are created and managed outside of our current Terraform configuration. For example, if a virtual network or a resource group was created manually or by another team's Terraform code, we can use a data block to query its ID and use it in our current deployment without managing the resource itself.

**Diagram to Draw:**

```text
[ External Azure Portal ]
        | (Managed manually)
        v
+-----------------------+
|  Existing Resource    |
|  (e.g., VNet: VNet-A) |
+-----------------------+
        ^
        | (Fetches ID/Info)
+-----------------------+
| Terraform Data Block  |
| data "azurerm_vnet"   |
+-----------------------+
        | (Passes VNet ID)
        v
+-----------------------+
| Terraform Resource    |
| (e.g., Subnet)        |
+-----------------------+

```

---

### Q2: Tell me the difference between count and for_each.

**Answer:**
Both `count` and `for_each` are meta-arguments used to create multiple instances of a resource, but they handle identification differently.
`count` uses an integer index (0, 1, 2) to identify each resource. This can cause issues if you remove an item from the middle of the list, as Terraform will recreate all subsequent resources due to the index shift.
`for_each` uses a map or a set of strings, assigning a unique string key to each resource instance. This ensures that adding or removing an item dynamically only affects that specific resource, without impacting the rest, making it much safer for dynamic deployments across multiple regions or configurations.

**Diagram to Draw:**

```text
COUNT (Index-Based)                  FOR_EACH (Key-Based)
-------------------                  --------------------
Resource[0] : VM-A                   Resource["Web"]: VM-A
Resource[1] : VM-B                   Resource["App"]: VM-B
Resource[2] : VM-C                   Resource["DB"] : VM-C

*If VM-B is removed:*                *If VM-B is removed:*
Resource[0] : VM-A (Safe)            Resource["Web"]: VM-A (Safe)
Resource[1] : VM-C (RECREATED!)      Resource["DB"] : VM-C (Safe)

```

---

### Q3: What is the difference between list and map?

**Answer:**
A `list` is an ordered collection of values of the same data type. Elements in a list are accessed using a zero-based index. Lists can contain duplicate values.
A `map` is an unordered collection of key-value pairs. In a map, the keys are strings and must be unique, while the values can be of any data type. Elements are accessed via their specific string key rather than a numerical index.

**Diagram to Draw:**

```text
DATA TYPE: LIST                      DATA TYPE: MAP
---------------                      --------------
Index | Value                        Key      | Value
----------------                     ----------------
  0   | "EastUS"                     "Region" | "EastUS"
  1   | "WestUS"                     "Env"    | "Prod"
  2   | "CentralUS"                  "Tier"   | "Standard"

*Accessed via Index:                 *Accessed via Key:
 var.locations[1] -> "WestUS"         var.config["Env"] -> "Prod"

```

---

### Q4: When we do terraform init, what happens?

**Answer:**
Running `terraform init` initializes the working directory containing your Terraform configuration files. It performs three primary tasks:
First, it initializes the backend where your state file will be stored.
Second, it downloads and installs any child modules referenced in your code.
Third, it downloads the necessary provider plugins (like the Azure or AWS providers) into the `.terraform` hidden directory and creates a lock file (`.terraform.lock.hcl`) to lock the provider versions.

**Diagram to Draw:**

```text
       [ terraform init ]
               |
 +-------------+-------------+
 |             |             |
 v             v             v
[Backend]  [Modules]    [Providers]
 Setup      Download     Download
 State      Child        Plugins
 Storage    Modules      (.exe files)
               |
               v
     +-------------------+
     | .terraform folder |
     | .terraform.lock   |
     +-------------------+

```

---

### Q5: What is redundancy?

**Answer:**
Redundancy is the architectural practice of duplicating critical infrastructure components or data across multiple fault domains. The goal is to eliminate single points of failure. For instance, if an application runs on a single Virtual Machine and that VM fails, the application goes down. By placing redundant copies of the VM across different servers, zones, or regions, the system can automatically failover to a healthy instance, ensuring continuous availability and reliability.

**Diagram to Draw:**

```text
         [ Incoming Traffic ]
                  |
         [ Load Balancer ]
          /               \
   (Primary)             (Secondary - Redundant)
+-------------+       +-------------+
|    VM-01    |       |    VM-02    |
| (Active)    |       | (Standby)   |
+-------------+       +-------------+
       \                 /
     [ Shared Database ]

```

---

### Q6: In a storage account, what are the required arguments?

**Answer:**
When provisioning an Azure Storage Account via Terraform (`azurerm_storage_account`), the mandatory arguments are:
`name`: Must be globally unique, lowercase, and numbers only.
`resource_group_name`: The logical container for the storage account.
`location`: The Azure region.
`account_tier`: Specifies the performance tier (Standard or Premium).
`account_replication_type`: Specifies the redundancy level (LRS, GRS, ZRS, etc.).

**Diagram to Draw:**

```text
resource "azurerm_storage_account" "example" {
  name                     = "mystorageacct123"      <-- Required (Unique)
  resource_group_name      = "my-rg"                 <-- Required
  location                 = "eastus"                <-- Required
  account_tier             = "Standard"              <-- Required
  account_replication_type = "LRS"                   <-- Required
}

```

---

### Q7: What are the application replication types?

**Answer:**
Azure Storage offers four primary replication types to ensure high availability:

* **LRS (Locally Redundant Storage):** 3 synchronous copies within a single physical data center.
* **ZRS (Zone-Redundant Storage):** 3 synchronous copies spread across three distinct availability zones within a single region.
* **GRS (Geo-Redundant Storage):** 3 synchronous copies in the primary region (like LRS), plus asynchronous replication to a paired secondary region (another 3 copies).
* **GZRS (Geo-Zone-Redundant Storage):** 3 synchronous copies across availability zones (like ZRS), plus asynchronous replication to a secondary region.

**Diagram to Draw:**

```text
Replication | Copies | Location Spread
--------------------------------------------------
LRS         |   3    | Single Data Center
ZRS         |   3    | 3 Availability Zones (1 Region)
GRS         |   6    | Primary Region (LRS) + Secondary Region (LRS)
GZRS        |   6    | Primary Region (ZRS) + Secondary Region (LRS)

```

---

### Q8: Do you know the difference between GRS and ZRS?

**Answer:**
The primary difference lies in how they protect against different types of failures.
**ZRS (Zone-Redundant Storage)** replicates your data across three separate availability zones within the *same* primary region. It protects your data from a complete data center failure (like power or cooling loss).
**GRS (Geo-Redundant Storage)** replicates your data three times within a single data center in the primary region, but then asynchronously copies it to a completely different *secondary region* hundreds of miles away. GRS protects your data against massive regional disasters (like earthquakes or floods) that could take down an entire region.

**Diagram to Draw:**

```text
        ZRS (Zone Redundant)                 GRS (Geo Redundant)
+---------------------------------+    +-----------------+    +-----------------+
|         REGION: East US         |    | REGION: East US |    | REGION: West US |
|                                 |    |                 |    |                 |
|  [Zone 1]  [Zone 2]  [Zone 3]   |    |   [Data Center] |===>|   [Data Center] |
|   Copy 1    Copy 2    Copy 3    |    |    (3 Copies)   |Sync|    (3 Copies)   |
+---------------------------------+    +-----------------+    +-----------------+
  (Protects against DC failure)          (Protects against Regional failure)

```

---

### Q9: What is Azure policy?

**Answer:**
Azure Policy is a governance tool provided by Microsoft to enforce organizational standards and assess compliance across your cloud environment. It uses a set of rules (in JSON format) to control what can and cannot be created. For example, you can create a policy that denies the creation of virtual machines unless they are in the "East US" region, or a policy that requires all resources to have a "CostCenter" tag.

**Diagram to Draw:**

```text
[ Azure User / CI/CD Pipeline ]
           | (Deployment Request)
           v
+-------------------------+
|     Azure Policy        | <--- Evaluates against rules 
| (e.g., Allow only VMs   |      (Allowed Regions, Tags)
|  in East US)            |
+-------------------------+
       /           \
  [PASS]          [FAIL]
    |               |
    v               v
 Resource       Deployment
 Created          Denied

```

---

### Q10: Can you explain the Microsoft hierarchy (Scope hierarchy)?

**Answer:**
The Azure Scope Hierarchy defines how resources are organized and how permissions (RBAC) and policies are inherited. It consists of four levels from top to bottom:

1. **Management Groups:** Used to organize multiple subscriptions and apply governance globally.
2. **Subscriptions:** The billing and deployment boundary.
3. **Resource Groups:** Logical containers used to group related resources sharing the same lifecycle.
4. **Resources:** The actual deployed services, like Virtual Machines, Storage Accounts, and VNets.
Policies applied at a higher level automatically flow down to all levels below it.

**Diagram to Draw:**

```text
      [ Management Group (Root) ]
               |
      [ Management Group (IT) ]
               |
         [ Subscription ]
          /            \
[Resource Group A]   [Resource Group B]
       |                     |
   [ VM, VNet ]           [ SQL, WebApp ]

```

---

### Q11: What is Azure Front Door?

**Answer:**
Azure Front Door is a global, Layer 7 (HTTP/HTTPS) load balancer and Content Delivery Network (CDN) service. Because it operates at Layer 7, it is designed specifically for web applications. It uses Microsoft's global edge network to route user traffic to the fastest and most available backend pool (like a web app in East US vs Europe). It also provides enterprise-grade security features like Web Application Firewall (WAF) and SSL termination.

**Diagram to Draw:**

```text
                   [ Users Globally ]
                           |
                           v
              +-------------------------+
              |   Azure Front Door      | (Global Edge Network)
              | (Layer 7 Routing / WAF) |
              +-------------------------+
                  /                   \
         (Path based routing)    (Latency based routing)
                /                       \
   +-------------------+          +-------------------+
   | Web App (East US) |          | Web App (West EU) |
   +-------------------+          +-------------------+

```

---

### Q12: Can you explain the Hub and Spoke model?

**Answer:**
The Hub and Spoke model is a network topology where a central virtual network (the Hub) connects to multiple isolated virtual networks (the Spokes) via VNet Peering.
The **Hub** acts as a central point of connectivity and security. It usually contains shared services like Azure Firewall, DNS servers, and the VPN or ExpressRoute Gateway that connects to on-premises data centers.
The **Spokes** host the actual application workloads (like web servers or databases). They don't have their own gateways; instead, they route their traffic through the Hub, which centralizes traffic inspection and reduces management overhead.

**Diagram to Draw:**

```text
                     [ On-Premises Network ]
                               |
                        (VPN / ExpressRoute)
                               |
+-------------------------------------------------------------+
|                       HUB VNET                              |
|  [ VPN Gateway ] ---> [ Azure Firewall ] ---> [ Shared AD ] |
+-------------------------------------------------------------+
           | (VNet Peering)                 | (VNet Peering)
           v                                v
+-----------------------+         +-----------------------+
|     SPOKE VNET 1      |         |     SPOKE VNET 2      |
|  [ Web Application ]  |         |   [ Database Tier ]   |
+-----------------------+         +-----------------------+

```
