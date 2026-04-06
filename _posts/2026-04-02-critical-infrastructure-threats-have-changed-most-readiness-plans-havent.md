---
title: "Critical infrastructure threats have changed. Most readiness plans haven’t"
date: 2026-04-02 09:00:00 +0100
categories: [Security, Defender]
tags: [Defender,Defender-XDR,Security-Exposure-Management,High-Value-Assets,Sentinel]
image:
  path: /assets/img/20260402/featured.jpg
  alt: Critical infrastructure threats have changed. Most readiness plans haven’t
---

## So what’s actually changed?

Right, so Microsoft has published another warning to critical infrastructure leaders, and for once it isn’t just the usual “threats are evolving” wallpaper. I think the real point is simpler, and a bit more uncomfortable: the threat model for critical infrastructure in 2026 is less about some movie-style hacker smashing a power station console, and more about boring, ugly access paths that start in identity, remote access, supplier relationships, or an internet-facing edge device nobody’s touched since 2022.

That’s why this matters. Microsoft Threat Intelligence is basically saying the old split between IT risk and OT risk doesn’t hold up anymore. Attackers don’t need to “hack the plant” first. They log in as someone who can reach it, pivot through a jump box, abuse a remote support tool, or sit quietly in a supplier path until the timing is useful. Grim, but not exactly surprising.

In my experience, that lands far beyond utilities and national infrastructure. Hospitals, manufacturers, transport operators, councils, logistics firms, even large retail distribution networks all have systems that become “critical infrastructure” the minute they stop working. If payroll, patient care, line control, warehouse automation, or core identity falls over, the label hardly matters.

I think Microsoft built this message because boards, insurers, and regulators are all asking the same question now: are you actually ready for the bad day, or have you just bought a pile of licences and hoped for the best? That’s a much better question. It also exposes how alot of organisations still use Defender, Sentinel, and Entra as separate tools, when they really need to behave like one joined-up readiness stack.

## What I like about this

The strongest part of Microsoft’s framing is that it pulls the conversation away from raw alert volume and back towards readiness. That sounds obvious, but plenty of security programmes still confuse “we collect telemetry” with “we can stop a serious incident”. Those are not the same thing. A SOC can be absolutely drowning in alerts and still miss the one path that matters because nobody marked the crown jewels in the first place.

This is where Defender XDR and Security Exposure Management actually earn their keep. When you identify high-value assets properly, map attack paths, and correlate signals across identities, endpoints, email, and cloud apps, you stop treating every red dot the same. That matters massively in critical environments, because the question isn’t “how many devices are at risk?” It’s “which compromised device would ruin my week, get me on a regulator call, or stop physical operations?”

I’ve seen environments with brilliant endpoint coverage on user laptops and absolutely no shared view of which servers run directory services, historian workloads, privileged access tooling, or vendor access gateways. That isn’t resilience. It’s expensive ambiguity. Once you start tagging or otherwise classifying those systems, the Defender and Sentinel story gets much better. Incidents involving those devices can be prioritised. Exposure can be filtered. Hunting becomes meaningful rather than theatre.

There’s also a very practical benefit to Microsoft’s direction here: the mature pieces are already there. Defender XDR Advanced Hunting is solid. Custom detections are solid. Sentinel analytics rules are solid. Conditional Access and Privileged Identity Management are proven. None of that is new or especially sexy, but it works when you wire it together properly.

For the community, that’s the real bit worth paying attention to. You don’t need to be running a nuclear facility to learn from this. If you support any organisation with a handful of systems that simply cannot fail, the same pattern applies. Focus on identities that can reach the assets, the assets that can stop operations, and the detections that show you lateral movement before it becomes a press statement.

## The catch, naturally

Of course, Microsoft being Microsoft, there’s a catch or three.

First, the tooling doesn’t magically solve OT visibility. Defender for Endpoint on a Windows jump server is useful, but it is not the same as understanding what’s happening on PLCs, HMIs, serial gateways, or industrial network segments. If your estate includes real OT, Defender for IoT or another proper OT monitoring platform is still part of the conversation. Pretending EDR alone covers that gap is how you end up discovering “air gapped” meant “there was a TeamViewer session somewhere”.

Second, the user experience is still a bit inconsistent. Some tenants get newer Security Exposure Management views and high-value asset experiences faster than others. Some admins still bounce between `security.microsoft.com`, `entra.microsoft.com`, and Azure portal like it’s a scavenger hunt. If you don’t see the exact menu Microsoft showed in a demo, your not losing it. The portal probably changed again.

