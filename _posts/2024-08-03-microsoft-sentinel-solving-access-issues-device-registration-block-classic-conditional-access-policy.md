---
title: "Solving Access Issues in Microsoft Sentinel: Device registration block"
date: 2024-08-03 16:30:00 +0100
categories: [Microsoft, Security] 
tags: [microsoft,sentinel,xdr,siem,cybersecurity,iam]
image:
  path: /assets/img/20240803/Microsoft_Sentinel_DataConnector_Sign-in_error1.png
  alt: Solving Access Issues in Microsoft Sentinel - Device registration block
---

# The Hidden Classic Conditional Access Policy

When I recently faced a frustrating access issue with Microsoft Sentinel, it took me on a deep dive into Azure's Conditional Access settings. This blog post is a comprehensive guide to solving the problem of modifying Data Connectors in Microsoft Sentinel when access is blocked due to unregistered devices, and how I ultimately found the answer in an unexpected place.

## The Problem

I was unable to modify any Data Connectors in Microsoft Sentinel. Each time I attempted, I was prompted to sign in again, only to be met with an error message stating that access was restricted because my device wasn't registered. Additionally, there was an irrelevant comment about my browser, which only added to the confusion. The sign-in logs provided some insight but were not entirely helpful, mentioning error code 50131 and pointing to "Microsoft_Azure_Security_Insights."

### What is Conditional Access?

**Conditional Access** is a critical component of Microsoft Entra (formerly Azure Active Directory), playing a vital role in the Zero Trust security model. It allows administrators to enforce specific controls and policies that grant or deny access based on defined conditions. For instance, Conditional Access can require multifactor authentication (MFA) for all users accessing specific applications, restrict access to certain geographic locations, or mandate that devices meet specific security compliance standards.

The importance of Conditional Access in protecting a tenant cannot be overstated. It helps mitigate risks by ensuring that only authenticated and authorized users can access sensitive resources, thereby strengthening the organization's overall security posture.

### Unifi Network and Syslog Server

The client has a Unifi network setup, which includes routers and switches. We established a VM in Azure to serve as a Syslog server, collecting logs from these devices and forwarding them to the Sentinel Log Analytics workspace.

## The Journey to the Solution

Despite my extensive troubleshooting efforts, which included reviewing Conditional Access policies and checking device compliance, I couldn't pinpoint the issue. A comment from Haas Daniel, buried within what seemed like an AI-generated response from Microsoft, redirected my focus to Conditional Access, specifically to the possibility of legacy "Classic policies."

For those who have been working with Microsoft for a long time, encountering issues related to Classic policies is not uncommon. My tenant, created in early 2012 with Intune activated since 2018, likely had such a policy in place. Traditional Conditional Access policies were not causing the issue; rather, it was an old Classic policy that was blocking access.

### Troubleshooting with Conditional Access and Sign-In Logs

Using the "What If" tool in Conditional Access and thoroughly examining the sign-in logs are essential steps in troubleshooting such issues. The "What If" tool simulates the impact of Conditional Access policies, helping identify potential conflicts and misconfigurations. Also, for those who are concerned, don't forget to activate the preview feature. 

## Discovery

![Desktop View](/assets/img/20240803/Microsoft_Sentinel_DataConnector_Sign-in_error3.png){: .shadow}
![img-description](/assets/img/20240803/Microsoft_Sentinel_DataConnector_Sign-in_error3.png)
_The hidden legacy classic Condtional Access policies tab_

As shown in the screenshot, the culprit was indeed a single legacy Classic policy called "Windows Defender ATP Device Policy." This policy, likely an automatically generated Enterprise app from a long time ago when Defender was known as ATP, was blocking access. While I suspect this policy was part of a security default configuration, I'm keen to learn more about its origins. If anyone has additional insights, please share.

### Resolution

After disabling (but not deleting) the policy to ensure it wouldn't cause further issues, I checked whether I could now modify the Data Connectors in Microsoft Sentinel. Success! The access issues were resolved, and I could proceed with the necessary modifications.

## Conclusion

In summary, solving the issue of modifying Data Connectors in Microsoft Sentinel required identifying and disabling a legacy Classic Conditional Access policy. For administrators facing similar problems, always consider the possibility of legacy policies and use the available troubleshooting tools in Azure AD to diagnose and resolve access issues.

If you have any additional insights or similar experiences, please share in the comments. Your contributions can help others facing the same challenges in older tenants.

