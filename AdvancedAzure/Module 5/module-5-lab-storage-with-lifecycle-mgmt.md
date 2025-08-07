## Deploy Storage with Lifecycle Management Policy

### Objective

In this lab, students will learn how to use **Bicep**, a domain-specific language (DSL) for deploying Azure resources declaratively. The lab focuses on automating storage lifecycle management to reduce costs and ensure compliance. Learners will write a Bicep template to create a **Storage Account** and attach a **lifecycle management policy** to it that deletes blobs after a certain period.

**Topics Covered:**

* Creating an Azure Resource Group using Azure CLI
* Writing a Bicep template to deploy a Storage Account
* Defining a blob lifecycle management policy in Bicep
* Deploying Bicep templates using Azure CLI
* Verifying deployment results in the CLI or Azure Portal

By completing this lab, learners will understand how to automate Azure resource deployment and management using Bicep templates.

---

## **Procedure**

> **Tip:** You’ll complete this entire lab using **Azure Cloud Shell (Bash)** — no local setup required!

1. **Launch Azure Cloud Shell**
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.
    - Choose **Bash** if prompted.
    - Wait for the Cloud Shell to fully initialize.

0. Create a Resource Group for your storage resources. Replace `<your-initials>` with your **own initials** to make the name unique.

    `student1 [ ~ ]$` `az group create --name storage-mgmt-<your-initials> --location eastus`

0. Create a working directory and navigate into it.

    `student1 [ ~ ]$` `mkdir storage--mgmt && cd storage--mgmt`

0. **Create a Bicep file to deploy VMSS infrastructure**.

    `student1 [ ~/storage-mgmt ]$` `vim storage-mgmt.bicep`

    ```bicep
    // Parameters for naming and location
    param storageAccountName string
    param location string = resourceGroup().location
    
    // Create a general-purpose v2 storage account with LRS redundancy
    resource storageAccount 'Microsoft.Storage/storageAccounts@2022-09-01' = {
      name: storageAccountName
      location: location
      sku: {
        name: 'Standard_LRS'
      }
      kind: 'StorageV2'
      properties: {
        accessTier: 'Hot'
      }
    }
    
    // Attach a lifecycle management policy to the storage account
    resource lifecyclePolicy 'Microsoft.Storage/storageAccounts/managementPolicies@2021-04-01' = {
      name: 'default'
      parent: storageAccount
      properties: {
        policy: {
          rules: [
            {
              enabled: true
              name: 'deleteOldBlobs'
              type: 'Lifecycle'
              definition: {
                filters: {
                  blobTypes: ['blockBlob']
                  prefixMatch: ['container1/']
                }
                actions: {
                  baseBlob: {
                    delete: {
                      daysAfterModificationGreaterThan: 30
                    }
                  }
                }
              }
            }
          ]
        }
      }
    }
    ```

0. **Deploy the Bicep template using Azure CLI**

    `student1 [ ~/storage-mgmt ]$`

    ```
     az deployment group create \
      --resource-group storage--mgmt-<your-initials> \
      --template-file storage-mgmt.bicep \
      --parameters storageAccountName=storagelc<yourinitials>
    ```

> ### What You’ve Just Created
>
> With this procedure, you deployed a **Storage Account** in Azure using a **Bicep template**, and attached a **lifecycle management policy** to it. This policy is configured to automatically **delete block blobs** stored in a container named `container1` after they have existed for more than **30 days**.
>
> Here’s what the Bicep template accomplishes:
>
> * **Creates a Storage Account** of type `StorageV2`, with `Standard_LRS` redundancy and an access tier set to `Hot`, optimized for frequent access.
> * **Defines a Lifecycle Policy Rule** that:
>
>   * Targets **block blobs** only.
>   * Applies only to blobs within a specific container (`container1/`).
>   * Automatically **deletes blobs** after 30 days to reduce storage costs and enforce data retention compliance.
>
> This pattern is commonly used in real-world environments to automate cost control, enforce retention policies, and reduce the need for manual storage management.

#### Verify the Deployment from the Azure Portal.

### **Confirm Storage Account Creation**

1. Sign in to [https://portal.azure.com](https://portal.azure.com)
0. Navigate to **Resource groups**.
0. Locate and click on the **Resource Group** you used during deployment: `storage-mgmt-<your initials`
0. Under the **Resources** list, find the **Storage Account** 
0. Click the storage account to open its **Overview** pane
0. Verify:

   * **Kind** is `StorageV2`
   * **Location** matches the value used (`eastus`, if default)
   * **Default Access Tier** is set to `Hot`

### **Verify Lifecycle Management Policy**

1. Inside the **Storage Account** page, from the left-sidebar, select **Data Management** section
0. Click on **Lifecycle management**
0. Confirm that a lifecycle policy exists with the following:

   * **Policy Name:** `deleteOldBlobs`
   * **Status:** `Enabled`
   * **Action:** `Delete blobs`
   * **Condition:**
     * **Blob type:** `Block Blob`
     * **Prefix Match:** `container1/`
     * **Age Condition:** `Base Blobs haven't been modified in 30 days.`

> You will need to click into the rule, and navigate the tabs to see the full definition

---

### Conclusion

In this lab, you learned how to use a Bicep template to automate the deployment of an Azure Storage Account with a lifecycle management policy. You created a resource group, authored the infrastructure-as-code using Bicep, deployed it using `az deployment group create`, and validated that your policy was applied. This lab demonstrates how Infrastructure as Code (IaC) and lifecycle management work together to enforce automated storage governance and cost optimization.
