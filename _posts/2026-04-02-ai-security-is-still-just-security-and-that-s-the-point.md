---
title: "AI security is still just security, and that's the point"
date: 2026-04-02 09:00:00 +0100
categories: [Security, Defender]
tags: [Defender,Defender-XDR,Microsoft-Entra,Microsoft-Purview,Microsoft-Sentinel,AI-Security,Copilot]
image:
  path: /assets/img/20260402/featured.jpg
  alt: AI security is still just security, and that's the point
---

## So what's this really about?

AI security isn’t a brand-new discipline. It’s the same identity, data protection, monitoring and governance work we’ve been doing for years, only now the consequences show up faster and in much weirder places. Microsoft’s latest guidance for CISOs leans into that idea, and I think they’re right to do it. Too many organisations are treating AI as either magic or inevitable, when really it’s just another workload with a larger blast radius and much better marketing.

Why does this guidance exist now? Because boards are asking awkward questions, business units are rolling out copilots and chatbots before security sees the design, and every vendor is promising “safe AI” with the sort of confidence normally reserved for people who haven’t had to run it in production. In my experience, most AI risk still lands in familiar buckets: over-permissioned users, poor data hygiene, weak app governance, exposed endpoints, and not enough monitoring. At heart, this is Zero Trust with a fancier press release.

Here’s the thing though. AI doesn’t usually invent a new security problem; it amplifies the ones you already had. Microsoft 365 Copilot is not a permission boundary. Azure OpenAI is not a substitute for access control. And if your tenant is already a bit messy, AI will surface that mess at machine speed. That’s exactly why applying security fundamentals to AI is useful advice, even if it sounds suspiciously boring.

## What I like about it

The positive side is that Microsoft isn’t pretending you need to throw away your existing security model and start again. If you’ve already invested in Microsoft Entra, Purview, Defender and Sentinel, you’ve got a decent foundation. Conditional Access, MFA, app consent controls, sensitivity labels, DLP, endpoint protection, and good logging still matter. None of that is futuristic, but boring controls are usually the ones that save your skin later.

Microsoft 365 Copilot is probably the best example of why this matters. People worry that Copilot will suddenly leak data, but the bigger issue is that it reveals existing oversharing. That sounds alarming, and honestly it is, but it’s also useful. Copilot forces organisations to finally look at SharePoint permissions, public Teams, stale access groups and those random “Everyone except external users” grants that have been lurking since 2019 like an unexploded bomb. Painful? Absolutely. Helpful? Also yes.

On the wider AI side, Defender’s discovery capabilities are genuinely handy for shadow AI. If you feed it telemetry from Defender for Endpoint or your proxy stack, Cloud Discovery gives you a view of which generative AI apps are actually being used, how much traffic is going to them, and how risky Microsoft rates them. That’s the sort of screenshot a CISO wants before a board meeting: real apps, real users, real traffic, not a hand-wavy statement that “we’ve got governance in place”.

I ran into this last week with a customer who swore the organisation had only approved Copilot. Defender told a slightly different story. There were users hitting ChatGPT, Gemini, Claude, and a note-taking AI tool I suspect was named by a marketing team after too much coffee. No incident, thankfully, but it was a perfect reminder that policy documents are not telemetry. The real game-changer here is visibility, because once you can see the AI estate, you can apply normal controls to it.

From a SecOps perspective, this is useful as well. Defender XDR and Sentinel let you treat AI activity as part of normal detection engineering rather than a separate science project. Risky consent, odd sign-ins, unusual uploads to AI apps, token misuse, data exfil patterns, all of that fits into workflows security teams already understand. If you’ve got Microsoft 365 E5, much of this is already in the stack. If you’re on E3, you can still do a lot, but discovery, richer data controls and risk-based access improve quickly once you add Entra ID P2, Defender for Cloud Apps, or E5 Compliance. Because obviously one licence was never going to be enough.

## The reality check

