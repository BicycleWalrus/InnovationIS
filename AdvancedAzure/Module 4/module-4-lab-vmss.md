# **Progressive Deployment: VM Scale Set with Autoscale and Diagnostics using Bicep**

## **Objective**

In this lab, learners will progressively build and deploy a Virtual Machine Scale Set (VMSS) using Bicep templates and Azure CLI from within the Azure Cloud Shell. Starting from infrastructure provisioning, learners will incrementally configure autoscaling and enable diagnostics logging to a dedicated storage account.

Each stage will be deployed through modular Bicep files, allowing learners to observe the evolving configuration both via the Azure CLI and the Azure Portal.

**Topics Discussed:**

* Authoring and deploying modular Bicep templates
* VM Scale Set configuration using platform images
* Implementing autoscaling based on CPU thresholds
* Enabling and validating boot diagnostics in Azure
* Verifying deployments and configurations in Azure Portal

**Summary:**
By the end of this lab, learners will have deployed and validated a complete VM Scale Set solution using Bicep, including autoscaling behavior and diagnostics logging.

---

## **Procedure**

> **Tip:** You’ll complete this entire lab using **Azure Cloud Shell (Bash)** — no local setup required!

### **Task 1: Provision Core Infrastructure Using Bicep**

1. **Launch Azure Cloud Shell**
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.
    - Choose **Bash** if prompted.
    - Wait for the Cloud Shell to fully initialize.

0. Create a Resource Group for your storage account. Replace `<your-initials>` with your **own initials** to make the name unique.

    `student1 [ ~ ]$` `az group create --name progressive-vmss-<your-initials> --location eastus`

0. Create a working directory and navigate into it.

    `student1 [ ~ ]$` `mkdir progressive-vmss && cd progressive-vmss`

0. **Create a Bicep file to deploy VMSS infrastructure**.

    `student1 [ ~/progressive-vmss ]$` `vim vmss-core.bicep`

    Paste the following content:

     ```bicep
    // Set default deployment location (can be overridden during deployment)
    param location string = 'eastus'
    
    // Set a name prefix for resources (used for naming VNet and NSG)
    param vmssName string = 'myScaleSet'
    
    // Create a Virtual Network (VNet) with a single subnet
    resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      // The name of the VNet will be something like "myScaleSet-vnet"
      name: '${vmssName}-vnet'
      location: location
      properties: {
        addressSpace: {
          // Define the overall IP address range for the VNet
          addressPrefixes: ['10.0.0.0/16']
        }
        subnets: [
          {
            // Create a subnet named "default" within the VNet
            name: 'default'
            properties: {
              // Subnet IP range (must be within the VNet range above)
              addressPrefix: '10.0.0.0/24'
            }
          }
        ]
      }
    }
    
    // Create a Network Security Group (NSG) with a rule to allow SSH
    resource nsg 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      // The NSG name will be "myScaleSet-nsg"
      name: '${vmssName}-nsg'
      location: location
      properties: {
        securityRules: [
          {
            // Define a security rule to allow SSH access (TCP port 22)
            name: 'AllowSSH'
            properties: {
              priority: 1000                      // Rule priority (lower number = higher priority)
              direction: 'Inbound'               // Applies to incoming traffic
              access: 'Allow'                    // Action: Allow the traffic
              protocol: 'Tcp'                    // Applies to TCP protocol
              sourcePortRange: '*'               // Accept from any source port
              destinationPortRange: '22'         // Allow only port 22 (SSH)
              sourceAddressPrefix: '*'           // Allow from any IP address
              destinationAddressPrefix: '*'      // Apply to all resources in subnet
            }
          }
        ]
      }
    }
    ```

   > This file sets up core networking infrastructure for a VM Scale Set deployment. **It provisions:**
   > - A VNet with a defined address space and a single subnet.
   > - A Network Security Group (NSG) with a rule to allow SSH access, which is critical for connecting to VM instances.
   > - Each resource name is dynamically constructed using the vmssName parameter, ensuring naming consistency across deployments.

0. **Deploy the base infrastructure**

    `student1 [ ~/progressive-vmss ]$` `az deployment group create --resource-group progressive-vmss-<your initials> --template-file vmss-core.bicep`

    > The JSON Output detailing the provisioned resources will appear, indicating success!

#### Verify in Azure Portal

