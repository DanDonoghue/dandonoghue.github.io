+++
title = "127-bit masks vs 64-bit masks on Point-to-Point links"
date = "2016-05-08T15:47:00Z"
description = "Explaining why /127 is better than /64 on Point-to-Point links by counterarguing a Reddit comment"
+++

I was looking through the usual subs on Reddit this morning and there was a post about things that scare people, and one of the comments was from someone who's "scared of IPv6". Fair enough, hex does look intimidating at first, especially when you're only familiar with 32-bit dotted-decimal. Underneath this was a comment that said someting along the lines of "you can tell how familiar someone is at IPv6 by whether they use /127 or /64 on their point-to-point links", which is a reference to [RFC6164](https://tools.ietf.org/html/rfc6164) compared to the commonly spouted "use /64s everywere" nonsense suggested by a lot of IPv6 learning resources.

This agreeable comment was then replied to by a third person, saying they use /64s on P2P links, and gave their reasons for doing so. I think their reasons are a load of horse shit, and so I'm writing this post with my counterarguments for each of their reasons.

# Address conservation isn't an issue

Do you know who else thought address conservation wasn't an issue? Literally everyone towards the beginning of the IPv4 Internet. Say you're a SMB, assigned a `/56` from your ISP. That's 256x `/64`s, which seems like plenty now, but how about in 15 or 20 years? You can't know, because you don't know what know what direction technology will go in by then. It's better to setup something you *might* not need, than to implement a change later on.

# /127s are always at risk of vendor incompatibility

Only if you don't update your gear. If you're one of those people, you're bad at your job and should refund your salary. If the vendor isn't implementing the standards properly, then you bang on to them until they do, or you drop them as a vendor. Simple as that.

# /127s [...] make human readability very difficult

This one is just crap. It's fairly obvious that after `9`<sub>16</sub> comes `A`<sub>16</sub>, and that `A`<sub>16</sub> is equal to `10`<sub>10</sub>. All you have to do to tell if an address is the first address or last address in a subnet programatically is to modulo the last nibble by `2`. If the result is `0`, it's the first address. If the result is `1`, it's the last address.

This is actually as easy for people in IPv6 as it is for `/31` addressing in IPv4, because you can tell the different between an even and an odd number right? There's only two addresses per subnet. If it's even, it's the first address, otherwise it's the last address.

# Neighbour Discovery attacks exist for /64

This one doesn't really argue against `/127`s, but in my view argues for them. IPv6 Neighbour Discovery can be used for a DoS attack against a pair of nodes connected using P2P links with a `/64`. The commentator argues that you can configure routers to prevent this, but why bother with the extra effort? It's far quicker to just use a `/127` and be done, than to use a `/64` and then setup a fuck load of mitigations, especially since the attack vector for a `/64` is larger than that of `/127`s because of all the extra parts of IPv6 that run by default on a `/64` prefix.
