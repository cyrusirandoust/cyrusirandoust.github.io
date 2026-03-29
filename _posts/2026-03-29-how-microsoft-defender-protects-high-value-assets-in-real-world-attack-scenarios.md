---
title: "How Microsoft Defender protects high-value assets in real-world attack scenarios"
date: 2026-03-29 09:00:00 +0100
categories: [Microsoft, Defender]
tags: [Defender,Defender-XDR,Security-Exposure-Management,Defender-for-Endpoint,Defender-for-Identity,High-Value-Assets,Sentinel]
image:
  path: /assets/img/20260329/defender-high-value-assets/featured.jpg
  alt: "A practical look at how Microsoft Defender and Security Exposure Management help protect domain controllers, web servers, and identity infrastructure that attackers actually care about."
---

Attackers do not care about the average device in your estate. They care about the machines and identities that let them own the rest of it. Domain controllers, identity sync servers, certificate infrastructure, admin workstations, internet-facing web servers tied to authentication, that’s where the real damage starts. So I think Microsoft’s push around protecting high-value assets with Microsoft Defender and Microsoft Security Exposure Management is less a shiny new feature and more an overdue correction.

Here’s the thing. Most security tools have spent years treating assets like they’re roughly equal unless an analyst adds context later. But attackers don’t work like that. They don’t look at a medium severity alert on a web server and think, “probably harmless”. They look at it and ask whether it’s the shortest route to a domain controller, an AD CS server, or Entra Connect. Microsoft clearly built this capability to answer that exact problem: stop thinking only in alerts, start thinking in paths and business impact.

In my experience, that matters a lot more for lean IT and security teams than for the giant enterprise poster-child tenants Microsoft always loves showing off. If your team is already buried in alerts, you need something that tells you what actually matters first. This is huge for hybrid organisations with more history than documentation. But, and this is important, if your not giving Defender the right telemetry and asset context, it won’t magically invent good security architecture on your behalf.

## The good stuff

The best part of this approach is that it finally gives Defender some sense of priority beyond pure technical severity. A vulnerable IIS server is one thing. A vulnerable IIS server that can be used to pivot into identity infrastructure is a very different problem. Microsoft Security Exposure Management helps Defender understand that relationship, and that changes how incidents get triaged. Instead of “device with suspicious behaviour”, you get “device with suspicious behaviour that sits three hops away from your crown jewels”. That’s a much more useful sentence when the phone goes at 11pm.

This is especially helpful in hybrid environments where identity, endpoint, and cloud telemetry all overlap. Defender for Endpoint can see process execution and device risk. Defender for Identity can see lateral movement, sensitive accounts, and weird domain behaviour. Exposure Management sits over the top and gives you the connective tissue. That means a compromised admin workstation, a stale service account, and an internet-facing server don’t show up as three unrelated headaches. They show up as one attack path. That’s a big difference.

I’ve seen this help most in scenarios where the original issue looked boring. Think an old web server with local admin rights, or a support account that was never scoped properly. On paper it’s just another misconfiguration. In reality it’s the first domino in a chain that ends with certificate abuse or domain dominance. Once you visualise that route, the remediation conversation gets a lot easier. “Patch more things” is vague. “Break this path to your PKI server today” tends to get people moving.

The other bit I like is the tie-in with automatic response. Defender XDR’s automatic attack disruption has been quietly one of the better pieces of the stack for a while now. When it works well, it stops attackers from using the initial foothold to move laterally into the systems that actually matter. And if you’ve got Microsoft Sentinel connected, that context flows nicely into the SOC workflow instead of living in a silo. That’s not glamorous, but it is practical, which I care about a lot more.

## The reality check

Naturally, there’s a catch. Actually, a few.

The first one is coverage. Defender can’t protect the box it can’t see, and high-value assets are often the exact systems that got skipped in some half-finished onboarding project three years ago. I ran into this last week in a lab where every user device was onboarded beautifully, but the Entra Connect server and an old certificate server were basically ghosts. Of course Microsoft being Microsoft, the tooling looked healthy enough at first glance until you started checking the right blades.

