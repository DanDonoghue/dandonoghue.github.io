+++
title = "Bridging Router Interfaces in IOS"
date = "2016-04-15T14:03:00Z"
description = "Needed to bridge two router interfaces on a Cisco router to save from having to install a switch."
+++

I have a strange network setup in my house with many subnets all over the place. To save from either putting a switch downstairs, or another cable run between downstairs and the main switches upstairs, I ended up just connecting the front-room TV to one of the WAN FastEthernet interfaces and setting up an internally-routed /24 for it. This setup gave the TV access to the Internet, but prevents it from reaching my Plex DLNA server, meaning no TV or films.

After some research, it seemed like the only way for DLNA to work was for both the client and server to be on the same subnet. Kind of annoying since putting in a switch by the router would've taken up a power socket that didn't exist. A little more research and I found out that the Cisco 1841 I'm using can create a bridge between the interfaces it has! Brilliant, I can put the TV on the same subnet as the rest of the house, without physically changing the setup.

In this setup, I was bridging my FastEthernet0/0 physical interface (the one the TV's connected to) with the FastEthernet0/1.50 interface, which is an 802.1q tagged sub-interface connected to the switch upstairs. Apparently you can bridge any kind of interface this way.

```
interface FastEthernet0/0
 no ip address
 ip virtual-reassembly
 duplex auto
 speed auto
 bridge-group 10
 bridge-group 10 spanning-disabled
interface FastEthernet0/1.50
 encapsulation dot1Q 50
 ip virtual-reassembly
 no cdp enable
 bridge-group 10
 bridge-group 10 spanning-disabled
bridge 10 protocol ieee
bridge crb
```

In the config shown above, I've setup bridge-group 10 to contain the **FastEthernet0/0** and **FastEthernet0/1.50** interfaces, disabled spanning-tree (since this is such a simple setup, it's not really worth the hassle), and set the bridging mode to CRB mode instead of IRB since this network doesn't need the router to be routing. If you need the router to be a gateway for the network, set the bridge mode to IRB and setup a BVI interface for it.

# Update

At some point after writing this article I changed my setup slightly, and now use a BVI. This involved setting up a 802.1Q subinterface on fa0/0, and setting that subinterface as native to allow the end device to connect without it having to be configured as a VLAN trunk.

The `bridge-group` value has to match the number of the `BVI` you wish to use (`10` in my case), but the BVI can be set to whatever number you want. Layer 3 addressing occurs on the BVI, as this is shared between the subinterfaces (`fa0/0.50` and `fa0/1.50`).

```
interface FastEthernet0/0.50
 encapsulation dot1Q 50 native
 no cdp enable
 bridge-group 10
 bridge-group 10 spanning-disabled
!
interface FastEthernet0/1.50
 encapsulation dot1Q 50
 ip virtual-reassembly
 no cdp enable
 bridge-group 10
 bridge-group 10 spanning-disabled
!         
interface BVI10
 ip address 192.168.100.1 255.255.252.0
 ip helper-address 10.1.0.2
 ip helper-address 10.1.0.3
 ip helper-address 10.1.0.50
 ip nat inside
 ip virtual-reassembly
 ip route-cache flow
!         
bridge 10 protocol ieee
bridge 10 route ip
!
```
