---
title: "Printer Automatus: The Ultimate Printer Management Script"
date: 2024-12-23 07:30:00 +0100
categories: [Microsoft, Intune] 
tags: [microsoft,intune,automation]
image:
  path: /assets/img/20241223/Printer_Automatus.webp
  alt: Printer Automatus - The Ultimate Printer Management Script
---

# Printer Automatus: The Ultimate Printer Management Script

## Sharing My Secret Weapon

Let’s be honest, printers are the bane of every IT professional’s existence. In a world where automation reigns supreme, managing printers remains one of the last strongholds of manual labor. They always find a way to derail us, whether it's driver issues, connection problems, or the unexplainable errors they love to throw at us during crunch time.

I’ve been managing printers for years, and trust me, I’ve seen it all. From random port conflicts to drivers that seem to disappear into the abyss, I realized the only way to make my life easier was to develop a tool that could take care of these annoying tasks for me. Enter: **Printer Automatus** (because, why not make it sound fancy?).

This script is not perfect. In fact, it probably lacks all the error handling a perfectionist would demand. But you know what? It works. For 99% of the cases I’ve had to deal with, it has been my secret weapon. Initially, I thought about keeping it to myself, maybe even selling it as some kind of IT sorcery. But then I remembered: **we all suffer from the printer overlords**. Why not share it so others can finally reclaim their sanity?

You can find the project and the tutorial on my GitHub: https://github.com/cyrusirandoust/Intunus/tree/main/printer-automatus 

## What This Script Does

This script automates nearly everything you need to set up a network printer on a Windows system. No more digging through `.inf` files or praying that a driver miraculously installs itself. Here’s what it handles for you:

- Creates a TCP/IP port for the printer’s IP address.
- Installs the driver using your specified `.inf` file.
- Adds the printer with the correct name and driver.
- Optionally sets it as the default printer.

It’s all wrapped up in a simple PowerShell script that even works with Intune for centralized deployments.

## How It Works

Before diving into the nitty-gritty, there are a few things to prepare:

### Step 1: Extract Your Driver Files
Printer drivers often come as `.exe` files that extract themselves during installation. To use this script, you need to extract these files manually. Tools like [7-Zip](https://www.7-zip.org/) make it easy:

1. Right-click the `.exe` file and choose **7-Zip > Extract Here** or **Extract to <Folder Name>**.
2. Find the extracted contents, including `.dll`, `.cat`, `.inf`, and other required files.
3. Place all these files, along with the script, in a single folder.

You’re now ready to proceed.

### Step 2: Customize the Script
The script is designed to be flexible, but you’ll need to update a few key variables to suit your environment. For example:

- Set the printer name, IP address, and driver name to match your setup.
- Specify the path to your `.inf` file if needed.

### Step 3: Use the Detection Script
If you’re deploying via Intune, you’ll also need a detection script to verify whether the printer has been successfully installed. This script checks if the printer is present on the system and tells Intune if the deployment was successful.

```powershell
$PrinterName = "SHARP BP-55C26 PCL6"

# Get the list of installed printers
$InstalledPrinters = Get-Printer

# Check if the printer is in the list
$PrinterInstalled = $InstalledPrinters.Name -contains $PrinterName

if ($PrinterInstalled) {
    Write-Host "Installed"
}
```

### Step 4: Package the Script for Intune
If you’re managing printers through Intune, you’ll need to package the script as a `.intunewin` file. The process is simple:

1. Use the [Microsoft Win32 Content Prep Tool](https://learn.microsoft.com/en-us/mem/intune/apps/apps-win32-app-management).
2. Run the packaging command, specifying your script and source directory.

   ```cmd
   .\IntuneWinAppUtil.exe -c "C:\Intune\source\printer-automatus" -s Install-PrinterAutomatus.ps1 -o "C:\Intune\output\printer-automatus"
    ```

   - `-c`: Path to the folder containing the script and supporting files.
   - `-s`: Name of the PowerShell script.
   - `-o`: Path to save the `.intunewin` package.

3. Upload the `.intunewin` file to Intune and configure the app with the detection script.

---

## Why I’m Sharing This

For years, this script was my secret weapon. It saved me countless hours of frustration and let me focus on the parts of IT that I actually enjoy. But as much as I wanted to hoard this gem, I realized something: **printers are a shared pain**. Keeping this to myself was just selfish.

By sharing this script, I hope to give you back some of your precious time. Maybe it won’t work in every single case, but for most printers, it gets the job done. And if it doesn’t? Well, you’re free to tweak it and improve it (just give me some credit, okay?).

## Conclusion

This script isn’t perfect, and I’ll be the first to admit it. But it works well enough to make my life—and hopefully yours—a little easier. Feel free to adapt it, improve it, or just enjoy having one less printer-related headache.

And remember, if this script helps you out, let me know! Sharing it with the community is my way of giving back, but hearing your success stories would be the cherry on top.

Happy printing (if that’s even possible)!