The second problem is the portal experience. Some of the views are under **Exposure management**, some are under **Assets** or **Endpoints**, some sit in **Settings**, and the labels still move around depending on which tenant rollout ring your stuck in. The docs say it “seamlessly integrates”, which in Microsoft-speak means you’ll spend a little while muttering at the screen and opening three tabs to find one setting.

There’s also a more technical limitation: attack path analysis can get noisy if the environment is flat, badly tagged, or full of legacy exceptions. If every admin account can touch everything and half the servers are local-admin free-for-alls, the graph ends up looking like someone spilled spaghetti over your Tier 0 design. Helpful, yes. Beautiful, no. And licensing is not magically solved by owning Entra ID P1 or P2. You need the relevant Defender workloads licensed and deployed, especially on servers and identity infrastructure, or you’re only getting half the story.

## How to actually use it

If you want this to be more than a nice demo, I’d approach it in five passes.

**1. Get the right telemetry in place first.**  
For servers, make sure Defender for Endpoint is actually deployed on the systems you care about. For domain controllers and supported identity servers, make sure Defender for Identity sensors are healthy. If you’ve got Microsoft 365 E5 Security or the Defender XDR suite, you’re in a decent place. If you’re licensing servers separately, check whether you’re using Defender for Servers through Defender for Cloud, or standalone Defender for Endpoint server licensing. Exposure Management entitlements can vary a bit by workload, so check before promising miracles to procurement.

On Windows Server, I usually verify the basics locally with PowerShell first:

```powershell
Get-Service Sense

Get-MpComputerStatus | Select-Object AMServiceEnabled, RealTimeProtectionEnabled, AntivirusSignatureVersion
```

On Linux servers:

```bash
mdatp health
```

