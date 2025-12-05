---
title: "Secure SSH Access to Azure VMs Using Microsoft Entra Private Access: A Cost-Effective Alternative to Azure Bastion"
date: 2025-12-05 11:00:00 +0100
categories: [Microsoft, Entra] 
tags: [microsoft,entra,private access,gsa, global secure access]
image:
  path: /assets/img/20251205/Entra_Private_Access_Secure_SSH_CyrusAcademy.png
  alt: Secure SSH Access to Azure VMs Using Microsoft Entra Private Access A Cost-Effective Alternative to Azure Bastion
---

# Secure SSH Access to Azure VMs Using Microsoft Entra Private Access: A Cost-Effective Alternative to Azure Bastion

Small and medium-sized businesses (SMBs) often face challenges managing secure and efficient remote access to Azure Virtual Machines (VMs). Azure Bastion provides a robust solution but can be costly and complex, especially for organizations managing multiple VMs across different Virtual Networks (VNets). **Microsoft Entra Private Access**, combined with **Entra ID Application Proxy** and **Windows 365** Cloud PCs, presents a budget-friendly and scalable alternative.

## Why Entra Private Access?

Entra Private Access leverages your existing Entra ID infrastructure, significantly reducing cost and complexity. It avoids additional infrastructure investments like Azure Bastion or a dedicated management network.

## Real-world Scenario: SMB Case Study

An SMB needed secure SSH access to multiple Azure Linux VMs distributed across various VNets. Azure Bastion, while effective, proved too expensive and cumbersome. They turned to Microsoft Entra Private Access, leveraging Entra ID Application Proxy to securely route connections from external developers and internal IT personnel.

### Architecture Overview

The proposed architecture involves:

- Microsoft Entra ID for identity management.
- Entra ID Application Proxy connector installed within the Azure infrastructure.
- Windows 365 Cloud PCs provisioned and managed by Microsoft Intune.
- Azure CLI with the SSH module installed via Intune.

### Step-by-Step Implementation

**Step 1: Setting Up Entra ID Application Proxy**

- Install Application Proxy connector on a Windows Server within Azure.
- Register the applications in Entra ID and configure secure external access.

**Step 2: Provisioning and Configuring Windows 365 Cloud PCs**

- Provision Cloud PCs for remote workers.
- Ensure devices are joined to Entra ID and managed via Intune.

**Step 3: Deploy Azure CLI and SSH Module via Intune**

Use this PowerShell script deployed via Intune:
```powershell
Set-ExecutionPolicy RemoteSigned -Scope Process -Force

$installerUrl = "<https://aka.ms/installazurecliwindows>"

$installerPath = "$env:TEMP\\AzureCLI.msi"

Invoke-WebRequest -Uri $installerUrl -OutFile $installerPath

Start-Process msiexec.exe -ArgumentList "/I $installerPath /quiet" -Wait

Remove-Item -Path $installerPath -Force

$env:Path += ";C:\\Program Files\\Microsoft SDKs\\Azure\\CLI2\\wbin"

az extension add --name ssh
```
**Step 4: SSH Connection via Azure CLI Using Private IP**

To ensure SSH connections use the private IP, particularly crucial for VMs protected by Azure Front Door or those publicly exposed but protected, execute:
```powershell
az ssh vm --name &lt;vm-name&gt; --resource-group &lt;resource-group-name&gt; --prefer-private-ip
```
Using the --prefer-private-ip parameter ensures SSH traffic bypasses the public IP, preventing connectivity issues associated with Azure Front Door and similar services.

### Assigning Least Privilege Permissions

For external users requiring SSH access, create a custom RBAC role:
```json
{

"Name": "Network Reader for SSH Access",

"Description": "Allows read access to network interfaces and public IP addresses for SSH authentication.",

"Actions": \[

"Microsoft.Network/networkInterfaces/read",

"Microsoft.Network/publicIPAddresses/read"

\],

"NotActions": \[\],

"AssignableScopes": \[

"/subscriptions/{subscription-id}/resourceGroups/{resource-group-name}"

\]

}
```
Assign this role narrowly to maintain the principle of least privilege.

## Advantages of this Approach

- **Cost-Effective:** Utilizes existing Entra infrastructure without additional licensing or resources.
- **Enhanced Security:** Leverages secure authentication mechanisms integrated into Entra ID.
- **Scalable:** Easily adapts to growing infrastructure needs without significant overhead.

## Conclusion

Entra Private Access combined with Entra ID Application Proxy, Windows 365 Cloud PCs, and Azure CLI provides SMBs with a secure, efficient, and cost-effective method to manage SSH access to Azure VMs. This approach addresses common SMB constraints, offering a robust alternative to more expensive and complex solutions.

## Further Reading

- [Microsoft Entra Private Access Documentation](https://learn.microsoft.com/en-us/entra/global-secure-access/concept-private-access)
- [Entra ID Application Proxy Setup](https://learn.microsoft.com/en-us/entra/identity/app-proxy/overview-what-is-app-proxy)
- [Azure CLI SSH Extension](https://learn.microsoft.com/en-us/cli/azure/ssh)