1. Navigate to **Resource groups > vmss-lab-<your-initials>**
0. Confirm the presence of:
    * Virtual Network (`myScaleSet-vnet`)
    * Network Security Group (`myScaleSet-nsg`)

> ### **What You’ve Accomplished So Far**
>
> In this task, you successfully deployed the **foundational infrastructure** for your Virtual Machine Scale Set using a Bicep template. Specifically:
> * You created a **Virtual Network (VNet)** named `myScaleSet-vnet` with a defined IP address space and a subnet named `default`.
> * You provisioned a **Network Security Group (NSG)** named `myScaleSet-nsg`, which includes an inbound rule allowing SSH (port 22) traffic.
> * You used **parameterized Bicep code**, enabling reusable and consistent naming across resources.
>
> You also deployed everything using the **Azure CLI in Cloud Shell**, reinforcing the importance of Infrastructure-as-Code and CLI-driven workflows.
>
> This sets the stage for deploying a Virtual Machine Scale Set in a secure, well-defined network environment in the next steps.


### **Task 2: Add VM Scale Set with Diagnostics Enabled**

1. **Return to the Azure Cloud Shell**.
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.

0. **Create a new Bicep file for VMSS with diagnostics**.

    `student1 [ ~/progressive-vmss ]$` `vim vmss-diagnostics.bicep`

    ```bicep
    // Set default deployment location
    param location string = 'eastus'
    
    // Consistent naming prefix for all related resources
    param vmssName string = 'myScaleSet'
    
    // Admin username for VM instances
    @minLength(1)
    @maxLength(32)
    @description('Admin username for the VM instances.')
    param adminUsername string
    
    // Secure admin password
    @secure()
    @description('Admin password for the VM instances.')
    param adminPassword string
    
    //
    // ────────────────────────────
    // NETWORKING COMPONENTS
    // ────────────────────────────
    //
    
    // Create a Virtual Network (VNet) with a single subnet
    resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: '${vmssName}-vnet'
      location: location
      properties: {
        addressSpace: {
          addressPrefixes: ['10.0.0.0/16']
        }
        subnets: [
          {
            name: 'default'
            properties: {
              addressPrefix: '10.0.0.0/24'
            }
          }
        ]
      }
    }
    
    // Create a Network Security Group (NSG) with a rule to allow SSH
    resource nsg 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      name: '${vmssName}-nsg'
      location: location
      properties: {
        securityRules: [
          {
            name: 'AllowSSH'
            properties: {
              priority: 1000
              direction: 'Inbound'
              access: 'Allow'
              protocol: 'Tcp'
              sourcePortRange: '*'
              destinationPortRange: '22'
              sourceAddressPrefix: '*'
              destinationAddressPrefix: '*'
            }
          }
        ]
      }
    }
    
    //
    // ────────────────────────────
    // DIAGNOSTICS STORAGE
    // ────────────────────────────
    //
    
    // Create a Storage Account for Boot Diagnostics
    resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
      name: '${vmssName}diag'
      location: location
      sku: {
        name: 'Standard_LRS'
      }
      kind: 'StorageV2'
    }
    
    //
    // ────────────────────────────
    // VIRTUAL MACHINE SCALE SET
    // ────────────────────────────
    //
    
    resource vmss 'Microsoft.Compute/virtualMachineScaleSets@2023-03-01' = {
      name: vmssName
      location: location
      sku: {
        name: 'Standard_DS1_v2'
        capacity: 2
        tier: 'Standard'
      }
      properties: {
        upgradePolicy: {
          mode: 'Manual'
        }
        virtualMachineProfile: {
          storageProfile: {
            imageReference: {
              publisher: 'Canonical'
              offer: 'UbuntuServer'
              sku: '18.04-LTS'
              version: 'latest'
            }
          }
          osProfile: {
            computerNamePrefix: 'vmss'
            adminUsername: adminUsername
            adminPassword: adminPassword
          }
          diagnosticsProfile: {
            bootDiagnostics: {
              enabled: true
              storageUri: storage.properties.primaryEndpoints.blob
            }
          }
          networkProfile: {
            networkInterfaceConfigurations: [
              {
                name: '${vmssName}-nic'
                properties: {
                  primary: true
                  ipConfigurations: [
                    {
                      name: '${vmssName}-ipconfig'
                      properties: {
                        subnet: {
                          id: vnet.properties.subnets[0].id
                        }
                      }
                    }
                  ]
                  networkSecurityGroup: {
                    id: nsg.id
                  }
                }
              }
            ]
          }
        }
      }
    }
    ```

   > **What this adds:** A VMSS using the Ubuntu image, with boot diagnostics enabled and output stored in a new Storage Account.

