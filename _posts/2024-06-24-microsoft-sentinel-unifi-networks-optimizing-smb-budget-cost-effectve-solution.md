---
title: "Optimizing SMBs Cybersecurity on a Budget: How Microsoft Sentinel and Unifi Networks Deliver Cost-Effective Solutions [PoC]"
date: 2024-06-24 00:31:00 +0100
categories: [Microsoft, Security] 
tags: [microsoft, sentinel,xdr,unifi,network,siem,cybersecurity, finops]
---

# This is why SMBs should activate Microsoft Sentinel

Recently, I was told that Microsoft Sentinel is built and aimed at massive companies, and that SMBs should look towards open-source solutions or leverage their on-premise vendors for good deals on SIEM. As this narrative seemed too common, I decided to challenge it by conducting a PoC (Proof of Concept) directly in production on an old client's tenant. I even promised to cover the costs myself if Sentinel expenses surpassed €5 monthly. This company, with fewer than 30 employees, utilizes Microsoft E5 licenses for everyone, so I think my fellow experts reading this article can already guess the outcome (you silly sausages).

For added fun, the scenario involved ingesting their on-premise network logs into Sentinel. This was to uncover all kinds of blind spots and correlate Microsoft Defender XDR logs with on-site activities.

## The Technology Stack

### Microsoft Sentinel and Defender XDR

**Microsoft Sentinel** is a cloud-native SIEM (Security Information and Event Management) system offering intelligent security analytics and threat intelligence across an enterprise. It's designed as a scalable, flexible, and cost-effective security solution suitable for any business size, including SMBs.

**Microsoft Defender XDR** (Extended Detection and Response) integrates with Sentinel, providing comprehensive threat detection and response capabilities across endpoints, identities, email, and applications.

### Unifi Network and Syslog Server

The client has a Unifi network setup, which includes routers and switches. We established a VM in Azure to serve as a Syslog server, collecting logs from these devices and forwarding them to the Sentinel Log Analytics workspace.

## Implementation Steps

1. **Setting Up the Syslog Server**: 
   We deployed a lightweight Linux VM in Azure and installed `rsyslog`. This server was configured to receive logs from the Unifi devices on the network.

    ```bash
   sudo apt-get update
   sudo apt-get install rsyslog
   sudo nano /etc/rsyslog.conf
    ```

    Add the following configuration to `/etc/rsyslog.conf`:
    ```bash
   module(load="imudp")
   input(type="imudp" port="514")
   *.* /var/log/unifi.log
    ```
   
   Restart the Syslog service:
    ```bash
   sudo service rsyslog restart
    ```

    I used this YouTube video to help me:

    [![Video Title](https://img.youtube.com/vi/V_iWTcAOQb4/0.jpg)](https://www.youtube.com/watch?v=V_iWTcAOQb4 "Microsoft Sentinel & Unifi")


2. **Configuring Unifi Devices**:
   Configure the Unifi devices to send their Syslog data to the IP address of the Azure VM. This typically involves accessing the network settings on each device, where you can specify the Syslog server's IP address and the port (in this case, port 514).

3. **Connecting the VM to Sentinel**:
   Set up the Azure VM to forward logs to an Azure Log Analytics workspace configured with Microsoft Sentinel. This step requires installing the Log Analytics agent on the VM to facilitate the transfer of logs.

   ```bash
   wget https://raw.githubusercontent.com/Microsoft/OMS-Agent-for-Linux/master/installer/scripts/onboard_agent.sh
   sudo sh onboard_agent.sh -w <Workspace ID> -s <Primary Key>
   ```

4. **Creating Data Connectors**:
   Within the Sentinel workspace, create data connectors for various sources:
   - Microsoft Defender for Cloud Apps
   - Microsoft Defender for Endpoint
   - Microsoft Defender for Identity
   - Microsoft Defender for Office 365
   - Microsoft Defender XDR
   - Microsoft Entra ID
   - Microsoft Entra ID Protection

   This ensures that logs from these sources are ingested into Sentinel, allowing for comprehensive threat detection and response.

   ![Desktop View]('/assets/img/Microsoft Sentinel UniFi data connector.png'){: .left .shadow}

## Cost Analysis

### Data Ingestion and E5 Grants
Microsoft 365 E5 licenses come with a data grant of up to 5 MB per user per day for certain data types, including Azure AD logs, Defender logs, and advanced hunting data. With 14 users, the total grant is 70 MB per day. Given our ingestion rate of around 60 MB per day, this covers all our ingestion needs under the grant, resulting in zero additional ingestion costs.

### Storage Costs
The primary costs outside of Sentinel itself come from storage, particularly Standard SSD Managed Disks, and the use of a virtual network. However, these costs are manageable with careful planning and resource allocation.

## Conclusion

By using Microsoft Sentinel in conjunction with Microsoft Defender XDR, we were able to set up a cost-effective and comprehensive security monitoring solution for a small business. Despite the initial perception that Sentinel is only for large enterprises, this PoC demonstrated that it can be effectively used by SMBs, especially when leveraging existing Microsoft 365 E5 licenses.

In the end, the total cost for the client was less than €5 per month, and the only additional expense was for the Unifi log ingestion, which came in at less than €1 a month. This setup not only provides robust security monitoring but also ensures that costs remain extremely affordable. So, next time someone says Microsoft Sentinel isn't for SMBs, you can point them to this real-world example and say, "Challenge accepted!"

