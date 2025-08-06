# **Progressive Network Deployment: VNets, Subnets, NSGs, and Peering**

## **Objective**

In this lab, learners will iteratively build a secure and connected virtual network environment using Bicep templates and the Azure CLI. Starting from a basic virtual network deployment, learners will layer in subnets, Network Security Groups (NSGs), and VNet peering—observing and verifying the changes in Azure Portal after each stage.

This progressive approach gives learners practical insight into how infrastructure components interact and evolve in a real-world, modular Infrastructure-as-Code workflow.

**Topics Discussed:**

* Authoring layered Bicep templates
* Deploying updates incrementally with `az deployment`
* Using Azure Portal to verify VNet structure and relationships
* Understanding modular development practices in Infrastructure-as-Code

**Summary:**
By the end of this lab, learners will have created, extended, and connected two VNets using multiple Bicep template deployments while verifying each stage in the Azure Portal.

## **Procedure**

> **Tip:** All of this lab will be completed in the browser-based Azure Cloud Shell. No local installation needed!

### **Task 1:** Create two VNets, each with their own Subnet using BICEP Templates.

1. **Launch Azure Cloud Shell**
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.
    - Choose **Bash** if prompted.
    - Wait for the Cloud Shell to fully initialize.

0. Create a Resource Group for your network. Replace `<your-initials>` with your **own initials** to make the name unique.

    `student1 [ ~ ]$` `az group create --name progressive-network-<your-initials> --location eastus`

0. Create a working directory and navigate into it.

    `student1 [ ~ ]$` `mkdir progressive-vnet && cd progressive-vnet`

0. **Create a basic Bicep file with only the VNets and subnets**

    `student1 [ ~/progressive-vnet ]$` `vim vnets-subnets.bicep`

    Paste the following content:

    ```bicep
    // Declare a parameter for the Azure region where resources will be deployed.
    // Default is 'eastus', but it can be overridden during deployment.
    param location string = 'eastus'
    
    // Define the first Virtual Network (vnet-alpha)
    resource vnet1 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: 'vnet-alpha'           // Name of the VNet
      location: location           // Use the location provided in the parameter
      properties: {
        addressSpace: {
          addressPrefixes: ['10.0.0.0/16']  // VNet's IP address space
        }
        subnets: [
          {
            name: 'subnet-alpha'           // Name of the subnet inside vnet-alpha
            properties: {
              addressPrefix: '10.0.1.0/24' // Subnet's IP range, must be within the VNet's address space
            }
          }
        ]
      }
    }
    
    // Define the second Virtual Network (vnet-beta)
    resource vnet2 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: 'vnet-beta'            // Name of the second VNet
      location: location           // Same location as vnet-alpha
      properties: {
        addressSpace: {
          addressPrefixes: ['10.1.0.0/16']  // Different IP range to avoid overlap with vnet-alpha
        }
        subnets: [
          {
            name: 'subnet-beta'           // Name of the subnet inside vnet-beta
            properties: {
              addressPrefix: '10.1.1.0/24' // Subnet's IP range
            }
          }
        ]
      }
    }

    ```

    > **Note:** This template creates two separate Virtual Networks (`vnet-alpha` and `vnet-beta`), each with its own subnet. It sets up basic network infrastructure without any security or connectivity between the VNets yet.

    > **Reminder:** Press `i` to enter Insert mode in Vim, and `Esc` + `:wq` to save and exit when you're done.

0. **Deploy the initial VNets and subnets**

    `student1 [ ~/progressive-vnet ]$` `az deployment group create --resource-group progressive-network-<your-initials> --template-file vnets-subnets.bicep`

    > The JSON Output detailing the provisioned resources will appear, indicating success!

#### Verify the resource in the Azure Portal.

