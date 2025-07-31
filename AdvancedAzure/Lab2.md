## **Lab 2: Azure Virtual Machine Scale Sets**

### **Lab Duration:** 60–90 minutes

### **Lab Files:**

* `02-vms/files/azuredeploy.json`
* `02-vms/files/azuredeploy.parameters.json`
* `02-vms/files/install_iis_vmss.zip`

### **Lab Objectives:**

By the end of this lab, students will be able to:

* Deploy virtual machines (VMs) using the Azure Portal, PowerShell, and ARM templates.
* Configure networking for Azure VMs running both Windows and Linux.
* Deploy and manage Azure Virtual Machine Scale Sets.
* Use Desired State Configuration (DSC) extensions to install applications on scale set instances.

---

## **Exercise 1: Deploy Azure VMs via Portal, PowerShell, and ARM Templates**

### **Task 1: Deploy a Windows Server 2022 VM in an Availability Set using the Azure Portal**

**Instructions:**

1. In the Azure Portal, click **Create a resource**.

0. Search for **Windows Server**, then select **\[smalldisk] Windows Server 2022 Datacenter** and click **Create**.

0. On the **Create a Virtual Machine** page, configure the following:

   **Basics Tab:**

   * Subscription: *Your default subscription*
   * Resource Group: `azscaleset-RG` (create new)
   * Virtual Machine Name: `azscaleset-vm0`
   * Region: `East US` or your nearest supported region
   * Availability Options: **Availability Set**
   * Create a new availability set named `azscaleset-avset0` with:

     * Fault domains: 2
     * Update domains: 5
   * Security Type: **Standard**
   * Image: **Windows Server 2022 Datacenter (smalldisk)**
   * Size: **Standard DS1 v2**
   * Administrator Account:

     * Username: `Student`
     * Password: `Pa55w.rd1234`
   * Inbound Ports: **None**
   * Licensing: Leave default

0. **Disks Tab:**

   * OS Disk Type: **Standard HDD**

0. **Networking Tab:**

   * Create a new Virtual Network: `azscaleset-RG-vnet`

     * Address Space: `10.103.0.0/16`
     * Subnet Name: `subnet0`
     * Subnet Range: `10.103.0.0/24`
   * Leave public IP settings and NSG as default

0. **Management Tab:**

   * Boot Diagnostics: **Enable with new storage account** (default)

0. **Advanced Tab:** Leave all defaults

0. Click **Review + Create**, then click **Create**

**Notes:**

* Deployment may take 5–7 minutes
* You will configure NSG settings in a later exercise

---

### ✅ **Task 2: Deploy an Azure VM running Windows Server 2022 Datacenter into the existing availability set by using Azure PowerShell**

In this task, you will deploy a second VM (`azscaleset-vm1`) into the availability set you created earlier, using Azure PowerShell within the Azure Cloud Shell.


#### **Step 1: Launch Azure Cloud Shell**

1. In the Azure portal, click the **Cloud Shell** icon in the top navigation bar.
2. Select **PowerShell** if prompted for a shell type.
3. If this is your first time launching Cloud Shell, follow the prompts to create a storage account. Accept the default settings.

#### **Step 2: Define basic VM variables**

```powershell
$vmName = 'azscaleset-vm1'
$vmSize = 'Standard_DS1_v2'
```

> These variables set the name and size of the VM.

#### **Step 3: Retrieve the resource group and location**

```powershell
$resourceGroup = Get-AzResourceGroup -Name 'azscaleset-RG'
$location = $resourceGroup.Location
```

#### **Step 4: Retrieve the availability set, virtual network, and subnet**

```powershell
$availabilitySet = Get-AzAvailabilitySet -ResourceGroupName $resourceGroup.ResourceGroupName -Name 'azscaleset-avset0'
$vnet = Get-AzVirtualNetwork -Name 'azscaleset-RG-vnet' -ResourceGroupName $resourceGroup.ResourceGroupName
$subnetid = (Get-AzVirtualNetworkSubnetConfig -Name 'subnet0' -VirtualNetwork $vnet).Id
```

#### **Step 5: Create a network security group, public IP, and network interface**

```powershell
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -Name "$vmName-nsg"
$pip = New-AzPublicIpAddress -Name "$vmName-ip" -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -AllocationMethod Static -Sku Standard 
$nic = New-AzNetworkInterface -Name "$($vmName)$(Get-Random)" -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -SubnetId $subnetid -PublicIpAddressId $pip.Id -NetworkSecurityGroupId $nsg.Id
```

