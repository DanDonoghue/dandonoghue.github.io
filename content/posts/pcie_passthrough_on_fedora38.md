+++
title = "PCIe Passthrough on Fedora 38"
date = "2023-05-20T10:00:00Z"
description = "How I got PCIe Passthrough to work for multiple VMs on F38"
+++

This is a lot simpler these days with modern-ish cards and a modern-ish OS because nVidia's lifted their wanky restrictions in their drivers. Unfortunately they haven't backported this change for cards from the Windows XP era which kind of sucks but fair enough, it is proper ancient.

For a single GPU passed through to one Windows XP virtual machine, I just had to add a "PCI Host Device" in virt-manager.. typically two actually, one for the GPU function 0, and one for the GPU's HDMI audio function 1. Once that's done, change the `Video` model to `none` and edit the VM's XML to look like the following.

{{< highlight xml >}}
<domain xmlns:qemu="http://libvirt.org/schemas/domain/qemu/1.0" type="kvm">
    ...
    <features>
        ...
        <hyperv mode="custom">
            ...
            <vendor_id state="on" value="1234567890ab">
        </hyperv>
        <kvm>
            <hidden state="on">
        </kvm>
        ...
    </features>
    ...
    <qemu:override>
        <qemu:device alias="hostdev0">
            <qemu:frontend>
                <qemu:property name="x-vga" type="bool" value="true"/>
            </qemu:frontend>
        </qemu:device>
    </qemu:override>
</domain>
{{</ highlight>}}

This XML will make KVM hide that it's a VM just enough for the nVidia driver to not throw a hissy. This is enough for passing through a 2nd GPU to a single VM, but I want more. I want to be able to play classic LAN multiplayer games on two VMs because I forgot how much effort it is running Windows XP on a physical computer. With VMs, I can use backing disk images to revert any changes so if I install something that breaks the OS it's only a couple commands to get something usable back.

Now, the hard part was IOMMU grouping. For some reason, the 2nd GPU was being put into a group with a bunch of other random shit that made it refuse to pass through. This took a little bit of effort to sort out, with the help of [this repo](https://github.com/some-natalie/fedora-acs-override). Follow the readme instructions, I opted to download the prebuild kernel (hint: you gotta be logged in to GitHub to download the build artefacts). Obviously, this work around will compromise stability and security and all that jizz, but this is just a fuck around thing to play a couple old games.

Once the patched kernel's installed, if you reboot and **highlight** the boot option with `acs` in the kernel version and then press **e** to edit it and add `pcie_acs_override=downstream,multifunction` then with any luck when you boot you're 2nd graphics card will be in it's own IOMMU group and be available for passthrough. If you want this option to be permanent then add it to `/etc/default/grub` otherwise you'll have to edit the boot commands every time.

I have 3x graphics cards installed on this system, a GTX 1060 6GB, a GT 710, and a 9500GT. The GT 710 ran solid with no issues at all, but the 9500GT would occasionally hang the system is you rebooted the VM. I could do with replacing it with something newer but it's perfectly capable of running the kind of games I'll be playing, plus it's a single slot GPU that fits in my bottom x16 slot.

Here's a video of how it looks at the end of it all

{{< youtube sbxhLOQTu_k >}}