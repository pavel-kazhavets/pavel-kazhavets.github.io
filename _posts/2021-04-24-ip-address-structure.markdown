---
layout: post
title:  "All you wanted to know about IP addresses but was too afraid to ask"
date:   2021-04-24 15:10:56 +0900
categories: networks cidr ip 
keywords: cidr, ip address, ip, ip structure, subnets
---

## Intro
You may have encountered something like
```
192.168.0.1/24
```
what is `/24` for?
Or something like this on the next line
```
255.255.255.0
```
is it some other IP address? Does it point to the same thing? If not - what does it point to?

IP addresses can be very confusing at first. They do have different representations of the same thing (binary vs dotted-decimal for IP addresses, binary vs dotted-decimal vs CIDR notation for subnet mask), you may need to _calculate_ something, etc (?). This article aims to clarify structure of an IP address and explain things you commonly see in relation to them.

Let's start with the basics.

------------------------------------------------------------------------------------------------------------
## Structure of an IP address
IP address essentially consist of 4 numbers separated by dots which are also called _octets_. What is an octet? Basically octet is a number consisting of 8 bits - eight 1's and 0's.
Some confusion starts right here and as most of the IP address related confusion is caused by machines not operating with decimal numbers (like `42`) but with binary numbers (like `00101010` which is also 42).
Machines also operate with IP addresses in their binary form, so something that we see as
```
192.168.0.1
```

for our computers looks like
```
 11000000.10101000.00000000.00000001
(192     .168     .0       .1       )
```

In order to read this article you *don't* need to be able to calculate binary-to-decimal and back in your head. I certainly can't do that on big enough numbers (where big is "higher than 2"). Just try to visually understand how described operations work and assume that binary conversions here are correct (trust me, I used online calculator).

One important thing about binary numbers you've seen above - they have limited amount of combinations. Let's see an example of 3 bits. All possible combinations are:
```
000 - 0
001 - 1
010 - 2
011 - 3
100 - 4
101 - 5
110 - 6
111 - 7
```

if we continute counting furter to 8 we'll see that 8 in binary format is `1000` which is four bits.
Which means that 3-bit number only has 8 variations (0 through 7) which is 2^3 where 2 is number of possible values (0 and 1) and 3 is number of bits.