Of course, Microsoft being Microsoft, none of this lives neatly in one place. Your AI control story is split across the Entra admin centre, the Microsoft Defender portal, the Purview portal, Azure, and occasionally an oddly specific settings page hidden behind “more options”. The docs imply everything integrates smoothly. The docs also imply many things.

There are also blind spots. Discovery depends on telemetry. DLP can help with uploads and sharing, but it won’t magically understand every prompt or user intention. Prompt injection is basically social engineering for machines, and no single toggle in Microsoft 365 makes it go away. What you can do is reduce impact through access control, data minimisation, isolation and monitoring. That’s still worth doing, but let’s not pretend it’s some silver bullet.

And then there’s the maturity problem. Most of the core controls I’m talking about here are GA and dependable. Some of the shinier AI-specific dashboards and posture views are still preview, region-dependent, or attached to extra licensing. Don’t build your entire governance model around a preview tile in a portal. Also, for custom Azure AI deployments, I still see far too many resources left with public network access and long-lived API keys because the project needed to “move fast”. Right. Because nothing says innovation like explaining an avoidable exposure to the audit committee.

## Setting this up

If you want to turn all of this into something practical, here’s the baseline I’d recommend. The PowerShell below uses Microsoft Graph PowerShell 2.x, and the Azure commands assume a current Azure CLI install. Most of the controls here are GA as of April 2026, even if some of the AI-specific reporting around them still feels a bit… assembled live on stage.

