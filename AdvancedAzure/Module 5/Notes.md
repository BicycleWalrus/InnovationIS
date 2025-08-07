### **Slide 1: Storage**

#### Speaker Notes:

**Opening Talking Points:**

* "Let’s kick off our discussion on Azure Storage—the backbone of nearly every cloud workload."
* "Whether you’re storing application logs, massive datasets, or user-uploaded files, Azure has a storage option that fits the need."

**Real-World Context:**

* "From simple file shares to globally distributed object stores, Azure provides scalable and secure storage services that integrate with compute, analytics, and more."
* "Storage is usually one of the first things provisioned in cloud architecture—it’s also where many cost and performance challenges emerge."

**Instructor Cues:**

* Ask: *“What kind of storage services have you used in Azure—Blob, File, or something else?”*
* Set expectations: *“By the end of this module, you’ll know when to use Blob vs Table vs File vs Queue, and how to deploy lifecycle-managed storage via Bicep.”*

---

### **Slide 2: Azure Storage Accounts**

#### Speaker Notes:

**Talking Points:**

* "Azure Storage Accounts are containers for storing **data objects**—they come in several service flavors, each with its own use case."
* "There are four primary types: Blob, File, Table, and Queue. Let’s walk through when to use each."

**Breakdown of Services:**

* **Blob Storage** – Object storage for unstructured data. Think images, videos, backups, or any binary data. It supports different storage tiers (Hot/Cool/Archive).
* **File Storage** – Fully managed SMB file shares you can mount like a network drive. Great for lift-and-shift or hybrid workloads using Azure File Sync.
* **Table Storage** – A NoSQL key-value store for massive, schema-less datasets. Azure Cosmos DB builds on this for global distribution.
* **Queue Storage** – Lightweight message queuing for async communication between distributed systems or microservices.

**Real-World Context:**

* "If you’re building a web app and need to store user uploads—Blob. If you’re syncing file shares to Azure from on-prem—File. Need durable, scalable messaging between services—Queue."

**Instructor Cues:**

* Ask: *“Which of these have you used in production? What were the use cases?”*
* Tie-in: *“Keep this breakdown in mind—we’ll reference it when designing lifecycle rules and Bicep deployments later.”*

---

### **Slide 3: Azure Storage Account Endpoints**

#### Speaker Notes:

**Talking Points:**

* "Each Azure Storage account exposes **REST endpoints** for each supported service type."
* "These endpoints are domain-based URLs that follow a consistent format and are service-specific."

**Endpoint Format Examples:**

* Blob: `https://<account>.blob.core.windows.net`
* File: `https://<account>.file.core.windows.net`
* Table: `https://<account>.table.core.windows.net`
* Queue: `https://<account>.queue.core.windows.net`

**Real-World Context:**

* "These endpoints are what clients, apps, SDKs, or the CLI hit when interacting with storage."
* "If you’re troubleshooting access or writing code against a storage API, this is what matters under the hood."

**Instructor Cues:**

* Ask: *“Has anyone had to troubleshoot a failed storage access using a REST endpoint or SAS token?”*
* Mention: *“Later, we’ll see how Bicep outputs and keys help us interact with these URLs programmatically.”*

---

### **Slide 4: Azure Storage Custom Domain Name**

#### Speaker Notes:

**Talking Points:**

* "By default, Azure uses Microsoft-owned domains for storage endpoints—but you can use a **custom domain** for your storage traffic."
* "You do this via Azure Front Door or Azure CDN to map your storage endpoint to a branded DNS name."

**Real-World Context:**

* "This is especially useful for customer-facing content or apps—where `cdn.myapp.com` looks better than `mystorage.blob.core.windows.net`."
* "It also gives you more control over caching, performance, and even WAF integration."

**Instructor Cues:**

* Ask: *“Have you used custom domains or Azure Front Door before?”*
* Tip: *“If performance and branding matter, custom endpoints are a low-effort win.”*
* Transition: *“Now that we understand how we access storage, let’s explore the different **Blob types** you’ll encounter.”*

