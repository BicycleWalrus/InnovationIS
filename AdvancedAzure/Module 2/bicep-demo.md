# Instructor Demo: Deploying a Storage Account with Bicep (Azure Cloud Shell)

## Objective

In this **Instructor Demo**, we will use Azure Cloud Shell to deploy a simple Azure Storage Account using a Bicep template. Cloud Shell comes preconfigured with the Azure CLI and Bicep tooling, making it ideal for quick deployments.

**Topics Demonstrated:**

* Using Azure Cloud Shell for Bicep-based deployments
* Authoring a parameterized Bicep file directly in Cloud Shell
* Deploying infrastructure via `az deployment group create`
* Verifying the deployment via CLI

**Summary:** This demo highlights how instructors can deploy Bicep-defined infrastructure quickly from within Azure Cloud Shell.

---

## Procedure

> ⚠️ **NOTE:** This is an Instructor Demo designed to be shown live or in a recorded walkthrough. Students will perform a similar task in a guided lab.

1. In the Azure Portal, click the **Cloud Shell** icon in the top navigation bar.

0. Select **Bash** as your shell environment (if prompted).

0. Wait for the shell to initialize and load.

0. Use CLI to create a Resource Group for our Bicep Template.

    `student1 [ ~ ]$` `az group create --name bicep-arm-template-<your initials> --location eastus`

0. Create a working directory in Cloud Shell

    `student1 [ ~ ]$` `mkdir bicep-demo && cd bicep-demo`

0. Create the Bicep template file using vim.

    `student1 [ ~/bicep-demo ]$` `vim main.bicep`

    > 📄 **What are we building here?**
    > The `main.bicep` file defines a simple Azure Storage Account using declarative syntax. It includes:
    > * **Parameters** for the storage account name, location, and tags
    > * A **resource block** to create the storage account with specified properties
    > * An **output** to return the deployed storage account name

    ```bicep
    param storageAccountName string
    param location string = resourceGroup().location
    param tags object = {
      environment: 'demo'
      createdBy: 'Instructor'
    }
    
    resource sa 'Microsoft.Storage/storageAccounts@2022-09-01' = {
      name: storageAccountName
      location: location
      sku: {
        name: 'Standard_LRS'
      }
      kind: 'StorageV2'
      tags: tags
      properties: {}
    }
    
    output saName string = sa.name
    ```

0. Chance the **storageAccountName** to `instructor-<your initials>`. Do this to help remind students to input their initials when running their lab.

0. **Instructors -- Use Comments to Describe the elements of this BICEP Template** -- See this example:

    ```bicep
    // PARAMETERS
    // Parameters allow users to customize the deployment without editing the template code.
    // This improves reusability and flexibility during deployment.
    
    param storageAccountName string = 'instructor-trp' 
    /* A required string parameter named `storageAccountName`. 
       This defines the name of the storage account to be created.
       In this example, a default value 'instructor-trp' is provided. */
    
    param location string = resourceGroup().location
    /* Optional parameter with a default value that dynamically retrieves the 
       location of the resource group where this template is being deployed. */
    
    param tags object = {
      environment: 'demo'
      createdBy: 'Instructor'
    }
    /* Optional object parameter defining metadata tags for the resource. 
       These tags are useful for resource classification, cost tracking, 
       and management. Here, two tags are defined: `environment` and `createdBy`. */
    
    // RESOURCE DEFINITION
    // This block declares the Azure Storage Account resource that will be deployed.
    
    resource sa 'Microsoft.Storage/storageAccounts@2022-09-01' = {
      name: storageAccountName     // Name of the storage account, pulled from the parameter above
      location: location           // Location of the storage account, defaults to resource group location
      sku: {
        name: 'Standard_LRS'       // Specifies the replication strategy and pricing tier (Locally Redundant Storage)
      }
      kind: 'StorageV2'            // Specifies the storage account type. 'StorageV2' is the most commonly used type.
      tags: tags                   // Applies the custom tags defined earlier to the storage account
      properties: {}               // Required property block, left empty for basic configuration
    }
    
    // OUTPUT
    // Outputs allow values from the deployment to be returned for further use, debugging, or scripting.
    
    output saName string = sa.name
    /* Outputs the name of the created storage account.
       This is helpful to reference the resource post-deployment, especially in scripts or pipelines. */
    ```

0. Deploy the Bicep file using Azure CLI. **Set the correct name for the Resource Group, by inputing the correct initials**. Note how we're using the $RANDOM environment variable to generate a pseudo-random number to ensure uniqueness.

    `student1 [ ~/bicep-demo ]$` `az deployment group create --resource-group bicep-arm-template-<your initials> --template-file main.bicep --parameters storageAccountName=storacct$RANDOM`

    > JSON Output showing details of the deployed storage account will appear, confirming successful deployment.

### View the resource in the Azure Portal

1. Open the [Azure Portal](https://portal.azure.com/).

0. Navigate to **Resource groups** and select your group.

0. Locate the newly created Storage Account.

0. Confirm the **Tags** and **Location** values match your Bicep file.

---

### Mandatory Clean-up.

1. Delete the Resource Group and all of its resources from the CLI.

    `student1 [ ~/bicep-demo ]$` `az group delete --name bicep-arm-template-<your-initials> --yes --no-wait`

    > Explanation:
    > * `--name bicep-arm-template`: Specifies the name of the resource group to delete.
    > * `--yes`: Automatically confirms the deletion (no interactive prompt).
    > * `--no-wait`: (Optional) Returns immediately without waiting for the operation to finish.

---

## ✅ Conclusion

In this Instructor Demo, we used **Azure Cloud Shell** to deploy a parameterized Bicep template for a storage account. This included writing a Bicep file directly in the Cloud Shell environment, deploying via the Azure CLI, and verifying the results both via CLI and the Azure Portal. Using Cloud Shell simplifies tooling setup and helps focus on learning Bicep syntax and deployment flow.

This demo lays the groundwork for students to perform a similar hands-on deployment in **Lab 1**.
* Start working on the **slide deck content**?
* Package everything into a **module delivery bundle**?