> ⚠️ You'll configure the NSG rules in a later exercise.

#### **Step 6: Set admin credentials**

```powershell
$adminUsername = 'Student'
$adminPassword = 'Pa55w.rd1234'
$adminCreds = New-Object PSCredential $adminUsername, ($adminPassword | ConvertTo-SecureString -AsPlainText -Force)
```

#### **Step 7: Set image details**

```powershell
$publisherName = 'MicrosoftWindowsServer'
$offerName = 'WindowsServer'
$skuName = '2022-Datacenter'
```

#### **Step 8: Define OS disk type**

```powershell
$osDiskType = 'Standard_LRS'
```

> 🔧 **Fix Applied**: Avoid using `'Standard_HDD'` — it is invalid. Use `'Standard_LRS'` instead.

#### **Step 9: Create the VM configuration**

```powershell
$vmConfig = New-AzVMConfig -VMName $vmName -VMSize $vmSize -AvailabilitySetId $availabilitySet.Id
Add-AzVMNetworkInterface -VM $vmConfig -Id $nic.Id
Set-AzVMOperatingSystem -VM $vmConfig -Windows -ComputerName $vmName -Credential $adminCreds 
Set-AzVMSourceImage -VM $vmConfig -PublisherName $publisherName -Offer $offerName -Skus $skuName -Version 'latest'
Set-AzVMOSDisk -VM $vmConfig -Name "$($vmName)_OsDisk_1_$(Get-Random)" -StorageAccountType $osDiskType -CreateOption FromImage
Set-AzVMBootDiagnostic -VM $vmConfig -Disable
Set-AzVMSecurityProfile -VM $vmConfig -SecurityType "Standard"
Register-AzProviderFeature -FeatureName UseStandardSecurityType -ProviderNamespace Microsoft.Compute
```

#### **Step 10: Deploy the VM**

```powershell
New-AzVM -ResourceGroupName $resourceGroup.ResourceGroupName -Location $location -VM $vmConfig
```

> 🕒 This may take several minutes. You can proceed to the next task while it runs in the background.

### ✅ **Result**

You have now used Azure PowerShell to deploy a second Windows Server 2022 VM into the same availability set created in Task 1. This task demonstrates command-line VM provisioning and infrastructure-as-code concepts, including network interface, NSG, public IP, and disk configuration.

---

Perfect. Let’s rewrite **Task 3** using **the Azure Portal**, just as the original lab intended — but now aligned with your improved structure, clarity, and standards.

---

## ✅ **Task 3: Deploy a Linux VM into the existing virtual network using the Azure Portal and a custom ARM template**

In this task, you’ll deploy a **Linux virtual machine** into the **existing virtual network (`azscaleset-RG-vnet`)** using a custom **ARM template** provided with the lab files.

---

### 🧭 **Objective:**

Use the **Azure Portal** to deploy an ARM template (`azuredeploy.json`) that provisions a Linux VM into your existing resource group and subnet.

---

### 📁 **Files Required:**

Ensure you have these files in your Cloud Shell file system or local machine:

* `02-vms/files/azuredeploy.json`
* `02-vms/files/azuredeploy.parameters.json`

---

### 🖥️ **Step-by-Step Instructions**

#### **Step 1: Open the Portal Template Deployment Page**

1. In the Azure Portal, search for and select **Deploy a custom template** (or go directly to **"Custom deployment"**).

   * Alternatively, browse to:
     **Home → Create a resource → Search "Template Deployment" → Select "Template deployment (deploy using custom templates)"**

#### **Step 2: Upload the ARM template**

1. Click **Build your own template in the editor**.
2. Click **Load file**, then upload `azuredeploy.json`.
3. Click **Save** at the bottom.

#### **Step 3: Fill in the deployment form**

1. **Subscription**: Use your default
2. **Resource Group**: Select `azscaleset-RG`
3. **Region**: Choose the same region used for earlier tasks (e.g., `East US`)
4. Complete the required parameters:

   * **Admin Username**: `Student`
   * **Admin Password**: `Pa55w.rd1234`
   * **VM Name**: `azscaleset-linuxvm`
   * **VM Size**: `Standard_DS1_v2`
   * **Virtual Network Name**: `azscaleset-RG-vnet`
   * **Subnet Name**: `subnet0`
   * **NIC Name**: `azlin-nic`
   * **Location**: (should auto-fill if not already provided)

> 🧠 Tip: If the template uses a parameters file, values may pre-fill automatically. Always double-check before proceeding.

#### **Step 4: Deploy**

