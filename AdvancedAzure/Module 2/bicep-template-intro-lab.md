# Deploying a Storage Account with Bicep (Azure Cloud Shell)

## Objective

In this lab, you’ll learn how to deploy a simple Azure Storage Account using a Bicep template—all from **Azure Cloud Shell**. You'll define parameters, tag your resources, and deploy using the Azure CLI.

**Topics Covered:**

* Writing Bicep templates with parameters and tags
* Using Azure Cloud Shell and CLI to deploy infrastructure
* Verifying resource creation in the Azure Portal
* Cleaning up resources using the CLI

**Summary:** By the end of this lab, you’ll have successfully deployed an Azure Storage Account using a Bicep template—all from within the browser using Cloud Shell.

---

## Procedure

> **Tip:** All of this lab will be completed in the browser-based Azure Cloud Shell. No local installation needed!

1. **Launch Azure Cloud Shell**
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.
    - Choose **Bash** if prompted.
    - Wait for the Cloud Shell to fully initialize.

0. Create a Resource Group for your storage account. Replace `<your-initials>` with your **own initials** to make the name unique.

    `student1 [ ~ ]$` `az group create --name bicep-arm-template-<your-initials> --location eastus`

0. Create a working directory and navigate into it.

    `student1 [ ~ ]$` `mkdir bicep-lab && cd bicep-lab`

0. Create a new Bicep template using Vim. **Visual Studio Code is installed in your learning environment if you'd prefer to use that**

    `student1 [ ~/bicep-demo ]$` `vim main.bicep`

    Then, paste the following content into the file and **replace `your-initials`** in the default name:

    ```bicep
    // PARAMETERS
    
    param storageAccountName string = 'student-yourinitials' 
    param location string = resourceGroup().location
    param tags object = {
      environment: 'student-lab'
      createdBy: 'student'
    }
    
    // RESOURCE DEFINITION
    
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
    
    // OUTPUT
    
    output saName string = sa.name
    ```

    > **Reminder:** Press `i` to enter Insert mode in Vim, and `Esc` + `:wq` to save and exit when you're done.

0. **Deploy the Bicep file using Azure CLI**. This command uses `$RANDOM` to generate a unique storage account name. **Update your resource group name accordingly.**

    `student1 [ ~/bicep-demo ]$` `az deployment group create --resource-group bicep-arm-template-<your-initials> --template-file main.bicep --parameters storageAccountName=storacct$RANDOM

> ✅ A successful deployment will return a JSON output with details about the deployed storage account.

### Verify the resource in the Azure Portal

1. Go to the [Azure Portal](https://portal.azure.com/)
2. Navigate to **Resource groups**
3. Find and open your resource group: `bicep-arm-template-<your-initials>`
4. Look for your new **Storage Account** and open it
5. Review the **Tags** and **Location** to confirm they match your template

### Clean up your resources

1. To avoid charges, delete your entire resource group:

    `student1 [ ~/bicep-demo ]$` `az group delete --name bicep-arm-template-<your-initials> --yes --no-wait`

    > This will delete the storage account and any other resources in the group.

---

## ✅ Conclusion

In this lab, you wrote and deployed a Bicep template using Azure Cloud Shell. You defined parameters, created a Storage Account resource, and verified the result via both the CLI and Azure Portal. This hands-on exercise gave you practical experience with declarative infrastructure using Bicep and Azure CLI—setting the stage for more complex infrastructure automation in future labs.