Third, licensing is still messy. Entra ID P1 is the floor for Conditional Access. P2 is where Identity Protection and PIM become available, and honestly that’s where serious privileged access hygiene starts. Defender for Endpoint Plan 2 is the baseline if you want advanced hunting, TVM, and proper response actions. Sentinel is separate consumption, because apparently incident response needed a taxi meter.

And finally, some response actions are harder in critical environments than the docs make them sound. Isolating a workstation is one thing. Isolating a production server that talks to a control system is a very different decision. If your only monitoring Windows endpoints and cloud identities, you may still be blind to the thing that will actually stop the plant.

## How I’d build this in a real tenant

If I had to improve critical infrastructure readiness this week in a Microsoft estate, I’d start with four things: mark what matters, lock down who can touch it, hunt around it, and rehearse containment before you need it.

**1. Mark the assets that actually matter**

Start in the Microsoft Defender portal at `https://security.microsoft.com`. Depending on your tenant layout, go to **Assets > Devices** or **Endpoints > Device inventory**. If you have the richer Exposure Management views available, check **Exposure management** for any **critical assets** or **high value assets** experience first. If not, don’t wait for perfect UI. Use machine tags.

The devices I care about first are domain controllers, VPN gateways, jump boxes, privileged admin workstations, management servers, historian servers, backup servers, and vendor access hosts. On the device inventory grid, the columns worth pinning are **Exposure level**, **Risk level**, **Sensor health**, and **Device tags**. If those critical systems aren’t onboarded and visible here, that’s your first problem.

For bulk tagging, the Defender API is still the easiest route. Export your device inventory so you have `MachineId` values, then use a simple CSV with `MachineId,Tag`.

```powershell
$tenantId  = "<tenant-id>"
$appId     = "<app-id>"
$appSecret = "<client-secret>"

$tokenBody = @{
  client_id     = $appId
  client_secret = $appSecret
  scope         = "https://api.securitycenter.microsoft.com/.default"
  grant_type    = "client_credentials"
}

$token = (Invoke-RestMethod -Method Post `
  -Uri "https://login.microsoftonline.com/$tenantId/oauth2/v2.0/token" `
  -Body $tokenBody).access_token

$headers = @{
  Authorization = "Bearer $token"
  "Content-Type" = "application/json"
}

Import-Csv .\critical-assets.csv | ForEach-Object {
  $body = @{
    Value  = $_.Tag
    Action = "Add"
  } | ConvertTo-Json

  Invoke-RestMethod -Method Post `
    -Uri "https://api.securitycenter.microsoft.com/api/machines/$($_.MachineId)/tags" `
    -Headers $headers `
    -Body $body
}
```

Give the app the relevant Defender API permissions such as `Machine.ReadWrite.All`, and in production use a certificate or managed identity rather than a client secret you’ll forget exists by next quarter. Microsoft’s docs for [machine tagging](https://learn.microsoft.com/microsoft-365/security/defender-endpoint/machine-tags) are worth bookmarking.

**2. Lock down the people who can reach those assets**

Next, move to Entra at `https://entra.microsoft.com` and go to **Protection > Conditional Access > Policies**. Create a policy for privileged users, server admins, and any vendor access group. I normally target groups rather than roles first, because it’s easier to reason about operationally.

Set **Cloud apps** to **All cloud apps** and in **Grant**, require an **Authentication strength** of **Phishing-resistant MFA**. Exclude emergency access accounts, obviously. If you have Entra ID P2, place the relevant roles behind PIM and make activation time-bound with justification. In a critical infrastructure environment, standing privilege should be treated like asbestos: if it’s everywhere, you’re already in trouble.