1. Click **Review + create**
2. After validation passes, click **Create**

### ✅ **Result**

After a few minutes, the deployment will complete. You’ll have a new **Linux VM** provisioned into the existing **virtual network** and **subnet**, using your own ARM template.

To verify:

* Navigate to **Resource Group → azscaleset-RG**
* Confirm the following resources were created:

  * Linux VM (e.g., `azscaleset-linuxvm`)
  * Network Interface
  * Network Security Group
  * Public IP
  * OS Disk

### 📌 **Instructor Notes**

* The purpose of this task is to demonstrate declarative infrastructure provisioning via ARM templates using the Azure Portal.
* Students should understand that this process is **portable** — the same template can be deployed via CLI, PowerShell, or REST API.
* Make sure students recognize the **link between the template structure and the generated resources**.

---

Excellent. Let's rewrite **Exercise 2** using your improved instructional format: structured, action-driven, and consistent in tone and clarity.

---

## 🧪 **Exercise 2: Configure Networking Settings of Azure VMs Running Windows and Linux**

---

### 🎯 **Objectives**

By the end of this exercise, you will:

* Assign static private and public IP addresses to Azure VMs.
* Connect to a Windows VM using RDP over a public IP.
* Connect to a Linux VM using SSH over a private IP.

---

### 🕓 **Estimated Time:** 25 minutes

---

## ✅ **Task 1: Assign Static Private and Public IPs**

### 🔧 Configure Public IP for Windows VM

1. In the Azure portal, navigate to **azscaleset-vm0-ip**.
2. Under **Settings**, click **Configuration**.
3. Ensure the **IP address assignment** is set to **Static**.
4. Copy and record the **Public IP address** — you’ll use this in Task 2.

---

### 🔧 Configure Private IP for Linux VM

5. In the portal, go to the **azlin-vm0** blade.
6. Select **Networking** from the menu.
7. In the **Networking** blade, locate and click on the **network interface**.
8. Under **Settings**, choose **IP configurations**.
9. Click **ipconfig1**.
10. Change the **Private IP assignment** from **Dynamic** to **Static** and set the IP to:

```
10.103.0.100
```

11. Click **Save**.
12. Restart the **azlin-vm0** VM to apply the change.

> 💡 **Note:** Static IPs are useful when configuring IP-based firewall rules, DNS servers, or persistent routing.

---

## ✅ **Task 2: Connect to the Windows VM via Public IP**

### 🔐 Open RDP Port in NSG

1. From the **azscaleset-vm0** blade, go to **Networking**.
2. Under **Inbound Port Rules**, click **Add inbound port rule**.
3. Configure the rule with the following:

| Setting                 | Value                   |
| ----------------------- | ----------------------- |
| Source                  | Any                     |
| Source port ranges      | \*                      |
| Destination             | Any                     |
| Destination port ranges | 3389                    |
| Protocol                | TCP                     |
| Action                  | Allow                   |
| Priority                | 100                     |
| Name                    | AllowInternetRDPInbound |

4. Click **Add**.

---

### 🖥️ Connect via RDP

5. Go to the **Overview** blade of **azscaleset-vm0**.
6. Click **Connect**, then select **RDP**.
7. Download and open the `.rdp` file.
8. When prompted, use the following credentials:

```
Username: Student
Password: Pa55w.rd1234
```

> 🧠 **Reminder:** Case sensitivity applies.

---

## ✅ **Task 3: Connect to the Linux VM via Private IP**

1. Inside the RDP session on **azscaleset-vm0**, open **Command Prompt**.
2. Run:

   ```
   nslookup azlin-vm0
   ```
3. Confirm the name resolves to:

   ```
   10.103.0.100
   ```

> 🧠 **Note:** Azure's internal DNS resolves VM hostnames to private IPs within the same virtual network.

---

### 🔐 SSH into the Linux VM

4. In the same session, run:

   ```bash
   ssh Student@azlin-vm0
   ```
5. Enter the password when prompted:

   ```
   Pa55w.rd1234
   ```

---

### 📋 Review NSG Rules for Linux VM

6. End the RDP session.
7. In the portal, go to the **azlin-vm0** blade → **Networking**.
8. Review the NSG inbound rules.

> 💡 **Insight:** Azure allows intra-VNet traffic by default, including SSH (port 22). No additional NSG rule was required for private access.

---

## 🏁 **Exercise 2 Summary**

You have:

✅ Configured static IPs for both public and private access
✅ Connected to a Windows VM over RDP using a public IP
✅ Connected to a Linux VM over SSH using only its private IP