Going back to our octets it means that each of 4 numbers in IP address is also limited and it is limited to 2^8 which is 256. And since we start counting from 0 it means that largest number you will ever see in IP address is 255 (all 1's in binary - `11111111`).

So even theoretically IP address `348.9.2.20` is not valid since 348 > 255.

That's roughly it for basic IP addresses. Let's move to subnets.

-------------------------------------------------------------------------------

## Subnets
Sometimes it may be very useful to group IP addresses together. Usually it's done when _describing_ IP addresses, for example in routing or firewalls.
Let's imagine you're a network administrator and you see some malicious activity from outside of your network. Some strange requests are arriving at your loadbalancers. They are coming from different IP addresses, but there is something common in them: part of IP address is the same, like this:
```
17.4.4.99
17.4.4.105
17.4.4.143
14.4.4.190
14.4.4.197
14.4.4.206
14.4.4.215
...
```

First 3 octets are the same, but the last one is different.
For you, as a network administrator, it would be very convenient to be able to block all IP addresses starting with `17.4.4`. And so it happens that there is a way to do so.

There is a mechanism for grouping IP addresses called subnetting and these groups of IP addresses are called subnets. Let's look at how it works.

In general terms subnets are collections of IP addresses. Number of IP addresses is determined by *subnet mask*.
Subnet mask, same as IP adress itself, is composed of 4 octets. There is a difference though - on binary level 1's in subnet mask are contiguous. At first you have multiple 1's in a row and then only 0's:
```
11111111.11111111.11111111.00000000
```
which in decimal would be 
```
255.255.255.0
```

Or something like that:
```
11111111.11111111.11000000.00000000
```
which is
```
255.255.192.0
```

Cool. But how does it work?
It's called a subnet *mask* for a reason. Subnet mask is applied to IP address in a particular way which allows to split IP address in two parts - subnet and host. Subnet part will identify which network IP address belongs to and host part will identify, well, host on this network.

Each bit of subnet mask has a matching bit in IP address since they have the same size. All IP address bits which have matching subnet mask bit set to 1 will belong to subnet part, each bit which has subnet mask bit set to 0 will belong to host part. 

Let's see on example of IP `192.168.0.1` with subnet mask `255.255.255.0`:

```
11111111.11111111.11111111.00000000 - subnet mask
11000000.10101000.00000000.00000001 - IP
(         subnet         ).( host )
```

First 3 octets in this IP will indicate a subnet and last octet will indicate a host. Which means that range of our possible hosts is range of possible values for 8 bits in last octet. This gives us IP addresses from `192.168.0.1` till `192.168.0.255`.

So, in order to understand which part of IP address is network our machines will look at how many contiguous 1's there are in subnet mask and will take the same amount of bits in IP address for network part and leva the rest for host part.

This is heavily used in routing and firewalls because it allows grouping IP addresses together instead of listing each individual host.
For example, if your home network uses `10.10.0.1-255` IP range you can specify `255.255.255.0` as a subnet mask in order to tell your machines to route internal packets directly (through switch) and not go to router.


OK, but what is this `/24` thing? 

## CIDR notation
This `/{number}` postfix is called CIDR notation and is basically another *view* of a subnet mask. Remember that 1's in subnet mask can only be contiguous? It means that instead of saying `11111111.11111111.00000000.00000000` we can just say that there are 16 bits with value `1` and everything else is `0`. And we write this number after IP address for convenience like `192.168.0.0/16`. 

It's as simple as that. Here are some examples of different subnet masks in different notations (all with `10.0.0.0` address, will work in the same way for other addresses):

| dotted-decimal  | binary                                | CIDR | example       | possible hosts            |
|-----------------|---------------------------------------|------|---------------|---------------------------|
| `255.255.255.0` | `11111111.11111111.11111111.00000000` | `24` | `10.0.0.0/24` | `10.0.0.1-10.0.0.255`     |
| `255.0.0.0`     | `11111111.00000000.00000000.00000000` | `8`  | `10.0.0.0/8`  | `10.0.0.1-10.255.255.255` |
| `255.248.0.0`   | `11111111.11111000.00000000.00000000` | `13` | `10.0.0.0/13` | `10.0.0.1-10.7.255.255`   |
|                 |                                       |      |               |                           |

Something like `10.0.0.25/16` would mean that there is a `10.0.0.25` IP address on a network with subnet mask `255.255.0.0`.
When trying to only describe network itself we usually set all host bits to 0. So network from this example can be described as `10.0.0.0/16`. Often describing network and setting host bits (like mentioning `10.0.0.25/16` in a firewall instead of `10.0.0.0/16`) would be considered invalid and rejected by the software you're using.

But how does one remember how many IPs there are in a subnet with particular subnet mask? Well, short answer - you don't (unless you're doing subnetting on a daily basis for years).
Small mental hint: CIDRs which are a multiple of 8 are quite easy to calculate - remember that there are 4 octets containing 8 bits? So:
* `/32` is a subnet for a single IP address
* `/24` is a subnet where only last number is variable (`10.0.0.*`)
* `/16` is a subnet where last two numbers are variable (`10.0.*.*`)
* `/8` is a subnet where last three numbers are variable (`10.*.*.*`)

Subnet's like `/13` or `/27` can be tricky to calculate in mind, so either use some IP calculator (I usually use [this one][ipcalc]) or remember another mental hint: increasing subnet mask by one - let's say from `/24` to `/25` will effectively split it in half. For example a single `10.0.0.0/24` network can be split into two: `10.0.0.0/25` (last octet values ranging from 1 to 127) and `10.0.0.128/25` (from 129 to 254).

There is a convention that first possible IP in the subnet (10.0.0.0 in 10.0.0.0/24 subnet) is used to identify network itself and the last one (10.0.0.255) is used for [broadcast][broadcast-address].


[ipcalc]: http://jodies.de/ipcalc
[broadcast-address]: https://en.wikipedia.org/wiki/Broadcast_address

