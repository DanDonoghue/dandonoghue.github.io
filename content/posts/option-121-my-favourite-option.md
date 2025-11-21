+++
title = "Option 121: My Favourite Option"
date = "2016-04-25T09:49:00Z"
description = "I had some performance issue with copying files to but not from a subnet. DHCP option 121 to the rescue!"
+++

I'll start this post off with this: don't judge my skills on how my home network is setup. I use this setup to test out and learn about different techniques, some good, some bad. I would never setup a client's network the way my home network is, but I wouldn't know the best way to do something without trying the wrong ways first.

As part of my server reinstallation, I've been messing about with the routing on my network. Previously, all routing for the LAN was handled by the server itself using the routing features in the Linux kernel. This brought with it several performance issues, mainly when the server was under load the latency of all traffic apart from subnet-local ended up going to shit. This time I decided to use my Cisco 1841 as the default gateway of client-site subnets, reducing the hop count to the demarc by one.

This has already given a performance gain of around 30ms on the RTT of packets to the Internet, and everything seems to work nicely. That was until I started noticing that all my domain computers were playing up, firstly when my desktop PC refused to connect up with my SCCM server and install the software it needed. Then I noticed that the other laptops I've got that are domain-joined were all performing slower than usual, with constant Explorer crashes, and shutdowns where the computer actually went to sleep becaues it took so long. Something was not right.

I had a go at a non-scientific performance test, whereby I copied a reasonably-sized ISO to and from the domain file server. Somewhat oddly, I could read the file at ~100MBps (960Mbps) but could only write at ~11MBps (88Mbps). It would appear that something was bottlenecking the upstream. Shit.

![File Transfer without route](/images/no_route_copy.png)

There's only one thing on my network that has 100Mbps interfaces, the Cisco 1841. Oh, it makes sense now. ICMP redirects appear to not persist between separate packets within a single stream, limiting the bandwidth between two 1Gbps nodes to the bandwidth available to the device issuing the redirects.

I needed some way of telling the clients to use the Cisco 1841 as the default gateway, except when connecting to the server subnet. This is what DHCP option 121 is for. Option 121 allows you to push CIDR routes to DHCP clients similar a one-way basic routing protocol. The client OS will then see these option 121 routes and add them to their routing table, as well as the default route. This means that you can have a network host with the default gateway 192.168.0.1/24 with the ability to reach 192.168.1.0/24 via a router on the same subnet at 192.168.0.2 without having that traffic passing through the default gateway. Definitely useful for when you've got a virtual network on a hypervisor that you wish to access from the LAN.

![File Transfer with route](/images/with_route_copy.png)

Option 121 can be setup on a Windows DHCP server by adding it as a scope option, and accepts a network address, classless subnet mask, and gateway address. Clients should either have routes to, or be directly connected to, the network where the gateway address is, otherwise the route won't work. You also need to add a route for 0.0.0.0/0 pointing to your default gateway within option 121, as some clients ignore the default gateway option when option 121 is specified.

![Adding DHCP option 121 to a Windows DHCP Server](/images/opt121.png)
