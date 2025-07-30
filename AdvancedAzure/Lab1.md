# Lab 1: Modularizing Azure Deployments with ARM Templates

**Objective**:
In this lab, you'll build and deploy modular Azure infrastructure using **Azure Resource Manager (ARM) templates**. Instead of defining all resources in a single file, you'll extract components into **linked templates**, upload them to **Azure Blob Storage**, and use a **SAS (Shared Access Signature)** URI to reference them securely in your main deployment file.

You’ll gain hands-on experience with:

* Authoring and editing JSON-based ARM templates
* Modularizing infrastructure as code
* Using `vim` and terminal tools to manage your workflow
* Uploading templates to Azure Storage and generating SAS tokens
* Deploying infrastructure from Cloud Shell using Azure CLI

## 🧪 What You’ll Build

By the end of this lab, you will have deployed:

* A modular infrastructure stack consisting of a Windows Virtual Machine
* A storage account deployed via a **linked template**
* Resources defined with dependency awareness and parameterized design

---

## 🧱 Lab Tasks

| Task | Description                                                                                 |
| ---- | ------------------------------------------------------------------------------------------- |
| 1    | **Set Up Your Working Environment** — clone the lab repo, configure your workspace          |
| 2    | **Create and Refactor Templates** — copy and simplify a base template for storage           |
| 3    | **Upload Linked Template to Azure Storage** — and generate a SAS token for secure reference |
| 4    | **Modify the Main Template** — replace inline storage with a `templateLink` deployment      |
| 5    | **Update Resource Dependencies** — reflect modular design in VM references and outputs      |
| 6    | **Deploy to Azure** — using Azure CLI and publicly accessible templates                     |
| 7    | **Cleanup and Next Steps** — tear down deployed resources and plan future modularization    |

---

## ⏱️ Estimated Lab Time

**45–60 minutes**

---

## 🧰 Tools Required

| Tool                   | Notes                                                                    |
| ---------------------- | ------------------------------------------------------------------------ |
| Azure Subscription     | Required to create resources (free tier OK)                              |
| Azure Cloud Shell      | Used for all command-line and deployment operations (PowerShell or Bash) |
| Git                    | Used to clone the lab files                                              |
| `vim` or `nano`        | Recommended for editing JSON templates                                   |
| jq or python JSON tool | Optional, for validating JSON structure                                  |

---

## 🧰 Task 1: Set Up Your Working Environment (Command-Line Focused)

**Goal**: Prepare your local environment for working with Azure Resource Manager (ARM) templates using command-line tools and `vim`.

### ✅ Step 1: Clone the Lab Repository

  1. Open a terminal or PowerShell session.
  
  0. Navigate to the directory where you want to download the lab materials.
  
  0. Run the following command to clone the repository:
  
     ```bash
     git clone https://github.com/jruels/adv-azure-classpage.git
     ```
  
  0. Change into the lab directory:
  
     ```bash
     cd adv-azure-classpage/labs/01-arm-templates
     ```

### ✅ Step 2: Create Your Working Directory

1. Create a new directory to store your customized templates:
  
    ```bash
    mkdir -p ~/lab1/templates/storage
    ```
  
0. Copy the base deployment template to your new working directory:
  
    ```bash
    cp templates/azuredeploy.json ~/lab1/templates/
    cp ~/lab1/templates/azuredeploy.json ~/lab1/templates/storage/storage.json
    ```
  
    > At this point, you should have two identical files:
  
      ```
      ~/lab1/templates/azuredeploy.json
      ~/lab1/templates/storage/storage.json
      ```

### ✅ Step 3: Open the Template Files in `vim`

You’ll use `vim` to edit and explore both JSON files.

1. Open the main template:
  
    ```bash
    vim ~/lab1/templates/azuredeploy.json
    ```
  
0. Use standard `vim` navigation and search commands (e.g., `/resources`) to locate and inspect the defined resource blocks.
  
0. Repeat for the linked template:
  
    ```bash
    vim ~/lab1/templates/storage/storage.json
    ```

### ✅ Step 4: Identify Defined Resources

In `azuredeploy.json`, review the top-level `resources` array. You should find definitions for:

* `Microsoft.Storage/storageAccounts`
* `Microsoft.Network/publicIPAddresses`
* `Microsoft.Network/virtualNetworks`
* `Microsoft.Network/networkInterfaces`
* `Microsoft.Compute/virtualMachines`

Take note of these, as you'll be extracting one of them into a standalone linked template in the next task.

> 💡**Tip**: If you're not comfortable in `vim`, feel free to use any terminal-based editor like `nano` or `neovim`. However, this lab assumes comfort with command-line workflows.