Then go to `https://security.microsoft.com` and check **Settings > Identities > Sensors** for your identity coverage. For server onboarding, Microsoft Learn is still the best starting point: [Onboard Windows Server to Microsoft Defender for Endpoint](https://learn.microsoft.com/en-us/defender-endpoint/onboard-windows-server).

**2. Mark the assets that are actually important.**  
This bit gets missed all the time. Go to **Assets > Devices** in the Defender portal, or **Endpoints > Device inventory** if your tenant still shows the older navigation. Open the record for your domain controllers, certificate servers, Entra Connect servers, admin jump hosts, and any authentication-adjacent web front ends. Set the **Device value** to **High** and add useful tags.

I tend to keep it simple: `Tier0`, `DC`, `PKI`, `EntraConnect`, `PAW`, `AuthWeb`. The screenshot you’re looking for here is the device details pane with the device value, exposure level, and tag area sitting annoyingly easy to miss in the overview card.

If you want to do this at scale, tag devices through the API instead of clicking forever. This works well if you already know the Defender device ID and you’ve got an app registration with the right permissions such as `Machine.ReadWrite.All`:

```powershell
$headers = @{
    Authorization = "Bearer $token"
    "Content-Type" = "application/json"
}

$body = @{
    Value  = "Tier0"
    Action = "Add"
} | ConvertTo-Json

Invoke-RestMethod -Method Post `
    -Uri "https://api.security.microsoft.com/api/machines/$DeviceId/tags" `
    -Headers $headers `
    -Body $body
```

If you’re still using the older `securitycenter` endpoint in scripts, update it. Future you will be less annoyed.

**3. Use Attack paths properly, not just as a pretty graph.**  
Now head to **Exposure management > Attack paths**. Filter for paths that end in high-value assets or the tags you just applied. The useful visual here is the graph with the critical asset as the destination node, upstream identities and devices feeding into it, and a remediation pane off to the side.

Don’t try to fix everything. Start with paths that end in domain controllers, PKI, identity sync, or internet-facing auth systems. Those are the paths that turn a bad day into a very expensive bad day. In a mature environment, the best recommendations are often boring: remove unnecessary local admin, tighten service account permissions, close unused RDP, patch a vulnerable web server, segment a legacy management network. Boring is good. Boring stops ransomware.

**4. Validate the risk with hunting, not vibes.**  
Once you’ve identified the critical devices, go into **Hunting > Advanced hunting** and see what’s actually happening around them. Microsoft Learn has the core guidance here: [Advanced hunting overview](https://learn.microsoft.com/en-us/defender-xdr/advanced-hunting-overview).

A quick check to list your high-value devices:

```kusto
DeviceInfo
| where AssetValue == "High"
| project DeviceId, DeviceName, OSPlatform, ExposureLevel, SensorHealthState, DeviceTags
| order by ExposureLevel desc
```

Then look for noisy or suspicious process activity on those systems:

```kusto
DeviceProcessEvents
| where Timestamp > ago(7d)
| where FileName in~ ("powershell.exe","pwsh.exe","rundll32.exe","regsvr32.exe","psexec.exe")
| join kind=inner (
    DeviceInfo
    | where AssetValue == "High"
    | project DeviceId, DeviceName, AssetValue, DeviceTags
) on DeviceId
| project Timestamp, DeviceName, FileName, ProcessCommandLine, AccountName, DeviceTags
| order by Timestamp desc
```

That query is not magic, but it’s a very good reality check. If you see routine admin activity, fine. If you see `rundll32.exe` doing something weird on a domain controller, well, congratulations, your afternoon just changed shape.

From there, use **Create detection rule** straight from the hunting query if it’s worth alerting on regularly.

**5. Make sure response actions are ready before you need them.**  
This part matters more than the marketing screenshots. In Defender for Endpoint, check your automation settings and device groups. For production servers, I usually start with more cautious automation and tighter approvals. For admin workstations and obvious high-risk endpoints, I’m happier being more aggressive. Go to **Settings > Endpoints > Device groups** and review the automation level for the groups containing your critical systems. Then monitor **Incidents & alerts > Action center** so you know what Defender is actually doing when it decides to contain something.

If your SOC runs in Sentinel, connect the **Microsoft Defender XDR** data connector in the Azure portal under **Microsoft Sentinel > Data connectors**. That gives analysts the incident story, entities, and response actions in the place they already live, instead of forcing them to swivel-chair across tools. It sounds small, but small bits of friction are where good detections go to die.

## Workarounds and tips

My biggest tip is simple: don’t rely on Microsoft’s inferred importance alone. Build your own tagging model around Tier 0 and the systems that would genuinely wreck your week if they were compromised. Domain controllers are obvious. Entra Connect, AD CS, privileged access workstations, jump hosts, SSO web servers, and secret-bearing app servers are just as important, even if the platform doesn’t always scream about them loudly enough.

If Attack paths feels noisy, that’s usually a sign the environment needs cleanup, not proof the feature is bad. Use device groups, segment old management networks, reduce local admin, and get serious about service account scoping. I also like creating one or two custom detections aimed specifically at unusual behaviour on tagged high-value devices. That gives you a safety net while the prettier Exposure Management insights catch up.

And don’t forget the blind spots. For public-facing workloads, pair this with Defender for Cloud or your existing external exposure tooling. Exposure Management is great at stitching relationships together, but it won’t replace proper hardening baselines, certificate hygiene, or sensible network design. A lot of ugly attack paths still come down to one over-permissioned service account and one server nobody wanted to reboot. Grim, but true.

## Bottom line

My verdict is pretty straightforward. I would use this, and I would use it most in hybrid environments where identity infrastructure still sits half on-prem and half in Microsoft 365. That’s exactly where attackers do the most damage, and exactly where defenders need better prioritisation. It won’t save a badly run environment on its own, but it does make the Defender stack far more useful when the basics are in place.

Bottom line: if you’ve already invested in Defender, spend the extra time identifying your crown jewels and checking the attack paths to them. That’s where the value is. If you’re testing this in your own tenant, I’d love to hear whether it’s been genuinely useful or just more graph-shaped stress. You can find the rest of my links around the blog, plus GitHub and socials, and follow along for more on Entra, Purview, Intune, Defender, and Sentinel.

---

Source: Microsoft Security Blog, [How Microsoft Defender protects high-value assets in real-world attack scenarios](https://www.microsoft.com/en-us/security/blog/2026/03/27/microsoft-defender-protects-high-value-assets/), published 27 March 2026.