Conditional Access is Entra P1 territory. PIM and risk-based Identity Protection need P2. For this use case, I wouldn’t cheap out. The [authentication strengths documentation](https://learn.microsoft.com/entra/identity/authentication/concept-authentication-strengths) is clear, and it’s one of the few places where Microsoft’s story is actually pretty tidy.

**3. Hunt around the critical devices, not just the noisy ones**

Once the assets are tagged, go back to Defender and open **Hunting > Advanced hunting**. This is where the whole thing starts to feel useful. I like to hunt for remote administration tools and lateral movement around tagged devices first, because that’s the sort of thing that turns into a very bad week in CI environments.

```kusto
let CriticalDevices =
    DeviceInfo
    | where DeviceTags has_any ("HVA","Critical","OT")
    | summarize Tags = any(DeviceTags) by DeviceId, DeviceName;
DeviceProcessEvents
| where Timestamp > ago(24h)
| where FileName in~ ("psexec.exe","paexec.exe","anydesk.exe","teamviewer.exe","winrs.exe","mstsc.exe")
| join kind=inner CriticalDevices on DeviceId
| project Timestamp, DeviceName, InitiatingProcessAccountName, FileName, ProcessCommandLine, Tags
| order by Timestamp desc
```

If this query returns your tagged devices and the expected tooling, good. If it returns half the estate, your tagging is wrong. If it returns nothing at all, either your environment is clean or more likely nobody has onboarded the important systems properly.

From there, either turn it into a **Custom detection** in Defender XDR, or send Defender XDR data into Sentinel and build a scheduled analytics rule. In Azure portal, go to **Microsoft Sentinel > Data connectors** and enable the **Microsoft Defender XDR** connector. Then in **Analytics**, create a scheduled rule from your validated query, run it every 5 minutes with a 15-minute lookback, and map **Host** and **Account** entities. The useful docs are [Advanced Hunting](https://learn.microsoft.com/microsoft-365/security/defender/advanced-hunting-overview) and the [Sentinel connector guidance](https://learn.microsoft.com/azure/sentinel/connect-microsoft-365-defender).

**4. Rehearse containment on something safe**

Finally, practise a containment action on a non-production jump host or test server. Not because the API is exciting, but because people freeze when they’ve never done it before.

```powershell
$machineId = "<machine-id>"

$body = @{
  Comment       = "Readiness drill - isolate critical asset test"
  IsolationType = "Full"
} | ConvertTo-Json

Invoke-RestMethod -Method Post `
  -Uri "https://api.securitycenter.microsoft.com/api/machines/$machineId/isolate" `
  -Headers $headers `
  -Body $body
```

This needs the relevant response permission such as `Machine.Isolate`. Do not point this at a domain controller or a production control server just because you want a dramatic demo. The point is to confirm who approves the action, what the SOC sees, how operations are notified, and how recovery actually works.

And if you genuinely run OT, add Defender for IoT or equivalent visibility into the plan. Defender for IoT is a separate licence, yes, because that would have been too straightforward otherwise. But if you need visibility into industrial protocols or unmanaged operational kit, it matters.

## The workarounds that save time

If the shiny high-value asset views aren’t present in your tenant yet, don’t sit around waiting for them. Use Defender machine tags, then mirror the same list into a Sentinel watchlist. It’s not elegant, but it works. I’d rather have a slightly scrappy control that exists today than a beautiful roadmap slide.

If licensing is mixed, prioritise where it has the biggest blast-radius reduction. Put Entra P2 on privileged users and anyone with operational access. Put Defender for Endpoint P2 on servers, jump boxes, and management workstations before you worry about every single office laptop. If budget is tight, spend it where compromise turns into outage.

I’d also strongly recommend separate vendor identities, separate admin workstations, and time-bound access wherever possible. The worst pattern I still run into is a shared contractor admin account that “everyone knows not to abuse”. Lovely. That sort of trust model belongs in a museum.

And if isolation isn’t operationally safe for certain assets, build your playbooks around account disablement, Conditional Access blocks, network segmentation, and upstream containment instead. Not every response action has to be endpoint isolation. Sometimes cutting off the identity path is the cleaner move.

## Bottom line

My take is pretty simple: Microsoft’s message here is right, even if the product story around it is still a bit messy. Critical infrastructure readiness in 2026 is less about buying another shiny control and more about being honest about what would actually hurt, who can reach it, and how quickly you can see and stop abuse around it.

Would I use Defender XDR, Sentinel, and Entra for this? Absolutely. But only if they’re wired together around critical assets and privileged access, not deployed as separate islands with separate owners.

Bottom line: defend the identities, mark the assets that matter, and rehearse the bad day before it arrives. If you’re working through this in your own tenant, have a look around the rest of the blog and feel free to connect with me through the usual links.

---

**Source:** Microsoft Security Blog, [*The threat to critical infrastructure has changed. Has your readiness?*](https://www.microsoft.com/en-us/security/security-insider/threat-landscape/threat-to-critical-infrastructure-has-changed), published 31 March 2026.