0. **Deploy the updated infrastructure**

   `student1 [ ~/progressive-vmss ]$`

    ```
    az deployment group create \
      --resource-group progressive-vmss-<your-initials> \
      --template-file vmss-diagnostics.bicep \
      --parameters vmssName='vmsslabtest' \
                   adminUsername='vmssadmin' \
                   adminPassword='MyP@ssw0rd123!'
    ```



#### Verify in Azure Portal

1. Go to your resource group in the Azure Portal
2. Confirm presence of:
   * **vmsslabtest** (Virtual Machine Scale Set)
   * **vmsslabtestdiag** (Storage Account)
3. Select **vmsslabtest** → Then under **Management, verify that **Boot Diagnostics** is **Enabled**.

> **Checkpoint:** You now have a fully deployed VMSS with diagnostics configured and visible in the Azure Portal.

---

### **Task 3: Add Autoscale Settings to VMSS**

1. **Return to the Azure Cloud Shell**.
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.

0. **Create a new Bicep file for VMSS with Autoscaling**.

    `student1 [ ~/progressive-vmss ]$` `vim vmss-autoscale.bicep`

    ```bicep
    // Set default deployment location
    param location string = 'eastus'
    
    // Consistent naming prefix for all related resources
    param vmssName string = 'myScaleSet'
    
    // Admin username for VM instances
    @minLength(1)
    @maxLength(32)
    @description('Admin username for the VM instances.')
    param adminUsername string
    
    // Secure admin password
    @secure()
    @description('Admin password for the VM instances.')
    param adminPassword string
    
    //
    // ────────────────────────────
    // NETWORKING COMPONENTS
    // ────────────────────────────
    //
    
    resource vnet 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: '${vmssName}-vnet'
      location: location
      properties: {
        addressSpace: {
          addressPrefixes: ['10.0.0.0/16']
        }
        subnets: [
          {
            name: 'default'
            properties: {
              addressPrefix: '10.0.0.0/24'
            }
          }
        ]
      }
    }
    
    resource nsg 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      name: '${vmssName}-nsg'
      location: location
      properties: {
        securityRules: [
          {
            name: 'AllowSSH'
            properties: {
              priority: 1000
              direction: 'Inbound'
              access: 'Allow'
              protocol: 'Tcp'
              sourcePortRange: '*'
              destinationPortRange: '22'
              sourceAddressPrefix: '*'
              destinationAddressPrefix: '*'
            }
          }
        ]
      }
    }
    
    //
    // ────────────────────────────
    // DIAGNOSTICS STORAGE
    // ────────────────────────────
    //
    
    resource storage 'Microsoft.Storage/storageAccounts@2023-01-01' = {
      name: '${vmssName}diag'
      location: location
      sku: {
        name: 'Standard_LRS'
      }
      kind: 'StorageV2'
    }
    
    //
    // ────────────────────────────
    // VIRTUAL MACHINE SCALE SET
    // ────────────────────────────
    //
    
    resource vmss 'Microsoft.Compute/virtualMachineScaleSets@2023-03-01' = {
      name: vmssName
      location: location
      sku: {
        name: 'Standard_DS1_v2'
        capacity: 2
        tier: 'Standard'
      }
      properties: {
        upgradePolicy: {
          mode: 'Manual'
        }
        virtualMachineProfile: {
          storageProfile: {
            imageReference: {
              publisher: 'Canonical'
              offer: 'UbuntuServer'
              sku: '18.04-LTS'
              version: 'latest'
            }
          }
          osProfile: {
            computerNamePrefix: 'vmss'
            adminUsername: adminUsername
            adminPassword: adminPassword
          }
          diagnosticsProfile: {
            bootDiagnostics: {
              enabled: true
              storageUri: storage.properties.primaryEndpoints.blob
            }
          }
          networkProfile: {
            networkInterfaceConfigurations: [
              {
                name: '${vmssName}-nic'
                properties: {
                  primary: true
                  ipConfigurations: [
                    {
                      name: '${vmssName}-ipconfig'
                      properties: {
                        subnet: {
                          id: vnet.properties.subnets[0].id
                        }
                      }
                    }
                  ]
                  networkSecurityGroup: {
                    id: nsg.id
                  }
                }
              }
            ]
          }
        }
      }
    }
    
    //
    // ────────────────────────────
    // AUTOSCALE SETTINGS
    // ────────────────────────────
    //
    
    resource autoscale 'Microsoft.Insights/autoscalesettings@2022-10-01' = {
      name: '${vmssName}-autoscale'
      location: location
      properties: {
        enabled: true
        targetResourceUri: vmss.id
        profiles: [
          {
            name: 'AutoScaleProfile'
            capacity: {
              minimum: '1'
              maximum: '5'
              default: '2'
            }
            rules: [
              {
                metricTrigger: {
                  metricName: 'Percentage CPU'
                  timeGrain: 'PT1M'
                  statistic: 'Average'
                  timeWindow: 'PT5M'
                  timeAggregation: 'Average'
                  operator: 'GreaterThan'
                  threshold: 75
                  metricResourceUri: vmss.id
                }
                scaleAction: {
                  direction: 'Increase'
                  type: 'ChangeCount'
                  value: '1'
                  cooldown: 'PT5M'
                }
              },
              {
                metricTrigger: {
                  metricName: 'Percentage CPU'
                  timeGrain: 'PT1M'
                  statistic: 'Average'
                  timeWindow: 'PT10M'
                  timeAggregation: 'Average'
                  operator: 'LessThan'
                  threshold: 25
                  metricResourceUri: vmss.id
                }
                scaleAction: {
                  direction: 'Decrease'
                  type: 'ChangeCount'
                  value: '1'
                  cooldown: 'PT10M'
                }
              }
            ]
          }
        ]
      }
    }
    ```

    > **Purpose:** This sets up autoscale rules to:
    >
    > * Increase VM count if CPU > 75% for 5 minutes
    > * Decrease VM count if CPU < 25% for 10 minutes