---

### **Slide 5: Azure Blob Storage Types**

#### Speaker Notes:

**Talking Points:**

* "Azure Blob Storage comes in three types—each designed for different usage patterns and performance needs."

**Type Breakdown:**

* **Block Blob** – Default for most uploads. Max 190.7 TB. Supports insert/replace/delete of blocks. Great for large, unstructured objects like backups, media files.
* **Append Blob** – Optimized for append-only operations. Max \~195 GB. Best for logging scenarios—new data is added to the end.
* **Page Blob** – Used for random read/write access. Max 8 TB. Ideal for VHDs and VM disks. Backed by standard/premium storage.

**Real-World Context:**

* "If you’re writing logs—go Append Blob. If you’re storing ISO images or massive files—Block Blob. If it’s a disk, use Page Blob."

**Instructor Cues:**

* Ask: *“Which Blob type have you used—or did you let Azure pick the default?”*
* Reinforce: *“This choice matters for performance and cost. Especially in scenarios like diagnostics, backups, or VM provisioning.”*

---

### **Slide 6: Use Cases**

#### Speaker Notes:

**Talking Points:**

* "Azure Blob Storage supports a huge variety of real-world use cases—and it’s critical to choose the right tier and configuration based on workload."

**Use Case Categories:**

* **Big Data & Analytics** – Used for ADLS Gen2 (Data Lake), ML pipelines, ingest and output stages.
* **Cloud-Native Workloads** – Store static web assets, logs, or unstructured data from microservices.
* **AI/ML & HPC** – Used for massive training datasets, scratch storage, or inference results.
* **Long-Term & Archival** – Use Cool and Archive tiers for compliance data, backups, or any infrequently accessed content.

**Real-World Context:**

* "Think about a data science pipeline: ingest data from blob, process it in a compute cluster, and write results back to blob. Same for backups—store infrequent data in Archive at a fraction of the cost."

**Instructor Cues:**

* Ask: *“Which of these patterns are most relevant to your teams or projects?”*
* Bridge to next: *“With the use cases in mind, let’s now shift focus to storage **tiers** and how lifecycle policies help automate transitions between them.”*

---

### **Slide 7: BLOB STORAGE tiers**

#### Speaker Notes:

**Talking Points:**

* "Azure Blob Storage supports **three access tiers**—Hot, Cool, and Archive. Each is optimized for a specific access pattern and cost profile."
* "Choosing the right tier isn’t just a cost optimization—it can significantly affect performance, billing, and lifecycle automation."

**Tier Breakdown:**

* **Hot Tier** – Optimized for frequent access. Lowest transaction costs, highest storage cost. Great for active files and operational workloads.
* **Cool Tier** – Ideal for infrequent access. Lower storage cost than Hot, but higher transaction cost. Must retain data for at least 30 days.
* **Archive Tier** – Cheapest storage, but retrieval can take hours. Minimum retention of 180 days. Used for compliance, long-term retention.

**Real-World Context:**

* "Let’s say you’re running monthly billing reports. You need to keep the reports forever but rarely access them—Archive is ideal. But raw telemetry from IoT devices coming in every second? That belongs in Hot."

**Instructor Cues:**

* Ask: *“Which tier are you using by default in your storage accounts? Do you configure lifecycle rules to move data automatically?”*
* Tip: *“We’ll practice setting tier transitions using Bicep and lifecycle management soon—this is where cost savings are hiding.”*

---

### **Slide 8: Demo – Create Azure Storage Account with Python (Setup)**

#### Speaker Notes:

**Talking Points:**

* "Let’s step through a quick **Python-based deployment** of an Azure Storage account using the SDK and Azure credentials."
* "This demo is cross-platform—works on both Windows and Linux."

**Setup Requirements:**

* Make sure students have:

  * An active Azure subscription
  * Python 3.8 or newer installed
  * Necessary libraries: `azure-identity`, `azure-mgmt-resource`, `azure-mgmt-storage`, and `azure-storage-blob`

