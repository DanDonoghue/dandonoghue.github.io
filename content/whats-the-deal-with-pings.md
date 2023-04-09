+++
title = "What's the deal with pings?"
date = "2016-05-15T19:20:00Z"
description = "Some people seem to think that turning ICMP Echo Replies off makes your network more secure. I'm going to explain why this is a load of old bollocks."
+++

Throughout the years I've been told to turn off ICMP Echo Request (`ICMP Type 8`) and ICMP Echo Reply (`ICMP Type 0`), commonly referred to as "Ping". Some of these people have been people who *should* know about this kind of thing, like University teachers and people who work in Network Administration. Their reason: "security". I say: "bollocks".

Using the old "security" excuse is bad enough, because as soon as you mention security to someone who doesn't actively keep up to date with current and past attacks they tend to assume you do. This flaw should usually be circumvented by referencing actual attack techniques that exploit the conditions you want to change, but at no point throughout the years has this been done by anyone whose told me to turn off ICMP Echo. Because of this, I decided to research **why** someone should turn off the ICMP type, and to be honest I was underwhelmed by the reasons.

# ICMP Echo can be used to determine if a host is up

Apparently ([1](http://www.linuxhowtos.org/Security/disable_ping.htm)), being able to determine if a host is up or not is bad. The typical reasoning for this is that if someone can tell if a host is up, attackers can use this information towards their reconnaissance against a target. What this reasoning lacks is the fact that you can determine this information without Echos. You can see this by looking at the way [Nmap](https://nmap.org/) works. Usually (non-stealth) Nmap fires off an ICMP Echo Request to the host you're scanning, and uses the response to determine if the host is up. Fair enough. 

The reasoning, however, falls down at the next stage. If there is no ICMP Echo Reply received, Nmap will then use other means of determining if the host is up such as performing a TCP or UDP connection and looking at the response received. A TCP packet to a non-listening port will cause a TCP `RST` packet to be sent back, thus confirming that a host is using the destination IP. A UDP packet sent to a non-listening port will cause an ICMP Port Unreachable (`ICMP Type 3, Code 3`) packet to be send, also confirming a live host. 

**You don't need ICMP Echo to determine if a host is up**, but it's easier for your IT staff to determine if something's working by using ICMP than it would be if they had to rely on a protocol that operates at a higher layer.

# ICMP Echo can be used for data exfiltration

ICMP Echo works by sending some `data` to a node, and that node returning the same `data` back. This is useful to Network Engineers because it can determine whether any bit-flipping has occured, which is where one or more bits in a packet have changed during transmission, which is symptomatic of a problem at a lower layer such as memory corruption, unreliable medium, or broken NICs. Because ICMP Echo packets can be any size, up to the maxiumum payload of any packet for the IP version in use, it's possible to fill the `data` portion of the packet with whatever you want, including the binary representation of internal documents.

**Turning off ICMP Echo does not prevent this, and there are other much more common protocols used for exfiltration like DNS, HTTPS, and HTTP**. If data exfiltration is a genuine security risk, you can limit the size of the payload at firewalls, and even trigger alerts for when a node is sending an abnormal amount of Echos to external addresses. If this is still not enough, you can obviously just block the ICMP Echo types at a firewall near your network boundaries, which will still allow your internal support staff to make use of Echos for diagnosing internal issues.

In fact blocking ICMP Echo Requests at the boundary firewalls is probably a good idea anyway, because there's typically not a legitimate reason for people outside your network to know the state of nodes inside your network. What I'm against is turning off ICMP Echo at the host-level as this removes a useful tool from the internal support staff.

# ICMP Echo has previously been used for DoS attacks

**The "Ping of Death" hasn't been relevant since the '90s**. If everyone was against any protocol that had a similar vulnerability at any point in its lifetime, we'd have no Internet. I studdied a vulnerability in Microsoft's Remote Desktop Protocol that caused the server to blue screen when it received a packet crafted in a specific way, but are people against RDP specifically because of this one (patched) vulnerability? No! So why the hate for ICMP Echo, and why cite this problem that was fixed nearly two decades ago?

# ICMP Echo source addresses can be spoofed

The idea behind this one is that if you send an Echo Request to a node, with a spoofed source address, the node will reply to the real node at the spoofed address. I talked about this in [127-bit masks vs 64-bit masks on Point-to-Point links](/127-bit-masks-vs-64-bit-masks-ptp/), but this does not just affect ICMP Echo. 

**You can do the same thing with any other protocol over IP. Turning off ICMP Echo doesn't solve it**, but instead you should be setting up spoofing protections at the network boundaries to drop any traffic with an unexpected source address for the interface. In fact, turning off ICMP Echo for this reason might give one a false sense of security, as you're likely to still be vulnerable to spoofing attacks, just on any other protocol than ICMP.
