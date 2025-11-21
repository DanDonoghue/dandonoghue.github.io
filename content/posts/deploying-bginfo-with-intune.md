+++
title = "Deploying BGInfo with Intune"
date = "2023-06-18T14:49:00Z"
description = "Quick and dirty deployment of BGInfo to make labbing easier"
+++

This is purely robbed from this [Reddit post](https://www.reddit.com/r/Intune/comments/rk9x8h/bginfo_for_intune_endpoint_manager/) with me un-fucking the formatting so it works.

The basic instructions are to:

1. Get BGInfo from [Sysinternals](https://learn.microsoft.com/en-us/sysinternals/downloads/bginfo)
2. Run BGInfo, create your template, and save it as `bginfo.bgi`
3. Get `IntuneWinAppUtil.exe` from this [GitHub repo](https://github.com/microsoft/Microsoft-Win32-Content-Prep-Tool)
4. Package the scripts, template, and BGInfo
5. Add the package to Intune, and deploy it

# Creating the .intunewin package

You'll need an install/uninstall script, which are shown here. Dump these in a folder as `install.ps1` and `uninstall.ps1` along with your `bginfo.bgi` templace and `BGInfo64.exe`. Since this is for labbing, I won't bother with signing or any of that stuff.

{{< gist DanDonoghue 56bc81798f97a8846c1ec9f7d911a4cc >}}

The next step is to run `intunewinutil.exe -c SOURCE -s bginfo64.exe -o DESTINATION -q`. Once this is done, you'll have a `.intunewin` file you can upload as a Win32 application.

# Creating the app in Intune

In this deployment I just used the *all-devices* deployment group, but you can target it at any group you want. I've done this on my setup for the virtio-win driver MSI so that it only installs on machines with a quick `device.deviceManufacturer -eq "QEMU"` rule.

{{< youtube BFyX8EyTPho >}}

| | |
|-|-|
|Install command: |`powershell -ex bypass -file install.ps1`|
|Uninstall command: |`powershell -ex bypass -file uninstall.ps1`|
|Detection folder: |`C:\Program Files\BGInfo`|
|Detection file: |`bginfo.bgi`|