---
title: "A Nuclear weapons story: Making SharePoint Zero-Day-Tolerant with Microsoft Entra, Microsoft Sentinel, & Security Copilot"
date: 2025-08-08 16:45:00 +0100
categories: [Microsoft, Entra, Sentinel, Copilot] 
tags: [microsoft,Entra,Sentinel,Copilot]
image:
  path: /assets/img/20250808/A_Nuclear_weapons_story_SharePoint.png
  alt: MA Nuclear weapons story - Making SharePoint Zero-Day-Tolerant with Microsoft Entra, Microsoft Sentinel, & Security Copilot
---

# A Nuclear weapons story: Making SharePoint Zero-Day-Tolerant with Microsoft Entra, Microsoft Sentinel, & Security Copilot

A SharePoint zero-day went from stage demo to real-world disaster after details likely leaked from Microsoft’s partner early-alert program (MAPP). Hundreds of **internet-facing** on-prem servers were hit. Fixes arrived, but the first patches didn’t fully close the door. The cure isn’t “patch faster” alone — it’s **pre-auth at the edge**, **AMSI in Full Mode**, key rotation, tight egress, and **better supply-chain trust**. And yes: **SharePoint Online wasn’t impacted**; **Microsoft 365 Local** can help if you deploy it with zero-trust controls. [MacGeneration](https://www.macg.co/ailleurs/2025/07/la-faille-ayant-mis-sharepoint-terre-viendrait-dun-partenaire-de-securite-informatique-de-microsoft-302765) [Microsoft](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/) [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)

## Introduction

In **May 2025**, on a stage in **Berlin**, a Vietnamese researcher named **Dinh Ho Anh Khoa** from **Viettel Cyber Security** pulled off a clean, surgical hack against **Microsoft SharePoint** at **Pwn2Own Berlin 2025**. His chain was elegant and scary: an **authentication bypass** feeding an **insecure deserialization** to get **remote code execution**, no valid user needed. The demo earned a **$100,000** prize and a name for the exploit path: **“ToolShell.”** [zerodayinitiative.com](https://www.zerodayinitiative.com/blog/2025/5/16/pwn2own-berlin-2025-day-two-results) [BleepingComputer](https://www.bleepingcomputer.com/news/security/hackers-exploit-vmware-esxi-microsoft-sharepoint-zero-days-at-pwn2own/)

From there, the clock started. Microsoft worked on patches while its long-running **Microsoft Active Protection Program (MAPP)** prepared selected security partners with pre-release details so defenders could get ahead. But a troubling timeline emerged: partners were briefed on **June 24**, **July 3**, and **July 7**—and **exploitation began on July 7**, right as that last briefing went out. The **July Patch Tuesday** updates followed, but the initial fix didn’t fully close the door, and attackers sprinted through the gap. Microsoft later tied activity to **Linen Typhoon**, **Violet Typhoon**, and **Storm-2603**. [Reuters](https://www.reuters.com/technology/microsoft-probing-if-chinese-hackers-learned-sharepoint-flaws-through-alert-2025-07-25/) [Microsoft](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/)

What made this so effective? Most victims were **internet-facing, on-premises SharePoint servers**—the kind of endpoints you can poke from anywhere. Once inside, attackers dropped **web shells** (think of them as back doors—e.g., files like spinstall0.aspx) and used the server’s **IIS worker process** to launch living-off-the-land commands. Microsoft’s guidance that followed was blunt: **patch to the latest cumulative update**, enable **AMSI with Microsoft Defender Antivirus**, **rotate ASP.NET machine keys**, and **restart IIS**—then **hunt** for artifacts and suspicious process chains. Importantly, **SharePoint Online** in **Microsoft 365** wasn’t affected; the trouble sat squarely with **on-prem** instances exposed to the public internet. [Microsoft](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/) [msrc.microsoft.com](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)

That brings us to the point of this article: _patching isn’t enough_. If we want on-prem workloads to survive the next unknown bug, they need to be **zero-day-tolerant by design**. My approach leans on three pillars you likely already own:

- **Microsoft Entra** — publish SharePoint **behind pre-authentication** (Entra **Application Proxy** or **Private Access**) so **every** request clears identity, device, and risk checks **before** it touches SharePoint.
- **Microsoft Sentinel** — keep the **right logs for long enough** and hunt with **KQL** so post-exploitation sticks out in minutes, not weeks.
- **Security Copilot** — accelerate triage and response, turning hunts and incidents into guided, repeatable playbooks for humans.

This isn’t theory. It’s how we make even legacy, internet-published apps **resilient** when the next “ToolShell” shows up. And yes—later in this piece I’ll give you **copy-paste KQL**, a **hardening checklist**, and **automation ideas** so you can put it in place today. [Microsoft](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/)

If you only remember one line, make it this: **Put SharePoint behind Entra pre-auth, hunt with Sentinel, and let Copilot speed the humans.** The rest is just implementation detail—and we’ll walk through that next.

## My thesis: Zero-Trust Supply Chain

We still need early sharing — defenders must prep. But we should **share signals, not secrets**.

**Practical moves for vendors & large orgs**

- **Tight partner rings** with telemetry-based trust and immediate revocation.
- **Staged, canary previews** (behavioral indicators, detections, mitigations) before deep exploit diffs.
- **Virtual patch guidance** shipped day-0 (WAF/pre-auth patterns, AMSI rules).
- **Signed partner access** with watermarking + monitoring for leak fingerprints.

This doesn’t kill collaboration; it **zero-trusts it**.

## Make on-prem SharePoint “zero-day tolerant”

The goal: even if a bug is unknown, attackers can’t hit it unauthenticated, and you can see/stop post-exploitation.

### 1\. Put SharePoint behind pre-authentication

**Don’t** expose SharePoint directly to the internet.  
**Do** publish it through **Microsoft Entra Application Proxy** or **Entra Private Access** so every request is identity-checked (Conditional Access) **before** SharePoint sees it. This breaks unauthenticated RCE chains.

- _How-to (SharePoint + App Proxy):_ Microsoft Learn step-by-step. [Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/app-proxy/application-proxy-integrate-with-sharepoint-server)
- _What is Entra Private Access (ZTNA):_ overview and deployment guides. [Microsoft Learn](https://learn.microsoft.com/en-us/entra/global-secure-access/concept-private-access)

**Suggested Conditional Access for the published app**

- Require **phishing-resistant MFA** (FIDO2/Windows Hello for Business).
- Require **compliant or hybrid-joined device**.
- Block access from risky sign-ins or risky users.
- Optional: location controls for high-value tenants.

### 2\. Turn on AMSI and use Full Mode (SPSE)

AMSI inspects **requests** and can block exploit payloads **before** they hit the endpoint logic. Use Microsoft Defender AV and enable Full Mode where available. Microsoft explicitly raised this in their response guidance. [Microsoft](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/) [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)

**Docs:** Configure AMSI integration for SharePoint Server (incl. request body scanning and Full Mode). [Microsoft Learn](https://learn.microsoft.com/en-us/sharepoint/security-for-sharepoint-server/configure-amsi-integration)

### 3\. Rotate machine keys and restart IIS after patching

Attackers stole ASP.NET **MachineKeys**; rotate them across the farm after applying updates or enabling AMSI — **then restart IIS**. (Microsoft called this out strongly.) [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)

```powershell
\# Rotate ASP.NET machine keys for each SharePoint Web App

Get-SPWebApplication | ForEach-Object {

Write-Host "Rotating keys for $($\_.Url)"

Set-SPMachineKey -WebApplication $_

Update-SPMachineKey -WebApplication $_

}

iisreset /restart
```

### 4\. Segment & egress-restrict your WFE

- WFEs should have **no outbound internet** except what’s needed (updates, Defender, time).
- Place admin interfaces on **management subnets/VPN** only.
- Optional: front with **Azure WAF on Application Gateway** for virtual-patch rules while you roll out fixes.

### 5\. Keep to supported versions and install cumulative updates

Microsoft released comprehensive updates for **2016/2019/Subscription Edition**; SharePoint Online wasn’t affected. [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)

## SharePoint Online vs Microsoft 365 Local vs classic on-prem

1. **SharePoint Online (M365)** — Microsoft runs the service; this incident did **not** affect it. You offload patching of the service layer. [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)
2. **Microsoft 365 Local** — an on-prem/sovereign deployment option announced for EU needs. Helpful for sovereignty and tight control, **not automatically air-gapped**. You still must do **pre-auth**, **segmentation**, **AMSI**, and **patch discipline**. [The Official Microsoft Blog](https://blogs.microsoft.com/blog/2025/06/16/announcing-comprehensive-sovereign-solutions-empowering-european-organizations/?utm_source=chatgpt.com)
3. **Classic on-prem** — highest flexibility, **highest exposure** if directly internet-facing.

## Hunt and detect (copy/paste KQL)

**Use where you have the data**

- **Defender XDR Advanced Hunting** tables (Device\*) → great for process/file telemetry.
- **Sentinel/Log Analytics** (W3CIISLog, SecurityEvent, Sysmon) → great for server and IIS logs.
- If you’re using the new **Sentinel Data Lake** for auxiliary logs

### 1\. Defender XDR: find dropped web shells (spinstall0.aspx)

(Microsoft shared this pattern in their guidance.) [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)

```kql
DeviceFileEvents

| where FolderPath has_any ("Web Server Extensions\\\\16\\\\TEMPLATE\\\\LAYOUTS",

"Web Server Extensions\\\\15\\\\TEMPLATE\\\\LAYOUTS")

| where FileName startswith "spinstall" and FileName endswith ".aspx"

| project Timestamp, DeviceName, InitiatingProcessFileName, InitiatingProcessCommandLine,

FileName, FolderPath, SHA256

| order by Timestamp desc
```

### 2\. Defender XDR: IIS worker spawning PowerShell (post-exploit)

(Microsoft shared this in their guidance.) <https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/>

```kql
DeviceProcessEvents

| where InitiatingProcessFileName =~ "w3wp.exe"

| where FileName in~ ("cmd.exe","powershell.exe")

| where ProcessCommandLine has_any ("-EncodedCommand","-ec","powershell")

| project Timestamp, DeviceName, InitiatingProcessFileName, FileName, ProcessCommandLine
```

### 3\. Sentinel (IIS logs): suspicious POSTs to ToolPane endpoint

_(Microsoft observed exploitation via ToolPane.)_ [_Microsoft_](https://www.microsoft.com/en-us/security/blog/2025/07/22/disrupting-active-exploitation-of-on-premises-sharepoint-vulnerabilities/)

```kql
W3CIISLog

| where csMethod == "POST"

| where csUriStem has "/\_layouts/" and csUriStem has "ToolPane"

| summarize cnt=count(), firstSeen=min(TimeGenerated), lastSeen=max(TimeGenerated)

by sSiteName, csUriStem, cIP, csUserAgent

| where cnt > 5 // tune threshold for your traffic
```

### 4\. Sentinel (Sysmon): web shell write → suspicious child process

```kql
// File create of \*.aspx under LAYOUTS + child process from w3wp.exe

let layouts = dynamic(\["\\\\15\\\\TEMPLATE\\\\LAYOUTS\\\\","\\\\16\\\\TEMPLATE\\\\LAYOUTS\\\\"\]);

let lookback = 7d;

let files =

Sysmon

| where TimeGenerated > ago(lookback) and EventID == 11

| where TargetFilename has_any (layouts) and TargetFilename endswith ".aspx";

let proc =

Sysmon

| where TimeGenerated > ago(lookback) and EventID == 1

| where ParentImage endswith "\\\\w3wp.exe"

| where Image has_any ("\\\\cmd.exe","\\\\powershell.exe","\\\\cscript.exe");

files

| join kind=leftouter proc on $left.Computer == $right.Computer

| project TimeGenerated=files.TimeGenerated, Computer, TargetFilename,

SuspiciousChild=proc.Image, ProcCmd=proc.CommandLine
```kql

### 5\. Sentinel (Windows Security logs): IIS worker spawning shells (no Sysmon)

```kql
SecurityEvent

| where EventID == 4688

| where ParentProcessName endswith "\\\\w3wp.exe"

| where NewProcessName has_any ("\\\\cmd.exe","\\\\powershell.exe","\\\\cscript.exe","\\\\wscript.exe")

| project TimeGenerated, Computer, Account, ParentProcessName, NewProcessName, CommandLine
```

### 6\. Quick analytic rule idea (every 5 min)

```kql
DeviceProcessEvents

| where InitiatingProcessFileName =~ "w3wp.exe"

| where FileName =~ "powershell.exe"

| where ProcessCommandLine has_any ("-EncodedCommand","-ec")
```

**Response (automation):** isolate host (Defender), kick an **IIS reset**, and add an **IR tag**. Then run the key-rotation script.

## Why the new Microsoft Sentinel Data Lake matters

You can now keep **high-volume, auxiliary logs** (IIS, firewall, DNS, inventories) for **years** at lower cost, and still query with **KQL jobs** in the Defender portal. That makes long-lookback hunts possible when zero-days stretch over months. [Microsoft Learn](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-lake-overview) [Microsoft](https://www.microsoft.com/en-us/security/blog/2025/07/22/microsoft-sentinel-data-lake-unify-signals-cut-costs-and-power-agentic-ai/)

**Example: schedule a daily job over IIS** (Data Lake table names may vary in your tenant):

```kql
// Saved as a Data Lake KQL job: hunt unusual POST bursts to /\_layouts/\*

IISDataLake

| where TimeGenerated between (ago(25h) .. now())

| where csMethod == "POST" and csUriStem has "/\_layouts/"

| summarize count() by bin(TimeGenerated, 1h), csUriStem

| order by TimeGenerated asc
```

Use **Summary Rules** to distill noisy logs into compact tables your real-time analytics can alert on. <https://learn.microsoft.com/en-us/azure/sentinel/summary-rules-tutorial>

## Minimal, opinionated hardening checklist

1. **Remove public exposure**; publish via **Entra App Proxy / Private Access** with Conditional Access. [Microsoft Learn](https://learn.microsoft.com/en-us/entra/identity/app-proxy/application-proxy-integrate-with-sharepoint-server)
2. **AMSI enabled** (SP2016/2019/SPSE), **Full Mode** on SPSE 25H1+, and **Defender AV**. [Microsoft Learn](https://learn.microsoft.com/en-us/sharepoint/security-for-sharepoint-server/configure-amsi-integration)
3. **Patch** to current cumulative updates; **then rotate machine keys**; **restart IIS**. [Centre de réponses Microsoft Security](https://msrc.microsoft.com/blog/2025/07/customer-guidance-for-sharepoint-vulnerability-cve-2025-53770/)
4. **Block egress** from WFEs; segment admin/management.
5. **Hunt and alert** using the KQL above; store auxiliary logs in **Sentinel Data Lake** for long-term hunts. [Microsoft Learn](https://learn.microsoft.com/en-us/azure/sentinel/datalake/sentinel-lake-overview)
6. **Practice IR**: scripted containment (host isolation), IIS recycle, key rotation, credential hygiene, restore from clean backups.

## Copilot for SecOps: fast triage

A simple prompt you can keep on hand:

**“Copilot, summarize SharePoint exploitation indicators in the last 24h across Defender and Sentinel. Highlight any spinstall\*.aspx file writes, w3wp.exe → powershell process chains, and POSTs to /\_layouts/ endpoints. Propose containment steps and list affected hosts and users.”**

_(Copilot doesn’t replace the KQL above — it speeds up human response.)_

## Conclusion

This wasn’t only a software bug. It was a **trust failure**. The fix is not just patching; it’s **zero-trusting the supply chain**, **pre-auth at the edge**, **AMSI in Full Mode**, and **keeping long-retention logs** so we can prove, not guess, what happened.