First, get discovery working before you start writing policy. In the Microsoft Defender portal, go to **Cloud Apps > Cloud Discovery** and filter the category to **Generative AI**. The view you want should show app name, discovered users, traffic, transactions and risk score. If I were dropping a screenshot here, it would be that table, with ChatGPT, Gemini, Claude and some entirely unapproved AI helper sorted by user count. Sanction the apps you’re willing to allow, mark the rest as unsanctioned, and export the results for review. If you need the official walkthrough, Microsoft Learn’s [Cloud Discovery setup guide](https://learn.microsoft.com/defender-cloud-apps/set-up-cloud-discovery) is still the best place to start. I also like querying Entra for AI-related service principals so I’m not relying on network discovery alone:

```powershell
Connect-MgGraph -Scopes "Application.Read.All","Directory.Read.All","DelegatedPermissionGrant.Read.All"

Get-MgServicePrincipal -All |
  Where-Object {$_.DisplayName -match "Copilot|OpenAI|ChatGPT|Gemini|Claude|Anthropic"} |
  Select-Object DisplayName, AppId, ServicePrincipalType
```

Once you’ve got that list, review any broad delegated permissions as well, especially `Files.Read.All`, `Sites.Read.All`, `Mail.Read` or worse. That’s often where third-party AI tools get far more access than anyone intended.

Next, lock down access. In **Microsoft Entra admin centre > Protection > Conditional Access**, create one policy for administrators of AI platforms and another for end users of approved AI services. Start in **Report-only** mode, exclude your break-glass accounts, and require MFA plus a compliant or hybrid joined device. Conditional Access needs Entra ID P1. If you have Entra ID P2, add sign-in risk or user risk to tighten things further. The [Conditional Access overview](https://learn.microsoft.com/entra/identity/conditional-access/overview) is worth revisiting here because AI projects have a bad habit of creating service accounts and exceptions before anyone asks whether they should exist.

Third, fix the data layer before Copilot or an internal assistant starts surfacing it. In **Microsoft Purview > Information protection > Labels**, make sure sensitive content is labelled in a way humans and controls can actually use. Then go to **Data loss prevention > Policies** and block or warn on labelled content being uploaded to unsanctioned cloud apps. For Copilot specifically, spend time in SharePoint and Teams reviewing broad access groups, stale sites, anonymous links and open sharing. Copilot follows permissions faithfully, which is great right up until it isn’t. Thats usually where the skeletons are.

Fourth, if you’re building on Azure OpenAI or Azure AI Foundry, treat those resources like production applications, not developer toys. In the Azure portal, check **Networking** and **Identity** first. I want to see **Public network access = Disabled**, at least one approved private endpoint, and **System assigned managed identity = On**. If local auth is still enabled, you’re leaving API keys around for someone to forget about later. You can audit the basics quickly with Azure CLI, and Microsoft’s [Azure OpenAI networking guidance](https://learn.microsoft.com/azure/ai-services/openai/how-to/network) fills in the detail:

```bash
az login

az cognitiveservices account list \
  --query "[?kind=='OpenAI'].{name:name,resourceGroup:resourceGroup,location:location,publicNetworkAccess:properties.publicNetworkAccess,disableLocalAuth:properties.disableLocalAuth}" \
  -o table

az cognitiveservices account update \
  --name <AzureOpenAIResourceName> \
  --resource-group <ResourceGroupName> \
  --set properties.publicNetworkAccess=Disabled properties.disableLocalAuth=true
```

Test that in non-production first, obviously. Azure has a way of humbling people who enjoy confidence.

Finally, wire the telemetry into Defender XDR or Sentinel and hunt for behaviour, not headlines. In **security.microsoft.com > Hunting > Advanced hunting** or **Microsoft Sentinel > Logs**, start with simple usage baselines, consent monitoring and impossible-to-ignore anomalies. If you’re using Sentinel, make sure the Entra ID and Microsoft Defender connectors are enabled. A basic query like the one below is crude, but it quickly shows which AI apps are active and whether usage is trending in ways your policy docs definitely didn’t approve:

```kusto
CloudAppEvents
| where AppName has_any ("Copilot","ChatGPT","Gemini","Claude")
| summarize Sessions=count(), Users=dcount(AccountDisplayName), IPs=dcount(IPAddress) by AppName, bin(Timestamp, 1d)
| order by Timestamp desc
```

From there, add detections for new enterprise app consents, risky sign-ins, and spikes in uploads to unsanctioned apps. Don’t overcomplicate it on day one. A simple baseline beats an elegant detection no one has time to tune.

## What I’ve learned

Where Microsoft doesn’t give you a perfect answer, you can still reduce the pain. If Cloud Discovery visibility is patchy, supplement it with firewall, proxy or Secure Web Gateway logs, and use Defender for Endpoint web protection to control high-risk AI sites on managed devices. It’s not elegant, but it works. Security teams spend far too much time waiting for a perfect control when a decent layered workaround would do the job today.

If Copilot is exposing overshared content, don’t try to fix every SharePoint permission issue in a single heroic project. Pilot Copilot with a tightly governed group first. Clean up a handful of high-value sites, kill “Everyone” style access, remove stale sharing links, and show leadership the before-and-after. That usually gets more traction than a giant remediation spreadsheet nobody funds. Every Copilot deployment becomes a permissions clean-up exercise eventually, so you may as well admit it early.

For Azure AI, bake managed identities, private endpoints and disabled public access into your landing zone templates so dev teams don’t have to remember them on a Friday afternoon. And publish a very short AI acceptable use standard. One page is plenty. Approved tools, banned tools, what needs review, and who owns exceptions. If your waiting for perfect AI governance, the business will happily invent its own by lunchtime.

## My verdict

My take is pretty simple: Microsoft’s guidance is sensible because it refuses to pretend AI needs an entirely new religion. The fundamentals still win. Know what AI is in use, control access, protect data, secure the platform, and monitor everything. I would absolutely use this model for Microsoft 365 Copilot, Azure OpenAI, and any organisation trying to get ahead of shadow AI without killing innovation.

Bottom line: AI doesn’t replace your security programme. It exposes whether you had one. If you’re working through this in your own tenant, have a look around the blog for more Entra, Defender, Purview and Sentinel content, and feel free to connect with me through the links on the site. Always happy to compare notes, or shared frustration.

---

Source: [Applying security fundamentals to AI: Practical advice for CISOs](https://www.microsoft.com/en-us/security/blog/2026/03/31/applying-security-fundamentals-to-ai-practical-advice-for-cisos/) on the Microsoft Security Blog.