---

## 🔗 Task 2: Refactor the Linked Template for Storage Resources

**Goal**: Convert the copied `storage.json` file into a standalone, linked ARM template that defines only a storage account and returns its `blob` URI as an output.

### 🧠 Background

You previously duplicated the full `azuredeploy.json` into `storage.json`. Now you’ll trim everything in `storage.json` **except** the storage account definition and reformat it for modular use.

### ✅ Step 1: Open the Linked Template in `vim`

```bash
vim ~/lab1/templates/storage/storage.json
```

### ✅ Step 2: Remove All Resources Except the Storage Account

Inside `vim`:

1. Press `/resources` and hit Enter to jump to the `resources` array.
  
0. Use visual mode (`v` or `V`) to highlight and delete all resource blocks **except** the one of type:
  
    ```json
    "type": "Microsoft.Storage/storageAccounts"
    ```
  
0. Delete other resource definitions by selecting them and pressing `d`.

### ✅ Step 3: Replace the `name` Property Reference

* Change the `"name"` property from using a variable to using a parameter:

Before:

```json
"name": "[variables('storageAccountName')]"
```

After:

```json
"name": "[parameters('storageAccountName')]"
```

You can search (`/storageAccountName`) and edit inline using `cw`, `i`, or `r` commands.

### ✅ Step 4: Remove the `variables` Section

* Press `/variables` and `dd` (delete line) repeatedly until the entire block is removed.
* If you're unsure, delete from the opening `{` to the closing `}` of the `variables` block.

### ✅ Step 5: Trim Parameters and Add One for `storageAccountName`

Locate the `parameters` section (search for `/parameters`) and modify it to include only:

```json
"location": {
  "type": "string",
  "defaultValue": "[resourceGroup().location]",
  "metadata": {
    "description": "Deployment location for all resources."
  }
},
"storageAccountName": {
  "type": "string",
  "metadata": {
    "description": "Name of the Storage Account."
  }
}
```

Delete all other parameter definitions.

### ✅ Step 6: Add the `outputs` Block

Scroll to the end of the file and add the following `outputs` section:

```json
"outputs": {
  "storageUri": {
    "type": "string",
    "value": "[reference(parameters('storageAccountName')).primaryEndpoints.blob]"
  }
}
```

### ✅ Step 7: Save and Exit

Press:

```
:wq
```

Your `storage.json` is now a proper modular linked template.

### 🔍 Verify File Structure

You can quickly preview your updated structure using:

```bash
cat ~/lab1/templates/storage/storage.json | jq '.'
```

(If `jq` is not installed, you can use `python -m json.tool` or inspect manually.)

### ✅ Result

Your `storage.json` now contains:

* Only the `storageAccounts` resource.
* Two parameters: `location` and `storageAccountName`.
* A `storageUri` output to pass back to the parent template.

---

## ☁️ Task 3: Upload the Linked Template to Azure Blob Storage

**Goal**: Make the `storage.json` linked template accessible to Azure Resource Manager by uploading it to Azure Blob Storage and generating a **SAS (Shared Access Signature)** URI.

Azure requires linked templates to be accessible via HTTPS. You cannot reference local files. To meet this requirement, you’ll upload the template to a blob container and generate a time-limited, signed URL.

### ✅ Prerequisites