**Real-World Context:**

* "This is a great example of Infrastructure as Code beyond just Bicep or ARM. Dev teams often use SDKs to automate storage creation inside apps or scripts."

**Instructor Cues:**

* Ask: *“Anyone here using Python SDKs with Azure before? What for—resource provisioning or data access?”*
* Optional: *“If you don’t have Python set up, follow along visually—we’ll link this code in the course GitHub.”*

---

### **Slide 9: Demo – Create Azure Storage Account with Python (Code)**

#### Speaker Notes:

**Talking Points:**

* "Here we build and run a Python script to create a storage account in Azure—programmatically."
* "This demonstrates how to authenticate with `DefaultAzureCredential`, create a resource group, and provision a new storage account."

**Walkthrough Details:**

* Replace `SUBSCRIPTION_ID` with actual ID
* Customize the resource group name, region, and globally unique storage account name
* Code handles creation of both resource group and the storage account using the Azure SDK

**Real-World Context:**

* "This mirrors what you'd do via Portal or Bicep—but gives you the flexibility to embed resource creation in a CI/CD pipeline or developer workflow."

**Instructor Cues:**

* Ask: *“Where do you think this approach is most useful—infrastructure engineers or app developers?”*
* Tip: *“Notice how the StorageV2 kind and HTTPS enforcement are explicitly declared—that’s a security best practice.”*

---

### **Slide 10: Demo – Upload to Azure Storage Account with Python**

#### Speaker Notes:

**Talking Points:**

* "Once our storage account is deployed, let’s upload a file to it using Python and the Blob SDK."
* "This script handles container creation and blob upload in one go."

**Code Highlights:**

* Uses UUID to create a unique blob name
* Checks for existing container and creates it if missing
* Uploads a local file (`myfile.txt`) to the container

**Real-World Context:**

* "This kind of automation is perfect for serverless jobs, CI/CD steps, or one-off data sync operations."
* "You could trigger this from Azure Functions, Logic Apps, or inside an ML training pipeline."

**Instructor Cues:**

* Ask: *“How would you secure this blob upload in a real app? Would you use SAS tokens, managed identity, or connection strings?”*
* Reinforce: *“This is a great base for building storage automation and aligns with real-world development workflows.”*

---

### **Slide 11: Pop Quiz – Blob Retention Scenario**

#### Speaker Notes:

**Scenario Recap:**

* "Let’s test your knowledge with a scenario: Your team needs to ensure that blobs **cannot be deleted or overwritten** for 7 days—except under legal hold."

**Answer Options Breakdown:**

* A. **RA-GRS** – Redundant storage option, but not a write/delete lock.
* B. **Immutable Blob Policies** – ✅ This is the correct answer. Enforce time-based retention and legal holds.
* C. **Soft-delete and versioning** – Allows rollback but doesn’t prevent overwrites.
* D. **GPv2 with snapshots** – Helps with recovery but not enforcement of immutability.

**Instructor Cues:**

* Ask: *“Anyone here implemented immutable blob storage before—maybe for compliance or audit logs?”*
* Transition: *“Let’s check the answer and briefly talk about when to use immutable policies.”*

---

### **Slide 12: Pop Quiz – Answer Review**

#### Speaker Notes:

**Answer Review:**

* "Correct answer: **B. Immutable Blob Storage policies with retention**."
* "This feature is critical for regulated industries—financial services, healthcare, legal—where data **must** be retained and cannot be altered."

**Real-World Context:**

* "With immutable policies, you can enforce write-once-read-many (WORM) behavior. This is useful for logs, legal archives, and sensitive datasets."
* "Legal hold support ensures you can override normal retention if you’re dealing with an investigation."

**Instructor Cues:**

* Ask: *“Can anyone think of a scenario in your org where immutability would’ve prevented data loss or non-compliance?”*
* Tip: *“These policies are available on standard GPv2 storage accounts and can be configured at the container level.”*
