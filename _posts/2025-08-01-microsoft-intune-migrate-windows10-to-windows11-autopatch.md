---
title: "Migrating Windows 10 to Windows 11 with Microsoft Intune"
date: 2025-08-01 11:30:00 +0100
categories: [Microsoft, Intune] 
tags: [microsoft,Intune]
image:
  path: /assets/img/20250801/migrate-to-windows-11.png
  alt: Migrating Windows 10 to Windows 11 with Microsoft Intune
---

# Migrating Windows 10 to Windows 11 with Microsoft Intune

With Windows 10 reaching end-of-life in October 2025, organizations are planning their move to Windows 11. Fortunately, modern tools like Microsoft Intune make this upgrade **much easier** than past OS migrations. Microsoft Intune (part of Microsoft Endpoint Manager) leverages Windows Update for Business to **remotely upgrade** devices with minimal admin effort. In this guide, we’ll cover how to use Intune to seamlessly migrate Entra ID–joined Windows 10 devices to Windows 11 with the least manual steps. We’ll also compare what it takes if you don’t have Intune, highlighting Intune’s strengths in simplifying the process.

Intune’s native reports in **Endpoint Analytics** let you check hardware readiness, TPM support, etc., before deploying. If you’re not using Intune, upgrades require manual installer runs or imaging. With Intune, you centrally manage target OS, scheduling, and compatibility — saving admin time and ensuring consistency.

Without Intune or modern management, upgrading many PCs can be labor-intensive – you might manually run in-place upgrades, use imaging tools, or rely on users to initiate updates. Intune eliminates those hassles by centrally pushing the Windows 11 feature update to eligible devices. You can even let Intune handle device eligibility, scheduling, and enforcement, so upgrades happen **automatically** with minimal disruption. Let’s dive into the requirements and steps to achieve this.

## 1\. Prerequisites & Readiness Checks

### Licensing & Enrollment

- Devices must be **Entra ID‑joined** or **Entra hybrid‑joined**, managed by Intune (or co‑managed via Configuration Manager with tenant attach enabled).
- Use Windows 10/11 **Enterprise E3/E5** or **Education A3/A5**, or Microsoft 365 Business Premium. Pro is supported but lacks gradual rollout options.

### Autopatch Integration

If your organisation is already enrolled in **Windows Autopatch**, Intune will automatically generate **Autopatch groups**, update rings and feature‑update policies for you. This creates standardised deployment rings and staged rollout schedules out‑of‑the‑box. Rather than manually creating update rings or feature update profiles, you can leverage these Autopatch groups—and if they don’t yet exist—you simply create them under **Tenant administration → Windows Autopatch → Autopatch groups** in the Intune admin centre. Then configure the deployment rings and ensure **feature updates** are enabled in your Autopatch release. You can then either use the multi‑phase feature update release workflows that assign updates to Autopatch groups, or assign standard Intune feature update policies directly to those groups. This way you combine Autopatch’s built‑in scaffolding with targeted, controlled Windows 11 deployment.

There is a super [Microsoft Tech community article from Akash Malhotra](https://techcommunity.microsoft.com/blog/windows-itpro-blog/upgrade-to-windows-11-with-windows-autopatch-groups/4434497) on that subject

### Configure Telemetry & Feedback

- Under **Tenant administration > Connectors and tokens > Windows data**, set **Enable features that require Windows diagnostic data** to **On**.
- Apply a **Device configuration policy** that sets telemetry to at least **Required** level and includes **Windows Update** monitoring via Health Monitoring.

### Hardware Readiness Report (Endpoint Analytics)

This report lets you see which devices are ready for Windows 11— **natively**, no Premium add‑on needed:

- In the **Intune admin center**, go to **Reports > Endpoint Analytics > Work from anywhere**, then click the **Windows** tab.
- You’ll see devices categorized as **Capable, Not Capable, Upgraded**, or **Unknown**. The **Readiness reason** column shows blockers (e.g. TPM, Secure Boot, CPU, storage).
- Devices marked **Unknown** often mean they’ve been inactive—check last check‑in time and focus on active devices.

You can filter and export this view, and sort devices by their readiness status or blocking reasons.

![Desktop View](/assets/img/20250801/screenshot-intune-work-anywhere-report.png){: .shadow}

Screenshot taken from a small prod Tenant with a live migration.

On the previous screenshot, you can see an old Intel NUC 7<sup>th</sup> generation that cannot be upgraded to Windows 11 due to old generation processor (and also probably old version of TPM 1.2 instead of 2.0).

## 2\. Planning with Feature Update Reports

Intune also provides built‑in Feature Update reporting to help plan OS roll‑outs:

- Go to **Reports > Windows updates > Windows Feature Update device readiness report**. There select your **Target OS** (e.g., Windows 11 version 23H2) and scope. Generate the report to see per‑device compatibility status and if any device is blocked from upgrading due to hardware or other issues.
- For a high‑level view, use **Windows Feature Update compatibility risks report**, which summarizes the top compatibility blockers across devices (broken apps, drivers, hardware, etc.).

These reports require the same prerequisites: Entra ID‑joined devices, telemetry enabled, devices running Windows 10/11 supported builds, and licensing validated in Windows data settings.

| **Report & location in Intune** | **What it shows** |
| --- | --- |
| Endpoint Analytics → Work from anywhere → _Windows_ | **Device-level readiness** (Capable / Not Capable / Unknown / Upgraded) + reasons like TPM, CPU, SecureBoot, etc. |
| Reports → Windows updates → _Feature update device readiness_ | Compatibility status per device for your specific Windows 11 target OS |
| Reports → Windows updates → _Compatibility risks_ | Aggregated view of common blockers (apps/drivers/hardware) across devices |

Summary Table: Readiness Reports & What They Show

## 3\. Updating to Windows 11 with Minimal Effort

Once readiness is validated, proceed:

### Define Feature Update policy

- Navigate to **Devices > Windows > Feature updates for Windows 10 and later**
- Create profile, name it, select target (e.g. Windows 11 23H2) and rollout behavior (Required or Available). If eligible hardware varies, enable fallback so devices that can't upgrade go to the latest Windows 10 instead.

### Configure Update Ring policy

- Set **feature update deferral** to a low or zero day value so upgrade begins promptly.
- Apply **quality update deadline** to force reboot/install since Windows 10 ignores feature update deadlines, ensuring upgrades complete.

![Desktop View](/assets/img/20250801/screenshot-intune-autopatch-policy-ring.png){: .shadow}

### Assign to groups

- Target a pilot group first, then broader device groups as rollout proceeds.

### Monitor deployment

- Feature update reports show which devices are in progress, completed, or failed.
- Compatibility risks report helps troubleshoot repeated failures.

## Handling Hybrid‑joined or Unmanaged Devices

**Hybrid Entra ID joined devices**: auto-enroll via Group Policy + Entra ID Connect. Upgrades behave the same once enrolled in Intune.

**Unmanaged or Workgroup devices**: first require **join to Entra ID** and **Intune enrollment** (e.g., via Company Portal). Only then can you use the above modern upgrade path. Otherwise you must fall back to manual upgrades or imaging—no built‑in Intune reporting or control.

## Conclusion

Intune’s modern tools and **Endpoint Analytics** empower you to start with clear hardware readiness insights and then deploy Windows 11 upgrades with minimal manual work. Using up‑to‑date Microsoft terminology (Entra ID, Feature Update policies, Work from anywhere report), this updated process reflects the current best practice. Admins can confidently migrate to Windows 11, armed with native reports on TPM, Secure Boot, CPU, compatibility risks — all without extra licenses.

YouTube video of the end-to-end process soon. 