3. **Deploy the autoscale configuration**

    `student1 [ ~/progressive-vmss ]$`

    ```
    az deployment group create \
      --resource-group progressive-vmss-<your-initials> \
      --template-file vmss-autoscale.bicep \
      --parameters vmssName='vmsslabtest' \
                   adminUsername='vmssadmin' \
                   adminPassword='MyP@ssw0rd123!'
    ```

    > ### What You’ve Just Deployed
    >
    > You’ve successfully deployed a **fully configured Virtual Machine Scale Set (VMSS)** architecture using a single, unified Bicep template. This included:
    >
    > * A **Virtual Network** with a subnet (`default`) for hosting the VMSS instances
    > * A **Network Security Group** with an inbound rule to allow SSH (port 22)
    > * A **Storage Account** for storing **boot diagnostics** (logs and screenshots)
    > * A **VM Scale Set** using the latest Ubuntu image, with 2 initial instances
    > * An **Autoscale profile** that automatically increases or decreases the number of instances based on CPU usage
    >
    > This deployment showcases the power of **modular Infrastructure-as-Code (IaC)** using Bicep, enabling you to declaratively provision scalable, secure, and observable infrastructure in Azure.

#### Verify in Azure Portal

1. Open your VM Scale Set resource
0. Navigate to **Availability + scaling** under **Properties** tab.
0. Validate:
   * **Default instance count** is 2
   * **Autoscale rules** match the defined thresholds

---

### Clean up your resources

1. To avoid charges, delete your entire resource group:

    `student1 [ ~/progressive-vmss ]$` `az group delete --name progressive-vmss-<your-initials> --yes --no-wait`

    > This will delete the **VMSS** and any other resources in the group.

---

## **Conclusion**

In this progressive lab, you incrementally built and deployed a robust VM Scale Set architecture using modular Bicep templates. Starting from the network foundation, you added diagnostics and then autoscale settings. At each stage, you verified the configuration in the Azure Portal, reinforcing best practices in infrastructure-as-code and observability.

This lab demonstrates a real-world pattern of deploying scalable, observable infrastructure using Bicep and the Azure CLI — tools essential for any modern DevOps or Cloud Engineer.
