+++
title = "LSI2008 SAS Controller on EL9"
date = "2023-04-09T17:41:00Z"
description = "The effort needed to get a SAS controller working on EL9"
+++

# Problem

I've neglected my home server for a while now so I figured I'd upgrade it with some new boot SSDs and a case that isn't hot-boxing my hard-drives. New boot drives, new OS; I decided to go with Rocky Linux 9.1 since CentOS has been fucked around with. 

Part of this upgrade meant moving my existing HDDs off of the integrated SATA controller onto their own dedicated SAS controller with nice clean breakout cables. I decided to test out my setup on a spare PC before messing with anything important and discovered that RedHat in their infinite wisdom have "depreciated" all models of the LSI2008 from EL. It still works fine on Fedora 38 and Windows out of the box, if that makes any sense at all?!

# Solution

By deprecating a perfectly fine module, RH turned something simple into a pain in the ass. Shit should be plug-and-play on Linux, that's like one of the biggest benefits of the whole thing.

This guy on YouTube explains the whole fix but the video's 33 minutes about a SAS controller, who's got the attention span for that!

{{< youtube 4fOAuXiynYM>}}

## Fixing with boot drive on seperate controller

So to sort this shitshow out, you have to enable the ElRepo repository, install `kmod-mpt3sas`, and reboot; which you can do as follows:

```
rpm --import https://www.elrepo.org/RPM-GPG-KEY-elrepo.org
dnf install https://www.elrepo.org/elrepo-release-9.el9.elrepo.noarch.rpm
dnf install kmod-mpt3sas
reboot
```

## Fixing with boot drive on SAS controller

If you want to have your boot drive on the SAS controller, you need to install with a driver disk. The guy in the YouTube video shows how to do this pretty well, but basically you got to download the correct driver disk from https://elrepo.org/linux/dud/el9/x86_64/ (the ones starting `dd-mpt3sas` for LSI2008) and write the image to a USB drive.

To force driver disk initialisation in EL, append `inst.dd` to the kernel args when you boot the installer. This will make it ask where to load the driver from, even if it doesn't auto-detect the disk.

Once shit's installed you can add the ElRepo repository just like above to have access to updates in the package manager.