1. Go to the [Azure Portal](https://portal.azure.com/)
0. Navigate to **Resource groups**.
0. Find and open your resource group: `progressive-network-<your-initials>`
0. Look for **vnet-alpha** and open it.
0. Click the **Properties** tab, then under **Subnets**, select `1`.
0. Verify the details of **subnet-alpha**.

> **What You’ve Accomplished So Far:**
> You’ve just deployed the foundational components of a networked environment in Azure using a Bicep template. Specifically:
>
> * Two **Virtual Networks** were created (`vnet-alpha` and `vnet-beta`)
> * Each VNet contains a **subnet** with a valid IP range
>
> At this stage, these VNets are isolated and not secured or connected. In the next steps, you’ll extend this foundation by adding **Network Security Groups (NSGs)** and then **VNet peering** to enable controlled communication between them.

---

### **Task 2:** Update Bicep template to include NSGs

1. **Return to the Azure Cloud Shell**
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.

0. Build a BICEP Template to include Network Security Groups.

    `student1 [ ~/progressive-vnet ]$` `vim vnets-subnets-nsgs.bicep`
    
    ```bicep
    param location string = 'eastus'

    // Create a Network Security Group for vnet-alpha
    resource nsg1 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      name: 'nsg-alpha'
      location: location
      properties: {} // No rules defined yet — this creates a default (empty) NSG
    }
    
    // Create a Network Security Group for vnet-beta
    resource nsg2 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      name: 'nsg-beta'
      location: location
      properties: {}
    }
    
    resource vnet1 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: 'vnet-alpha'
      location: location
      properties: {
        addressSpace: {
          addressPrefixes: ['10.0.0.0/16']
        }
        subnets: [
          {
            name: 'subnet-alpha'
            properties: {
              addressPrefix: '10.0.1.0/24'
    
              // Associate subnet-alpha with nsg-alpha
              networkSecurityGroup: {
                id: nsg1.id
              }
            }
          }
        ]
      }
    }
    
    resource vnet2 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: 'vnet-beta'
      location: location
      properties: {
        addressSpace: {
          addressPrefixes: ['10.1.0.0/16']
        }
        subnets: [
          {
            name: 'subnet-beta'
            properties: {
              addressPrefix: '10.1.1.0/24'
    
              // Associate subnet-beta with nsg-beta
              networkSecurityGroup: {
                id: nsg2.id
              }
            }
          }
        ]
      }
    }
    ```

    > **Focus for Learners:** This version adds NSGs (nsg-alpha and nsg-beta) and attaches each one to its respective subnet. While no rules are defined yet, this establishes a security boundary for each subnet and prepares them for future rule enforcement.

    > **Reminder:** Press `i` to enter Insert mode in Vim, and `Esc` + `:wq` to save and exit when you're done.

0. **Deploy updated template with NSGs.**

    `student1 [ ~/progressive-vnet ]$` `az deployment group create --resource-group progressive-network-<your-initials> --template-file vnets-subnets-nsgs.bicep`

    > The JSON Output detailing the provisioned resources will appear, indicating success! **NOTE**: This is IaC BICEP Idempotence at work! We were able to run the template which defined the same assets deployed earlier. Notice how it didn't recreate assets, or try to produce new ones? This is proof that BICEP Templating will only produce or configure assets if there are differences from previous runs.

#### Verify the resource in the Azure Portal.

1. Go to the [Azure Portal](https://portal.azure.com/)
0. Navigate to **Resource groups**.
0. Find and open your resource group: `progressive-network-<your-initials>`
0. Look for **nsg-alpha** and open it.
0. Verify the details of **nsg-alpha**.

> **What You’ve Accomplished So Far:**
> In this task, you extended your existing Bicep template to include **Network Security Groups (NSGs)** for both subnets. You also associated each NSG with its respective subnet:
>
> * `nsg-alpha` was attached to `subnet-alpha`
> * `nsg-beta` was attached to `subnet-beta`
>
> This adds a layer of security control to your virtual networks, even though no explicit rules have been defined yet. You also witnessed **Bicep’s idempotent behavior** — reapplying the template did not recreate or duplicate existing resources. Azure only made the necessary updates (adding the NSGs), showcasing the power and safety of Infrastructure-as-Code.

### **Task 3:** Extend the Bicep template to include VNet Peering

1. **Return to the Azure Cloud Shell**
    - In the [Azure Portal](https://portal.azure.com/), click the **Cloud Shell** icon at the top right.

0. Build a BICEP Template to include VNet Peering.

    `student1 [ ~/progressive-vnet ]$` `vim vnets-subnets-nsgs-peering.bicep`

    ```bicep
    param location string = 'eastus'

    // Previously created NSG for vnet-alpha
    resource nsg1 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      name: 'nsg-alpha'
      location: location
      properties: {}
    }
    
    // Previously created NSG for vnet-beta
    resource nsg2 'Microsoft.Network/networkSecurityGroups@2023-04-01' = {
      name: 'nsg-beta'
      location: location
      properties: {}
    }
    
    // Existing VNet vnet-alpha with subnet and NSG association
    resource vnet1 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: 'vnet-alpha'
      location: location
      properties: {
        addressSpace: {
          addressPrefixes: ['10.0.0.0/16']
        }
        subnets: [
          {
            name: 'subnet-alpha'
            properties: {
              addressPrefix: '10.0.1.0/24'
              networkSecurityGroup: {
                id: nsg1.id
              }
            }
          }
        ]
      }
    }
    
    // Existing VNet vnet-beta with subnet and NSG association
    resource vnet2 'Microsoft.Network/virtualNetworks@2023-04-01' = {
      name: 'vnet-beta'
      location: location
      properties: {
        addressSpace: {
          addressPrefixes: ['10.1.0.0/16']
        }
        subnets: [
          {
            name: 'subnet-beta'
            properties: {
              addressPrefix: '10.1.1.0/24'
              networkSecurityGroup: {
                id: nsg2.id
              }
            }
          }
        ]
      }
    }
    
    // NEW: VNet Peering from vnet-alpha to vnet-beta
    resource peer1to2 'Microsoft.Network/virtualNetworks/virtualNetworkPeerings@2023-04-01' = {
      name: 'peer-alpha-to-beta'
      parent: vnet1 // This peering is defined from the perspective of vnet-alpha
      properties: {
        remoteVirtualNetwork: {
          id: vnet2.id // Link to vnet-beta
        }
        allowVirtualNetworkAccess: true // Enables traffic between VNets
      }
    }
    
    // NEW: VNet Peering from vnet-beta to vnet-alpha
    resource peer2to1 'Microsoft.Network/virtualNetworks/virtualNetworkPeerings@2023-04-01' = {
      name: 'peer-beta-to-alpha'
      parent: vnet2 // This peering is defined from the perspective of vnet-beta
      properties: {
        remoteVirtualNetwork: {
          id: vnet1.id // Link to vnet-alpha
        }
        allowVirtualNetworkAccess: true // Enables traffic between VNets
      }
    }

    ```

    > **Summary of the Change:** The new `peer1to2` and `peer2to1` resources create a bidirectional VNet peering connection, allowing vnet-alpha and vnet-beta to communicate with each other. Each side must explicitly define the peering for connectivity to be established in both directions. This is the first step in enabling network connectivity between the previously isolated networks.

    > **Reminder:** Press `i` to enter Insert mode in Vim, and `Esc` + `:wq` to save and exit when you're done.

0. **Deploy the initial VNets and subnets**

    `student1 [ ~/progressive-vnet ]$` `az deployment group create --resource-group progressive-network-<your-initials> --template-file vnets-subnets-nsgs-peering.bicep`

    > The JSON Output detailing the provisioned resources will appear, indicating success!

#### Verify the resource in the Azure Portal.

1. Go to the [Azure Portal](https://portal.azure.com/)
0. Navigate to **Resource groups**.
0. Find and open your resource group: `progressive-network-<your-initials>`
0. Look for **vnet-alpha** and open it.
0. Click the **Properties** tab, then under **Peerings**, select `peer-alpha-to-beta`.
0. Verify the details.

> **What You’ve Accomplished:**
> You’ve completed the full progression of deploying a secure and connected Azure network environment using **Bicep templates** and the **Azure CLI**. Specifically:
>
> * You created two Virtual Networks (`vnet-alpha` and `vnet-beta`) with their own subnets.
> * You added **Network Security Groups (NSGs)** to each subnet, preparing for traffic control and protection.
> * You established **bidirectional VNet peering**, enabling communication between the two networks.
>
> Along the way, you used **incremental Bicep deployments** to safely evolve your infrastructure — a foundational skill for modern Infrastructure-as-Code workflows in Azure.

> **You now have a reusable, modular Bicep deployment that mirrors real-world enterprise networking patterns.**

### Clean up your resources

1. To avoid charges, delete your entire resource group:

    `student1 [ ~/bicep-demo ]$` `az group delete --name progressive-network-<your-initials> --yes --no-wait`

    > This will delete the **VNet** and any other resources in the group.

## **Conclusion**

This progressive lab demonstrated how to incrementally construct an Azure network architecture using modular Bicep templates. Learners began with a foundational VNet and subnet deployment, then added NSGs, and concluded with bi-directional VNet peering. Each stage reinforced best practices in Infrastructure-as-Code and provided immediate feedback via Azure Portal inspection.

This modular approach allows for better testing, reuse, and maintainability of production-grade infrastructure code.