* You must have an active Azure subscription.
* You must be able to access the [Azure Cloud Shell](https://shell.azure.com), and switch to **PowerShell** mode.
* Your `storage.json` file should already be prepared from Task 2 and located at:

```
~/lab1/templates/storage/storage.json
```

## ⚙️ Step-by-Step Instructions

### ✅ Step 1: Launch Azure Cloud Shell in PowerShell Mode

1. Open your browser and go to:

    👉 [https://shell.azure.com](https://shell.azure.com)

0. If prompted, log in with your Azure account.

0. Switch to **PowerShell** by selecting it in the dropdown at the top-left corner of the Cloud Shell pane.

### ✅ Step 2: Create a Resource Group, Storage Account, and Blob Container

Paste and execute the following script **line by line**, entering values when prompted.

```powershell
# Prompt for values
$projectNamePrefix = Read-Host -Prompt "Enter a project name (no spaces or special characters):"
$location = Read-Host -Prompt "Enter an Azure location (e.g. westus, centralus)"

# Construct names
$resourceGroupName = "${projectNamePrefix}rg"
$storageAccountName = "${projectNamePrefix}stracc"
$containerName = "linktempblobcntr"
$fileName = "storage.json"
$linkedTemplateURL = "https://raw.githubusercontent.com/jruels/adv-azure-classpage/master/labs/01-arm-templates/.solutions/storage.json"

# Download linked template into Cloud Shell
Invoke-WebRequest -Uri $linkedTemplateURL -OutFile "$home/$fileName"

# Create a new resource group
New-AzResourceGroup -Name $resourceGroupName -Location $location

# Create a storage account
$storageAccount = New-AzStorageAccount `
  -ResourceGroupName $resourceGroupName `
  -Name $storageAccountName `
  -Location $location `
  -SkuName "Standard_LRS"

# Set context and create blob container
$context = $storageAccount.Context
New-AzureStorageContainer -Name $containerName -Context $context

# Upload the template file
Set-AzureStorageBlobContent `
  -Container $containerName `
  -File "$home/$fileName" `
  -Blob $fileName `
  -Context $context

# Generate a SAS URI valid for 24 hours
$templateURI = New-AzureStorageBlobSASToken `
  -Context $context `
  -Container $containerName `
  -Blob $fileName `
  -Permission r `
  -ExpiryTime (Get-Date).AddHours(24.0) `
  -FullUri

# Output values for future use
Write-Host "`n--- OUTPUT VALUES ---"
Write-Host "Resource Group Name: $resourceGroupName"
Write-Host "Storage Account Name: $storageAccountName"
Write-Host "Linked Template URI (SAS): $templateURI"
Write-Host "------------------------`n"
```

> 📌 **IMPORTANT:** Save the values for `$resourceGroupName` and `$templateURI`. You’ll need them in Task 4.

### 🧪 Optional: Verify the URI

You can test that your link is publicly accessible by visiting the generated URI in your browser. It should download `storage.json`.

## ✅ Result

By the end of this task, you will have:

* Created a resource group and storage account.
* Uploaded `storage.json` to a secure blob container.
* Generated a 24-hour SAS token and URI.

This SAS-enabled URI will allow your main ARM template to reference `storage.json` in Task 4.

---

## 🔁 Task 4: Modify the Main Template to Use the Linked Storage Template

**Goal**: Replace the inlined storage account resource in your main `azuredeploy.json` file with a reference to the uploaded `storage.json` linked template using its SAS URI.

### 🧠 Background

You’ve uploaded the `storage.json` template to Azure Blob Storage and generated a **SAS token URI** in Task 3. Now you will:

* Remove the original `Microsoft.Storage/storageAccounts` resource from the main template.
* Insert a reference to the linked template using the `Microsoft.Resources/deployments` resource type.
* Pass parameters (`storageAccountName`, `location`) from the main template to the linked one.

## 🧰 File to Edit

```
~/lab1/templates/azuredeploy.json
```

Open the file with:

```bash
vim ~/lab1/templates/azuredeploy.json
```

### ✅ Step 1: Delete the Storage Resource from `resources`

1. Press `/resources` in `vim` and hit Enter to jump to the top of the `resources` array.

2. Search for the storage block:

   ```json
   {
     "type": "Microsoft.Storage/storageAccounts",
     ...
   }
   ```

3. Use visual mode (`V`) to select the entire object and delete it with `d`.

> 🔎 Be careful to remove **only** the storage account block. Do not delete unrelated resources.

### ✅ Step 2: Insert the Linked Template Reference

Just after the opening of the `resources` array, add the following block (replace the URI placeholder):

```json
{
  "name": "linkedTemplate",
  "type": "Microsoft.Resources/deployments",
  "apiVersion": "2024-07-01",
  "properties": {
    "mode": "Incremental",
    "templateLink": {
      "uri": "<PASTE_YOUR_SAS_URI_HERE>"
    },
    "parameters": {
      "storageAccountName": {
        "value": "[variables('storageAccountName')]"
      },
      "location": {
        "value": "[parameters('location')]"
      }
    }
  }
},
```

> 🧠 **What this does**:
>
> * Tells the main template to call a remote template using `templateLink`.
> * Passes in `storageAccountName` and `location` so the linked template can use them.

📌 **Remember to replace** `<PASTE_YOUR_SAS_URI_HERE>` with the exact SAS URL you copied in Task 3.

### ✅ Step 3: Save and Exit

Inside `vim`, press:

```bash
:wq
```

To save and exit.

### ✅ Step 4: Validate Your JSON

Run the following to check the structure:

```bash
cat ~/lab1/templates/azuredeploy.json | jq .
```

## ✅ Result

Your `azuredeploy.json` file now:

* No longer defines a storage account directly.
* References a modular template for the storage account.
* Passes required parameters to the linked template.

---

## 🔧 Task 5: Update the Main Template to Reflect New Dependencies

**Goal**: Now that the storage account is defined in a linked template, you must update the `azuredeploy.json` file to ensure:

1. The virtual machine **depends** on the linked template (not the old inline storage resource).
2. The VM diagnostic profile uses the **output value** from the linked template to access the storage URI.

## 🧰 File to Edit

```
~/lab1/templates/azuredeploy.json
```

Open it with:

```bash
vim ~/lab1/templates/azuredeploy.json
```

## ✅ Step 1: Update the `dependsOn` Block for the Virtual Machine

1. Search for the virtual machine definition:

   In `vim`, type:

   ```vim
   /"type": "Microsoft.Compute/virtualMachines"
   ```

0. Locate the `dependsOn` section. It may look like this:

   ```json
   "dependsOn": [
     "[resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))]",
     "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
   ],
   ```

0. Modify it to depend on the linked template instead:

   ```json
   "dependsOn": [
     "linkedTemplate",
     "[resourceId('Microsoft.Network/networkInterfaces/', variables('nicName'))]"
   ],
   ```

   > 🧠 This ensures the virtual machine waits for the **linked storage template** to complete before deploying.

## ✅ Step 2: Replace the Storage URI in the Diagnostics Profile

1. Still in the VM resource block, locate the following:

   ```json
   "diagnosticsProfile": {
     "bootDiagnostics": {
       "enabled": true,
       "storageUri": "[reference(resourceId('Microsoft.Storage/storageAccounts/', variables('storageAccountName'))).primaryEndpoints.blob]"
     }
   }
   ```

2. Replace the `storageUri` line with a reference to the **linked template’s output**:

   ```json
   "diagnosticsProfile": {
     "bootDiagnostics": {
       "enabled": true,
       "storageUri": "[reference('linkedTemplate').outputs.storageUri.value]"
     }
   }
   ```

   > 🔎 This uses the value returned from the `outputs` section of `storage.json`, allowing your main template to retrieve the proper URI dynamically.

### ✅ Step 3: Save and Exit

Once your edits are complete:

```bash
:wq
```

### ✅ Step 4: Validate the Template

Use this to check the structure and sanity of your file:

```bash
cat ~/lab1/templates/azuredeploy.json | jq .
```

Or:

```bash
az deployment group validate \
  --resource-group <your-rg> \
  --template-file ~/lab1/templates/azuredeploy.json
```

> This will confirm your syntax is clean and dependencies are correctly structured.

## ✅ Result

At this point:

* The storage account is deployed from the **linked template**.
* The virtual machine **waits** on the linked template via `dependsOn`.
* Boot diagnostics uses the linked template’s `storageUri` **output value** instead of trying to calculate it itself.

---

Absolutely! Here's **Task 6** rewritten to match your standards: clean structure, terminal-first workflow, `vim`-friendly editing assumptions, and precise instructions using **Azure CLI** from **Cloud Shell** — no VS Code or GUI steps required.

---

## 🚀 Task 6: Deploy Your ARM Templates to Azure Using Azure CLI

**Goal**: Deploy the main ARM template (`azuredeploy.json`) from **Azure Cloud Shell** using the `az deployment group create` command, referencing the template via a publicly accessible Blob Storage URI.

---

### 🧠 Background

Azure CLI only supports linked templates if they are accessible over HTTPS. You already:

* Modularized `storage.json` and uploaded it to blob storage (Task 3)
* Inserted a `templateLink` in `azuredeploy.json` (Task 4)
* Updated `dependsOn` and `diagnosticsProfile` to use the linked template output (Task 5)

Now you'll upload the final `azuredeploy.json`, retrieve its HTTPS URL, and deploy it from Cloud Shell.

---

## ✅ Step 1: Upload `azuredeploy.json` to Blob Storage

You’ll use the same storage account and container created in **Task 3**.

### ⬇ Upload the template

1. Open Azure Cloud Shell (PowerShell or Bash):

   👉 [https://shell.azure.com](https://shell.azure.com)

2. Upload the file from your local machine:

   Use the **Upload/Download** menu (top-right of Cloud Shell UI) to upload:

   ```
   ~/lab1/templates/azuredeploy.json
   ```

   > 💡 The upload goes into the Cloud Shell’s home directory (usually `/home/<username>`).

3. Use the following to upload it into blob storage (update values accordingly):

```powershell
# Reuse existing values or reassign if necessary
$resourceGroupName = "<your previously used RG name>"
$storageAccountName = "<your previously used storage account name>"
$containerName = "linktempblobcntr"
$fileName = "azuredeploy.json"
$context = (Get-AzStorageAccount -ResourceGroupName $resourceGroupName -Name $storageAccountName).Context

# Upload to blob
Set-AzureStorageBlobContent `
  -Container $containerName `
  -File "$home/$fileName" `
  -Blob $fileName `
  -Context $context

# Generate SAS token for the main template
$templateUri = New-AzureStorageBlobSASToken `
  -Context $context `
  -Container $containerName `
  -Blob $fileName `
  -Permission r `
  -ExpiryTime (Get-Date).AddHours(24) `
  -FullUri

Write-Host "`nTemplate URI for deployment:"
Write-Host $templateUri
```

📌 **Copy the `$templateUri` value** — you’ll use it for the final deployment step.

---

## ✅ Step 2: Deploy the Template Using Azure CLI

Use this command to deploy the full environment:

```powershell
az deployment group create `
  --name arm-linked-deploy `
  --resource-group armlabrg `
  --template-uri "https://armlabstracc.blob.core.windows.net/public/lab1/templates/azuredeploy.json?sv=2025-05-05&se=2025-07-31T17%3A59%3A04Z&sr=b&sp=r&sig=qeznD8AFoekF7PmbP%2BE8SElOtk1%2BdI6R5AKStLXdAKc%3D"
```

📥 You will be prompted for:

* **adminUsername**
* **adminPassword**
* **dnsLabelPrefix**

Enter any values you like — they will be used to create the Windows VM.

---

### 🔍 Step 3: Verify Deployment (Optional)

While the CLI may appear unresponsive during deployment, you can confirm progress via the Azure Portal:

👉 [https://portal.azure.com/#view/HubsExtension/BrowseResourceGroups](https://portal.azure.com/#view/HubsExtension/BrowseResourceGroups)

You should see:

* A VM named `SimpleWinVM`
* Public IP, NIC, VNet, and your linked storage account
* A successful deployment under **Deployments** in the resource group

---

## ✅ Result

If successful, your deployment:

* Pulled in the linked `storage.json` via `templateLink`
* Deployed storage, networking, and compute resources
* Used modular ARM template design

---

Absolutely! Here is **Task 7**, written in your preferred style — terminal-based, vim-friendly, and focused on clear, practical cleanup and next-step considerations.

---

## 🧹 Task 7: Cleanup and Next Steps

**Goal**: Prevent unnecessary Azure charges by deleting your deployed resources, and understand where to go next if you want to expand the modularization of your ARM templates.

---

## ✅ Step 1: Delete Your Resource Group

All resources created during this lab (VM, NIC, public IP, VNet, storage) were deployed into a single resource group. You can delete the entire group to clean up in one command.

From **Azure CLI** (PowerShell, Bash, or Cloud Shell):

```bash
az group delete --name <your-resource-group-name> --yes --no-wait
```

**Example:**

```bash
az group delete --name armlabrg --yes --no-wait
```

> 🧠 The `--no-wait` flag returns immediately without waiting for deletion to finish.

---

## ✅ Step 2: Confirm Deletion

If you'd like to confirm that the resource group is queued for deletion:

```bash
az group show --name armlabrg
```

Once deleted, this command will return an error indicating the group no longer exists.

---

## ✅ Step 3: (Optional) Remove Templates from Cloud Shell Home

If you uploaded files into your Cloud Shell environment (like `storage.json` or `azuredeploy.json`), you can clean them up:

```bash
rm ~/storage.json
rm ~/azuredeploy.json
```

---

## 🧭 What's Next?

If you’d like to continue modularizing your ARM templates, here are some suggested next steps:

### ➕ Modularize Networking

Create a new `network.json` file that includes:

* Virtual network
* Subnet
* Network interface
* Public IP

Then reference it in `azuredeploy.json` the same way you did for `storage.json`.

### 🧱 Modularize Compute

Move the VM definition into a `vm.json` linked template, and pass in:

* Storage URI (from `storage.json`)
* NIC ID (from `network.json`)

This leads to a fully modular and composable infrastructure deployment strategy.

### 🖼️ Example Module Dependency Diagram

```
+------------------+
|  storage.json    |
|  (StorageAccount)|
+--------+---------+
         |
         v
+--------+--------+       +------------------+
|   vm.json       |<------|  network.json    |
| (VirtualMachine)|       | (VNet, NIC, PIP) |
+-----------------+       +------------------+
```

---

## ✅ Summary

In this final task, you:

* Deleted your deployed Azure resources to avoid charges.
* Learned how to expand modular design further by isolating networking and compute layers.
* Practiced cleanup hygiene with uploaded templates and shell files.
