+++
title = "Generating Option 121 Classless Static Route options for DHCP"
date = "2023-05-12T21:45:00Z"
description = "How to work out what value to provide for O-121 to provide routes to clients"
+++

Option 121, while incredibly usefull, sure is a fucking pig to use. The option tends to be specified using hexadecimal values of shortened IPv4 network addresses prepended with the CIDR notiation of the mask also in hex. This leads to something like `option 121 hex 180a.0100.0a00.1efe` in Cisco-ish. Wtf does that mean??

## What is O-121

For those who don't know, DHCP O-121 is another way to provide routes to clients other than setting the `default-gateway` option. The main difference is that O-121 lets you send multiple CIDR routes allowing an automatic routing table like below. This can make routing to other local networks far more efficient than just routing the traffic through the default gateway, especially for networks like mine where the DGW is a piece of shit 1841 with 100Mbps NICs and my server has 4-8Gb of bandwidth to the LAN depending on what I'm doing with it at the time. The obvious answer would be to use a L3 switch but I don't have one and DHCP options are free.

| Route          | NextHop       |
| -------------- | ------------- |
| 0.0.0.0/0      | 192.168.0.1   |
| 172.16.0.0/24  | 192.168.0.100 |
| 172.16.21.6/32 | 192.168.0.200 |

## How the value works

So how do we work out the option values for these routes? We have to convert everything to hex. Nibble-by-nibble. So for example the `172` and `16` ports of the network address have to be converted to hex as `ac` and `10`. The mask also needs converting, so a `24` bit mask is written as `18`. This gives us the most of what we need for writing out the `172.16.0.0/24` route but how? The format is `<mask><network><gateway>` so we start with `18` (\24 in hex), then add `ac` and `10` (172 and 16 in hex) after.


Now we have `18ac10` which is our network address. In this notation, we only need the network-portion to be specified, hence this is all we need. If you wanted to specify a route to a host address, you'll obviously have to provide all 32 bits (ie. `20ac101506`) and prepend a 32-bit (`20`) mask value.

If we wanted to have 2 layer-3 domains on one layer-2 segment simply adding `18ac10` now would let your clients talk to each other, if you were that way inclined (I am not).

Next step, add the next hop address to our value. To do this, convert the address the same way as before. Using the example in the table above, we're using `192.168.0.100` for this network's gateway. This works out as `c0`, `a8`, `00`, and `64`. Combine everything and we get a value of `18ac10c0a80064`. 

This might have to be written slightly differently depending on which DHCP server you're using, for example some will use it as `18:ac:10:c0:a8:00:64` and others will use it as `0x18ac10c0a80064`. The Cisco engineers must've been smoking something when they wrote their implementation because they take it as dotted-quads. I'm sure they'll have a Cisco Certified Hexadecimal Dotting Professional certification somewhere in their programmes. Anyways, the trick for Cisco is to whack a `.` in after every 4th position, giving us `18ac.10c0.a800.64`

## Script to generate this shit on it's own

You can generate these values using the printf command, for example to generate a `/32` route to `176.16.21.6` via `192.168.0.200` you can run:

```
$ printf "%02x" 32 172 16 21 6 192 168 0 200
20ac101506c0a800c8
```

With this, you can also create multiple routes. For everything in the table of networks above, this script will do:

```
$ printf "%02x" 00 192 168 0 1 24 172 16 192 168 0 100 32 172 16 21 6 192 168 0 200 
00c0a8000118ac10c0a8006420ac101506c0a800c8
```

The script is the easiest way, by-far.