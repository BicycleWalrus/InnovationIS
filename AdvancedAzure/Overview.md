## **Day 1 – Core Azure Services & Infrastructure as Code**

> **Theme:** Modern cloud engineers must deploy confidently, automate securely, and reason about what Azure *does* under the hood—not just how to click through it.

---

### **Module 1: Azure Platform & Control Plane Orientation**

* ARM as Azure's deployment engine
* Resource groups, regions, subscriptions
* Portal, CLI, and SDK layers of interaction
* *Instructor Demo*: Azure Portal walkthrough
* *(Optional Lab)*: Deploy a VM via the Portal for reference

---

### **Module 2: Infrastructure as Code – Intro to Bicep**

* What is Bicep and why it replaced JSON
* Anatomy of a Bicep file: parameters, variables, resources, outputs
* Tooling: VS Code + Bicep extension, `az deployment`
* *Instructor Demo*: Deploy storage account using Bicep
* *Lab 1*: Author and deploy a basic Bicep template (e.g., storage + tags)

---

### **Module 3: Networking Deep Dive – VNets, NSGs, Peering**

* VNets, subnets, IP addressing, DNS options
* NSGs, Application Security Groups, service tags
* VNet peering and regional/global scenarios
* *Instructor Demo*: Diagram-based network walkthrough
* *Lab 2*: Bicep-deploy VNet with subnets, NSGs, and peering

---

### **Module 4: Compute Patterns & VM Scale Sets**

* VM types, availability sets vs zones, scale sets
* Placement groups, autoscaling rules, lifecycle
* *Instructor Demo*: Compare scale set options
* *Lab 3*: Deploy VMSS using Bicep with autoscale profile and diagnostics

---

### **Module 5: Storage Tiers, Types, and Blobs**

* Blob, File, Table, Queue – when to use what
* Premium vs Hot/Cool/Archive tiers
* *Instructor Demo*: Access blob from deployed storage account
* *Lab 4*: Bicep-deploy storage with lifecycle management policy

---

## **Day 2 – Identity, Applications, Monitoring, and Resilience**

> **Theme:** Empowering secure access, designing for scale, and implementing observability across modern Azure architectures.

---

### **Module 6: Identity & Access with Microsoft Entra**

* Microsoft Entra overview (formerly Azure AD)
* User types, groups, roles, and RBAC
* App registrations, SSO, MFA, conditional access
* *Instructor Demo*: Entra user/group setup and access assignment
* *Lab 5*: Configure RBAC and deploy a secure Bicep resource using a managed identity

---

### **Module 7: Azure App Services & Serverless Patterns**

* App Service, Container Apps, Azure Functions
* Deployment slots, scaling models, CI/CD integration
* *Instructor Demo*: Deploy a Function via Portal + CLI
* *Lab 6*: Bicep-deploy a Function App with storage + monitoring

---

### **Module 8: Durable Functions & Integration Patterns**

* Durable Functions, bindings, orchestration vs activity
* Event-driven architecture in Azure (Event Grid, Service Bus)
* *Instructor Demo*: Durable function sequence (chaining + fan-out/fan-in)
* *Lab 7*: Deploy Durable Function via Bicep with Cosmos DB output

---

### **Module 9: Azure Monitor, Autoscale & Observability**

* Azure Monitor, Log Analytics, Application Insights
* Metrics vs logs, autoscaling with custom triggers
* *Instructor Demo*: Configure autoscale on VMSS using CPU metric
* *Lab 8*: Configure autoscale and Azure Monitor alerts via Bicep

---

### **Module 10: Resilient Architecture & Geo-Distributed Design**

* Traffic Manager, Azure Front Door, CDN
* Global failover patterns and DNS routing
* *Instructor Demo*: Priority vs performance routing demo
* *Lab 9*: Deploy geo-redundant app infra with Traffic Manager

---

### **Final Wrap-Up: Capstone and Q\&A**

* Tie back to Day 1 & 2 concepts
* Group discussion: “Design a resilient Azure app for a real use case”
* Review Bicep patterns, resource structure, and deployment lifecycle
* Discuss Azure resource graph, tags, and policy enforcement
* Provide next steps for continued learning (GitHub, Learn, Docs)

---

## **Supporting Materials To Be Developed**

* **Speaker Notes Rewrite** for each module with:
  * Key talking points
  * Real-world context
  * Sample questions
  * Demo cues and lab tie-ins
* **All Labs Rewritten in Bicep**
* **Instructor Demos** scripted (CLI + Portal variants)
* **Optional Slide Updates** for visuals (networking, scale sets, etc.)
