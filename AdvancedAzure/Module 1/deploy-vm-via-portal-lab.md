# Deploy a VM via the Portal Interface

## Objective

In this lab, students will learn how to deploy a virtual machine (VM) in Microsoft Azure using the Azure Portal. While Infrastructure as Code (IaC) tools like Bicep or Terraform are often used in production environments, the portal offers a visual, beginner-friendly way to understand Azure resource relationships and deployment flows. This lab will guide students through deploying a VM and its essential dependencies—such as networking and storage—entirely through the "Create a virtual machine" wizard.

* Learn how to navigate the Azure Portal.
* Use the VM creation wizard to provision a VM.
* Automatically provision supporting resources (NIC, NSG, VNet, etc.).
* Understand default options selected during VM creation.

By the end of this lab, you will have deployed a functional virtual machine with all required components using the Azure Portal.

---

## Procedure

1. Log into the Azure Portal
   Open your browser and navigate to [https://portal.azure.com](https://portal.azure.com).
   Sign in with your Azure credentials.

0. Navigate to the Virtual Machines service
   In the search bar at the top of the portal, type: `Virtual Machines`
   Select the **Virtual Machines** service from the dropdown list.

0. Click **+ Create** > **Virtual Machine**
   This begins the wizard for creating a new virtual machine.

0. Configure the **Basics** tab
   Fill in the following information:

   * **Subscription**: Select your default subscription.
   * **Resource group**: Click **Create new** and name it `rg-portal-vm-<your initials>`.
   * **Virtual machine name**: `portal-demo-vm-<your initials>`
   * **Region**: Choose a region close to you (e.g., `East US`).
   * **Availability options**: Set to **No infrastructure redundancy required**.
   * **Image**: Select `Ubuntu Server 22.04 LTS -x64 Gen2`.
   * **Size**: Use the default (e.g., `Standard_D2s_v3 - 2 vcpus, 8 GiB Memory`).
   * **Authentication type**: Select `Password`.
   * **Username**: `azureuser`
   * **Password**: Provide a strong password and confirm it.
   * **Public inbound ports**: Choose `Allow selected ports`
   * **Select inbound ports**: Check `SSH (22)`

0. Leave **Disks** tab as default
   No changes needed. The OS disk will be a Standard SSD by default.

0. Review **Networking** tab
   Azure will automatically create:

   * A Virtual Network (`portal-demo-vm-<initials>-vnet`)
   * A Subnet
   * A Network Interface
   * A Network Security Group with port 22 open
     No changes are needed—keep the defaults.

0. Skip **Management**, **Advanced**, and **Tags** tabs
   Accept all default settings. Click **Review + Create** or use the tab bar to move through them.

0. On the **Review + create** tab, validate settings
   Confirm everything looks correct. You should see a **Validation passed** message.

0. Click **Create**
   Azure will begin deploying your virtual machine and its required resources.

0. Wait for deployment to complete
    When finished, click **Go to resource** to view your new VM.

---

## Mandatory Cleanup

1. Navigate to the Resource Groups service
   In the search bar at the top of the portal, type: `Resource Groups`
   Select the **Resource Groups** service from the dropdown list.

0. Open the `rg-portal-vm-<your initials>` Resource Group.
   Click `rg-portal-vm-<your initials>` Resource Group.
   This will open the Resource Group details page.

0. Delete the Resource Group.
   Click **Delete resource group**.
   This will open the Delete a Resource Group menu.

0. Confirm deletion of the Resource Group.
   Type `rg-portal-vm-<your initials>` in the **Enter Resource Group name to confirm deletion** text box.
   Click **Delete**.
   The Delete Confirmation popup window will appear.

0. Click **Delete** to confirm deletion of the Resource Group.

   > Remember, we're billed for all of the resources we create. We don't like spending money for no reason, so we'll need to clean up all of the resources we created. **The easiest way to do this is by deleting the Resource Group which contains all of our resources!**

---

## Conclusion

In this lab, you deployed a fully functional virtual machine in Azure using the Portal interface. You learned how Azure groups dependent resources—like virtual networks, public IPs, and security groups—during the deployment process. This exercise provided a visual walkthrough of Azure resource relationships, which will reinforce your understanding when transitioning to Infrastructure as Code tools like Bicep or Terraform.
